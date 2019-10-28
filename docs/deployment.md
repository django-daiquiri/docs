Deployment
==========

### Configuration

In production, you should create a dedicated user for your Daiquiri application. All steps for the installation, which do not need root access, should be done using this user. As before, we assume this user is called `daiquiri` and it's home is `/srv/daiquiri` and therefore your `app` is located in `/srv/daiquiri/app`.

In addition, a few more settings need to be configured in your `.env` file. The most important change is to set `DEBUG=False`.

```
DEBUG=False
ALLOWED_HOSTS=<hostname>

# ADMINS will get E-Mails in case of an error
ADMINS=Anna Admin <admin@example.com>, Manni Manager <manager@example.com>

LOG_DIR=/var/log/django/daiquiri
```

### Web server

Daiquiri can be run in different configurations, both with [Apache2] and [NGINX] as web server. Daiquiri itself is using the [wsgi] protocol for the communication between the HTTP and the Python layer. The recommended way of deploying Daiquiri is using [Apache2] as a reverse proxy and [Gunicorn] as [wsgi] server.

For this setup, you need to add:

```
PROXY=True
```

to your `.env` file.

First install Gunicorn inside your virtual environment:

```bash
pip install gunicorn
```

Then, test `gunicorn` using:
```bash
gunicorn --bind 0.0.0.0:8000 config.wsgi:application
```

This should serve the application like `runserver`, but without the static assets, like CSS files and images. After the test kill the `gunicorn` process again.

Systemd will launch the `gunicorn` process on startup and keep running. In order to start/restart/stop the web application as well as the asyncronous workers with one command, first create a pseudo-service for your Daiquiri application by creating the file `/etc/systemd/system/daiquiri.service` (you will need root/sudo permissions for that):

```bash
[Unit]
Description=pseudo-service for all Daiquiri services

[Service]
Type=oneshot
ExecStart=/bin/true
RemainAfterExit=yes

[Install]
WantedBy=network.target
```

Then create the systemd service file for the actual web application in `/etc/systemd/system/daiquiri-app.service`:

```
[Unit]
Description=Daiquiri gunicorn daemon
PartOf=daiquiri.service
After=daiquiri.service

[Service]
User=daiquiri
Group=daiquiri

WorkingDirectory=/srv/daiquiri/app
EnvironmentFile=/srv/daiquiri/app/.env

Environment=GUNICORN_BIN=/srv/daiquiri/app/env/bin/gunicorn
Environment=GUNICORN_WORKER=5
Environment=GUNICORN_PORT=9000
Environment=GUNICORN_TIMEOUT=120
Environment=GUNICORN_PID_FILE=/var/run/gunicorn/daiquiri/pid
Environment=GUNICORN_ACCESS_LOG_FILE=/var/log/gunicorn/daiquiri/access.log
Environment=GUNICORN_ERROR_LOG_FILE=/var/log/gunicorn/daiquiri/error.log

ExecStart=/bin/sh -c '${GUNICORN_BIN} \
  --workers ${GUNICORN_WORKER} \
  --pid ${GUNICORN_PID_FILE} \
  --bind localhost:${GUNICORN_PORT} \
  --timeout ${GUNICORN_TIMEOUT} \
  --access-logfile ${GUNICORN_ACCESS_LOG_FILE} \
  --error-logfile ${GUNICORN_ERROR_LOG_FILE} \
  config.wsgi:application'

ExecReload=/bin/sh -c '/usr/bin/pkill -HUP -F ${GUNICORN_PID_FILE}'

ExecStop=/bin/sh -c '/usr/bin/pkill -TERM -F ${GUNICORN_PID_FILE}'

[Install]
WantedBy=daiquiri.target
```

The setup needs to have several directories for logfiles set up with the correct permissions. This can be done using [tmpfiles.d]. First, create a file `/etc/tmpfiles.d/daiquiri.conf`:

```
d /run/celery/daiquiri        750 daiquiri daiquiri
d /run/gunicorn/daiquiri      750 daiquiri daiquiri

d /var/log/django/daiquiri    750 daiquiri daiquiri
d /var/log/celery/daiquiri    750 daiquiri daiquiri
d /var/log/gunicorn/daiquiri  755 daiquiri daiquiri
```

Then run:

```bash
systemd-tmpfiles --create
```

The `daiquiri` systemd service needs to be started and enabled like any other service:

```bash
sudo systemctl daemon-reload
sudo systemctl start daiquiri
sudo systemctl enable daiquiri
```

Next, install the web server. On Debian/Ubuntu use:

```bash
sudo apt install apache2 libapache2-mod-xsendfile

sudo a2enmod proxy
sudo a2enmod remoteip
sudo a2enmod headers
```

and on CentOS 7/8 use

```bash
sudo yum install httpd mod_xsendfile
```

Then, edit the virtual host configuration to create a reverse proxy to the Gunicorn server:

```
# in /etc/apache2/sites-available/000-default.conf  on Debian/Ubuntu
# in /etc/httpd/conf.d/vhost.conf                   on RHEL/CentOS
<VirtualHost *:80>
    ...

    DocumentRoot "/var/www/html"

    XSendFile on
    XSendFilePath <FILES_BASE_PATH from the Daiquiri settings>
    XSendFilePath <QUERY_DOWNLOAD_PATH from the Daiquiri settings>

    RequestHeader set X-Forwarded-Proto 'https' env=HTTPS

    ProxyPass /static !
    ProxyPass /cms !
    ProxyPass / http://localhost:9000/
    ProxyPassReverse /dev http://localhost:9000/

    Alias /static/ /srv/daiquiri/app/static_root/
    <Directory /srv/daiquiri/app/static_root/>
        Require all granted
    </Directory>

    # if you intent to use the WordPress integration
    Alias /cms/ /opt/wordpress/
    <Directory /opt/wordpress/>
        AllowOverride all
        Require all granted
    </Directory>
    <Location /cms/wp-json/>
        Deny from  all
    </Location>
</VirtualHost>
```

On CentOS `selinux` needs to be set to persive (or configured properly):

```bash
setenforce permissive
```

Start the Apache server:

```bash
sudo systemctl restart apache2  # on Debian/Ubuntu
sudo systemctl restart httpd    # on RHEL/CentOS
```

Your Daiquiri app should now be available on the configured virtual host, but again, without the static assets, like CSS files and images.

### Static assets

As you can see from the virtual host configurations, the static assets such as CSS and JavaScript files are served independently from the WSGI-python script. In order to do so, they need to be gathered in the `static_root` directory. This can be achieved by running:

```bash
python manage.py collectstatic
```

in your virtual environment. The Apache user needs to have read permissions to `/srv/daiquiri/app/static_root/`.

In order to apply changes to the code, the `daiquiri-app` job needs to be reloaded.

### Asyncronous workers

In production, and especially if you intent to run several Daiquiri applications on the same RabbitMQ instance, it is recomended to use [virtual hosts] and users with RabbitMQ. In order to create a virtual host and a user use the following commands on your RabbitMQ host:

```bash
# first enable the managemetn interface and crate an admin user
rabbitmq-plugins enable rabbitmq_management
rabbitmqctl add_user admin <a secret password>
rabbitmqctl set_user_tags admin administrator

# then create a vhost and a user for your daiquiri app
rabbitmqctl add_vhost <vhost>
rabbitmqctl add_user <user> <a secret password>
rabbitmqctl set_permissions -p <user> <user> ".*" ".*" ".*"
rabbitmqctl set_permissions -p <user> admin ".*" ".*" ".*"
```

Then add `CELERY_BROKER_URL` to the `.env` file of your Daiquiri application:

```bash
CELERY_BROKER_URL=amqp://<user>:<password>@<host>:<port>/<vhost>
```

As with the Gunicorn process, the asyncronous workers are also using systemd to launch and keep running. Create the following service files for the three workers:

```
# in /etc/systemd/system/daiquiri-default-worker.service
[Unit]
Description=celery worker for the default queue
PartOf=daiquiri.service
After=daiquiri.service

[Service]
Type=forking
User=daiquiri
Group=daiquiri

WorkingDirectory=/srv/daiquiri/app
EnvironmentFile=/srv/daiquiri/app/.env

Environment=CELERY_BIN=/srv/daiquiri/app/env/bin/celery
Environment=CELERYD_NODE=daiquiri_default
Environment=CELERYD_QUEUE=default
Environment=CELERYD_CONCURRENCY=1
Environment=CELERYD_PID_FILE=/var/run/celery/daiquiri/default.pid
Environment=CELERYD_LOG_FILE=/var/log/celery/daiquiri/default.log
Environment=CELERYD_LOG_LEVEL=INFO

ExecStart=/bin/sh -c '${CELERY_BIN} multi start ${CELERYD_NODE} \
  -A config \
  -Q ${CELERYD_QUEUE} \
  -c ${CELERYD_CONCURRENCY} \
  --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} \
  --loglevel=${CELERYD_LOG_LEVEL}'

ExecStop=/bin/sh -c '${CELERY_BIN} multi stopwait ${CELERYD_NODE} \
  --pidfile=${CELERYD_PID_FILE}'

ExecReload=/bin/sh -c '${CELERY_BIN} multi restart ${CELERYD_NODE} \
  -A config \
  -Q ${CELERYD_QUEUE} \
  -c ${ELERYD_CONCURRENCY} \
  --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} \
  --loglevel=${CELERYD_LOG_LEVEL}'

[Install]
WantedBy=daiquiri.service
```

```
# in /etc/systemd/system/daiquiri-query-worker.service
[Unit]
Description=celery worker for the query queue
PartOf=daiquiri.service
After=daiquiri.service

[Service]
Type=forking
User=daiquiri
Group=daiquiri

WorkingDirectory=/srv/daiquiri/app
EnvironmentFile=/srv/daiquiri/app/.env

Environment=CELERY_BIN=/srv/daiquiri/app/env/bin/celery
Environment=CELERYD_NODE=daiquiri_query
Environment=CELERYD_QUEUE=query
Environment=CELERYD_CONCURRENCY=1
Environment=CELERYD_PID_FILE=/var/run/celery/daiquiri/query.pid
Environment=CELERYD_LOG_FILE=/var/log/celery/daiquiri/query.log
Environment=CELERYD_LOG_LEVEL=INFO

ExecStart=/bin/sh -c '${CELERY_BIN} multi start ${CELERYD_NODE} \
  -A config \
  -Q ${CELERYD_QUEUE} \
  -c ${CELERYD_CONCURRENCY} \
  --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} \
  --loglevel=${CELERYD_LOG_LEVEL}'

ExecStop=/bin/sh -c '${CELERY_BIN} multi stopwait ${CELERYD_NODE} \
  --pidfile=${CELERYD_PID_FILE}'

ExecReload=/bin/sh -c '${CELERY_BIN} multi restart ${CELERYD_NODE} \
  -A config \
  -Q ${CELERYD_QUEUE} \
  -c ${ELERYD_CONCURRENCY} \
  --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} \
  --loglevel=${CELERYD_LOG_LEVEL}'

[Install]
WantedBy=daiquiri.service
```

```
# in /etc/systemd/system/daiquiri-download-worker.service
[Unit]
Description=celery worker for the download queue
PartOf=daiquiri.service
After=daiquiri.service

[Service]
Type=forking
User=daiquiri
Group=daiquiri

WorkingDirectory=/srv/daiquiri/app
EnvironmentFile=/srv/daiquiri/app/.env

Environment=CELERY_BIN=/srv/daiquiri/app/env/bin/celery
Environment=CELERYD_NODE=daiquiri_download
Environment=CELERYD_QUEUE=download
Environment=CELERYD_CONCURRENCY=1
Environment=CELERYD_PID_FILE=/var/run/celery/daiquiri/download.pid
Environment=CELERYD_LOG_FILE=/var/log/celery/daiquiri/download.log
Environment=CELERYD_LOG_LEVEL=INFO

ExecStart=/bin/sh -c '${CELERY_BIN} multi start ${CELERYD_NODE} \
  -A config \
  -Q ${CELERYD_QUEUE} \
  -c ${CELERYD_CONCURRENCY} \
  --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} \
  --loglevel=${CELERYD_LOG_LEVEL}'

ExecStop=/bin/sh -c '${CELERY_BIN} multi stopwait ${CELERYD_NODE} \
  --pidfile=${CELERYD_PID_FILE}'

ExecReload=/bin/sh -c '${CELERY_BIN} multi restart ${CELERYD_NODE} \
  -A config \
  -Q ${CELERYD_QUEUE} \
  -c ${CELERYD_CONCURRENCY} \
  --pidfile=${CELERYD_PID_FILE} \
  --logfile=${CELERYD_LOG_FILE} \
  --loglevel=${CELERYD_LOG_LEVEL}'

[Install]
WantedBy=daiquiri.service
```

Then, the worker can be started and enabled as before:

```bash
sudo systemctl daemon-reload

sudo systemctl start daiquiri-default-worker
sudo systemctl start daiquiri-query-worker
sudo systemctl start daiquiri-download-worker

sudo systemctl enable daiquiri-default-worker
sudo systemctl enable daiquiri-query-worker
sudo systemctl enable daiquiri-download-worker
```

Caching
-------

To use [memcached] as cache, first install it from the distribution:

```bash
apt install memcached  # Debian/Ubuntu
yum install memcached  # CentOS
```

On CentOS memcached needs to be restricted to listen only to localhost in `/etc/sysconfig/memcached`:

```
PORT="11211"
USER="memcached"
MAXCONN="1024"
CACHESIZE="64"
OPTIONS="-l 127.0.0.1,::1"
```

Then memcached can be enabled and started:

```bash
systemctl start memcached
systemctl enable memcached
```

[Apache2]: https://httpd.apache.org/
[NGINX]: https://www.nginx.com/
[wsgi]: https://docs.djangoproject.com/en/stable/howto/deployment/wsgi/
[mod_wsgi]: https://modwsgi.readthedocs.io/en/develop/
[mod_proxy]: https://httpd.apache.org/docs/2.4/mod/mod_proxy.html
[Gunicorn]: https://gunicorn.org/
[IUS repository]: https://ius.io/
[virtual hosts]: https://www.rabbitmq.com/vhosts.html
[tmpfiles.d]: https://www.freedesktop.org/software/systemd/man/tmpfiles.d.html
[WordPress.org]: https://wordpress.org/download/
[memcached]: https://memcached.org/
