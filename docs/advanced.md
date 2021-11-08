Advanced features
=================

Apache2 and mod_wsgi
--------------------

This is an alternative deployment without the need of systemd scripts for the web application. Any asyncronous workers, however, would still need these systemd scripts.

The first thing, you have to do, is to uncomment:

```python
from dotenv import load_dotenv
load_dotenv()
```

in `config/wsgi.py` in your `app`.

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


Wagtail integration
-------------------

Since [Wagtail](https://wagtail.io) is a Django-base CMS, it can be used together with Daiquiri easily. The main steps are explained [in the Wagtail documentation](https://docs.wagtail.io/en/stable/advanced_topics/add_to_django_project.html). Here we assume that the customization to daiquiri are located in a Django app called `daiquiri_app` in your `daiquiri-app` directory. The file layout should be like this:

```plain
├── config
├── daiquiri_app
│   ├── __init__.py
│   ├── migrations
│   ├── models.py
│   ├── static
│   └── templates
├── manage.py
├── media_root
├── static_root
└── vendor
```

To enable Wagtail, add the following blocks to `config/settings/base.py`:

```python
from . import MIDDLEWARE

...

INSTALLED_APPS = DJANGO_APPS + [
    'daiquiri_app',

    ...

    'wagtail.contrib.forms',
    'wagtail.contrib.redirects',
    'wagtail.embeds',
    'wagtail.sites',
    'wagtail.users',
    'wagtail.snippets',
    'wagtail.documents',
    'wagtail.images',
    'wagtail.search',
    'wagtail.admin',
    'wagtail.core',

    'modelcluster',
    'taggit',
] + ADDITIONAL_APPS

...

MIDDLEWARE += [
    'wagtail.contrib.redirects.middleware.RedirectMiddleware'
]
```

This adds the Wagtail apps and a Wagtail-specific middleware. In `config/urls.py` add:

```python
from django.conf.urls.static import static

...

from wagtail.admin import urls as wagtailadmin_urls
from wagtail.core import urls as wagtail_urls

urlpatterns = [
    ...

    path('wagtail/', include(wagtailadmin_urls)),
    path('cms/', include(wagtail_urls)),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

This adds the Wagtail backend at `/wagtail/` and the content at `/cms/`. You can also use:

```python
    path('', include(wagtail_urls)),
```

to create the home page with Wagtail as well. In this case, it needs to be the last entry, since otherwise it would override all other urls.

After this Wagtail pages can be created as explained in the Wagtail documentation, starting [here](https://docs.wagtail.io/en/stable/topics/pages.html). In our example the page models would reside in `daiquiri_app/models.py` and the corresponding templates in `daiquiri_app/templates/daiquiri_app`.

Initially, Wagtail creates a home page model without content, which of little use. The first step is usually to remove this Page and Site and create a new Site with a different `HomePage` model. This can be done automatically with the following data migration (e.g. for the app `daiquiri_app`).

```python
# -*- coding: utf-8 -*-
from django.db import migrations


def data_migration(apps, schema_editor):
    ContentType = apps.get_model('contenttypes.ContentType')
    Page = apps.get_model('wagtailcore.Page')
    Site = apps.get_model('wagtailcore.Site')
    HomePage = apps.get_model('daiquiri_app.HomePage')

    Page.objects.get(slug='home').delete()

    # Create content type for homepage model
    content_type, created = ContentType.objects.get_or_create(model='homepage', app_label='daiquiri_app')

    # Create a new homepage
    homepage = HomePage.objects.create(title='CosmoSim', slug='home', content_type=content_type,
                                       path='00010001', depth=2, numchild=0, url_path='/', locale_id=1)

    # Create a site with the new homepage set as the root
    Site.objects.create(hostname='localhost', root_page=homepage, is_default_site=True)


class Migration(migrations.Migration):

    dependencies = [
        ('wagtailcore', '0002_initial_data'),
        ('daiquiri_app', '0001_initial'),
    ]

    operations = [
        migrations.RunPython(data_migration),
    ]
```


WordPress integration
---------------------

WordPress can be used as CMS for the documentation and/or the presentation of the project as well. Since WordPress uses a separate technology stack, its integration is much more complicated. For new projects we suggest to use Wagtail. WordPress is mainly used for existing Daiquiri sites.

For this documentation, we assume WordPress will be installed `/opt/wordpress`. First, you need to install PHP:

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


Double reverse proxy setup
--------------------------

When running Daiquiri behind two reverse proxy servers, one as generic HTTP proxy as entrypoint to the local network and one to combine Gunicorn and Static assets on one application server, the follow setup can be used:

```bash
Internet -> HTTP proxy (192.168.0.10) -> Application server (192.168.0.20) -> Gunicorn
                                                                           -> Static assets
```

The virtual host configuration on the HTTP Proxy looks like this:

```apache
<VirtualHost *:443>
    ...

    ProxyPass / http://app.local/
    ProxyPassReverse / app.local/

    <Location />
        RequestHeader set X-Forwarded-Proto 'https' env=HTTPS
    </Location>
</VirtualHost>
```

On the application server the virtual host configuration is this:

```apache
<VirtualHost *:80>
    ...

    SSLEngine on
    SSLCertificateFile ...
    SSLCertificateKeyFile ...
    SSLCACertificateFile ...
    SSLProtocol All -SSLv2 -SSLv3
    SSLCipherSuite 'EDH+CAMELLIA:EDH+aRSA:EECDH+aRSA+AESGCM:EECDH+aRSA+SHA384:\
        EECDH+aRSA+SHA256:EECDH:+CAMELLIA256:+AES256:+CAMELLIA128:+AES128:\
        +SSLv3:!aNULL:!eNULL:!LOW:!3DES:!MD5:!EXP:!PSK:!DSS:!RC4:!SEED:\
        !ECDSA:CAMELLIA256-SHA:AES256-SHA:CAMELLIA128-SHA:AES128-SHA'

    RemoteIPHeader X-Forwarded-For
    RemoteIPTrustedProxy 192.168.0.10/32

    Alias /static/ /srv/daiquiri/app/static_root/
    <Directory /srv/daiquiri/app/static_root/>
        Require all granted
    </Directory>

    Alias /cms/ /opt/wordpress/
    <Directory /opt/wordpress/>
        AllowOverride all
        Require all granted
    </Directory>

    ProxyPass /static !
    ProxyPass /cms !
    ProxyPass / http://localhost:9000/
    ProxyPassReverse / http://localhost:9000/

    <Location /cms/wp-json/>
        Deny from all
    </Location>
</VirtualHost>
```

The `RemoteIPHeader` implies that the `remoteip` module for Apache is installed and enabled. Up to date TLS/SSL settings can be found on [bettercrypto.org](https://bettercrypto.org).


Daiquiri client
---------------

[Daiquiri client](https://github.com/django-daiquiri/client) is a is a python library meant to be used with the Daiquiri Framework. It provides a set of functions which can be used to use the API of a Daiquiri powered website inside a script. The nessesarry HTTP requests are abstracted in a transparent way.

Daiquiri client can be installed using

```bash
pip install --upgrade https://github.com/django-daiquiri/client
```

A script for getting the emails of all users using Daiquiri Client could look like this:

```
from daiquiri_client import Client

client = Client(DAIQUIRI_URL, TOKEN)

for profile in client.auth.get_profiles():
    print(profile['user']['email'])
```

where DAIQUIRI_URL is the url of the Daiquiri site and TOKEN is your API token, which can be obtained from the Daiquiri site at the URL */accounts/token/*.

A commen use case for Daiquiri client is the update of the metadata of a database schema. For this, first add the schema manually using the metadate management and make sure `Automatically discover tables and columns` is checked. Then prepare a yaml file of the form:

```yaml
- name: daiquiri_data_obs
  title: Observational data
  description: Some observational data
  long_description: Some more information about the data.
  attribution: Please cite the following paper ...
  order: 1
  license: CC0
  doi: 10.1000/xyz123
  published: 2020-01-01
  updated: 2018-01-01
  access_level: PUBLIC
  metadata_access_level: PUBLIC
  creators:
  - name: Anna Admin
    first_name: Anna
    last_name: Admin
    orcid: https://orcid.org/0000-0001-2345-6789
    affiliations: "Institute of applied Administration\nInstitute of theoretical Managament"
  - name: Manni Manager
    orcid: https://orcid.org/0000-0001-2345-6790
    affiliations: Institute of theoretical Managament
  contributors:
  - name: Some computer guy

  tables:
  - name: stars
    title: Stars
    description: Some stars data
    order: 1
    license: CC0
    doi: 10.1000/xyz123/123
    published: 2020-01-01
    updated: 2018-01-01
    access_level: PUBLIC
    metadata_access_level: PUBLIC
    creators:
    - name: Anna Admin
      first_name: Anna
      last_name: Admin
      orcid: https://orcid.org/0000-0001-2345-6789
      affiliations:
      - Institute of applied Administration
      - Institute of theoretical Managament
    - name: Manni Manager
      orcid: https://orcid.org/0000-0001-2345-6790
      affiliations:
      - Institute of theoretical Managament

    columns:
    - name: id
      ucd: meta.id;meta.main
    - name: ra
      ucd: pos.eq.ra;meta.main
    - name: dec
      ucd: pos.eq.dec;meta.main
    - name: parallax
      ucd: pos.parallax
```

and a python script or notebook with the following content:

```python
import yaml
from daiquiri_client import Client

DAIQUIRI_URL = 'http://localhost:8000'
TOKEN = 'a35b0eb94ef906648445c9214bed30265af1062d'

with open('update_metadata.yml') as f:
    local_schemas = yaml.safe_load(f.read())

client = Client(DAIQUIRI_URL, TOKEN)

for remote_schema in client.metadata.get_schemas(nested=True):
    for local_schema in local_schemas:
        if remote_schema['name'] == local_schema['name']:
            client.metadata.update_schema(remote_schema['id'], local_schema)

            for remote_table in remote_schema['tables']:
                for local_table in local_schema['tables']:

                    if remote_table['name'] == local_table['name']:
                        client.metadata.update_table(remote_table['id'], local_table)

                        for remote_column in remote_table['columns']:
                            for local_column in local_table['columns']:

                                if remote_column['name'] == local_column['name']:
                                    client.metadata.update_column(remote_column['id'], local_column)
```

And execute it in a virtual environment where `daiquiri_client` is installed.
