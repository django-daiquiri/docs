# Appendix

## Deployment with Apache2 and mod_wsgi

This is an alternative deployment without the need of systemd scripts for the web application. Any asyncronous workers, however, would still need these systemd scripts.

Install the Apache server and `mod_wsgi` on Debian or Ubuntu using:

```bash
sudo apt install apache2 libapache2-mod-wsgi-py3
```

On CentOS 7 you need to enable the [IUS repository] first. Then install using:

```
sudo yum install httpd python35u-mod_wsgi
```

Then, edit the virtual host configuration:

```
# in /etc/httpd/sites-available/default  on Debian/Ubuntu
# in /etc/httpd/conf.d/vhost.conf        on RHEL/CentOS
<VirtualHost *:80>

    ...

    WSGIDaemonProcess daiquiri user=daiquiri group=daiquiri \
        home=/srv/daiquiri/app python-home=/srv/daiquiri/app/env
    WSGIProcessGroup daiquiri
    WSGIScriptAlias / /srv/daiquiri/app/config/wsgi.py process-group=daiquiri
    WSGIPassAuthorization On

    Alias /static /srv/daiquiri/app/static_root/
    <Directory /srv/daiquiri/app/static_root/>
        Require all granted
    </Directory>

    <Directory /srv/daiquiri/app/config/>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
</VirtualHost>
```

Start the Apache server:

```bash
# on Debian/Ubuntu
sudo systemctl start apache2
sudo systemctl enable apache2

# on RHEL/CentOS
sudo systemctl start httpd
sudo systemctl enable httpd
```

Your Daiquiri app should now be available on the configured virtual host, and the deployment can be continued as usual.


## WordPress integration

WordPress can be used as CMS for the documentation and/or the presentation of the project. For this documentation, we assume WordPress will be installed `/opt/wordpress`. First, you need to install PHP:

```bash
# Debian 10
sudo apt-get install php7.3 php7.3-mysql php-pear
pear install HTTP_Request2

# Ubuntu 18.04
sudo apt-get install php7.2 php7.2-mysql php-pear
pear install HTTP_Request2
```

The code needs to be downloaded from [WordPress.org] and extracted:

```bash
# as root
cd /opt
wget https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
```

The the [Daiquiri theme] and the [Daiquiri plugin] need to be installed:

```bash
# as root
cd /opt/wordpress/wp-content/themes
git clone https://github.com/django-daiquiri/wordpress-theme daiquiri

cd /opt/wordpress/wp-content/plugins
git clone https://github.com/django-daiquiri/wordpress-plugin daiquiri
```

The WordPress directory need to be owned by the Apache2 user:

```bash
chown -R www-data:www-data /opt/wordpress  # Debian/Ubuntu
chown -R apache:apache /opt/wordpress      # CentOS
```

Then, create the `wp-config.php` file

```bash
cp /opt/wordpress/wp-config-sample.php /opt/wordpress/wp-config.php
```

and edit the file for the database connection for WordPress (not the same as for Daiquiri, needs MySQL or MariaDB):

```
define( 'DB_NAME', 'database_name_here' );

define( 'DB_USER', 'username_here' );

define( 'DB_PASSWORD', 'password_here' );

define( 'DB_HOST', 'localhost' );
```

In addition, update the salt values and add the following at the end of the file, but before `That's all, stop editing!`:

```
define('DAIQUIRI_DEBUG', False);
define('DAIQUIRI_URL', 'https://<the url your daiquiri app>');

define('COOKIEPATH','/');
define('SITECOOKIEPATH',COOKIEPATH);
define('ADMIN_COOKIE_PATH',COOKIEPATH);
define('PLUGINS_COOKIE_PATH',COOKIEPATH);
```

The go to http://<your url>/cms/ and follow the WordPress instalation. You can use a username which you already registered in Daiquiri. After the installation log in into the WordPress backend. Then:

* Activate the Daiquiri theme under `Appearance -> Themes`
* Activate the Daiquiri plugin under `Appearance -> Plugins`

Logout of WordPress. Login to Daiquiri. You should now be logged in into WordPress as well. The syncronization of WordPress and Daiquiri users is done using [wp-cli](https://wp-cli.org/). See [settings](/settings/#daiquiriwordpresssettings) for more information about using WordPress with Daiquiri.
