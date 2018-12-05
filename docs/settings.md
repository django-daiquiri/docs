Settings
========

A Daiquiri application can be customised using various settings. Since Daiquiri is based on Django, we use its [build-in settings system](https://docs.djangoproject.com/en/2.1/topics/settings/). Almost every setting has a default value, which is set in the `daiquiri.core.settings` module ([Code on github](https://github.com/aipescience/django-daiquiri/tree/master/daiquiri/core/settings)). All settings can be changed for your particular `app` in `config/settings/base.py` (app specific) or `config/settings/local.py` (machine specific). Most of the settings can and should be customized in this way. In particular, the following settings can be overridden.


Django settings
---------------

#### DEBUG

Default: `False`

Debug mode.

See also [DEBUG](https://docs.djangoproject.com/en/2.1/ref/settings/#std:setting-DEBUG) in the Django documentation.

#### SECRET_KEY

Secret key for Django.

See also [SECRET_KEY](https://docs.djangoproject.com/en/2.1/ref/settings/#std:setting-SECRET_KEY) in the Django documentation.

#### ALLOWED_HOSTS

Default: `['localhost']`

List of allowed hosts for this app.

See also [ALLOWED_HOSTS](https://docs.djangoproject.com/en/2.1/ref/settings/#std:setting-ALLOWED_HOSTS) in the Django documentation.

#### TIME_ZONE

Default: `'UTC'`

#### EMAIL_BACKEND

Default: `'django.core.mail.backends.console.EmailBackend'`

Sets the backend used for email delivery. On a production system, this will be `django.core.mail.backends.smtp.EmailBackend`. For testing/demonstration `django.core.mail.backends.console.EmailBackend` can be used.

See also [Sending email](https://docs.djangoproject.com/en/2.1/topics/email/) in the Django documentation.

#### EMAIL_HOST

Default: `'localhost'`

Hostname of the SMTP server.

#### EMAIL_PORT

Default: `'25'`

Port of the SMTP server.

#### EMAIL_HOST_USER

Default: `''`

User for the SMTP server.

#### EMAIL_HOST_PASSWORD

Default: `''`

Password for the SMTP server.

#### EMAIL_USE_TLS

Default: `False`

Designates whether a `STARTTLS` is used (usually on port 587).

#### EMAIL_USE_SSL

Default: `False`

Designates whether a implicit TLS connection is used (usually on port 465).

#### DEFAULT_FROM_EMAIL

Default: `''`

Sets the FROM field for the E-mails send.

#### LOGGING_DIR

Default: `os.path.join(BASE_DIR, 'log')`

The location of the log files produced by Daiquiri. On a production system `/var/log/daiquiri/` might be a good choice.


django-allauth settings
-----------------------

#### ACCOUNT_LOGOUT_ON_GET

Default: False

Designates if a GET request is sufficient to logout. The seeting needs to be set to `TRUE` if the WordPress integration is used.

#### ACCOUNT_USERNAME_MIN_LENGTH

Default: 4

The minimal length of usernames.

#### ACCOUNT_PASSWORD_MIN_LENGTH

Default: 4

The minimal length of passwords.

#### ACCOUNT_EMAIL_VERIFICATION

Default: `'mandatory'`

Designates if new users need to verify their email addresses before logging in. Options are

* `'mandatory'`
* `'optional'`
* `'none'`

See also [ACCOUNT_EMAIL_VERIFICATION](https://django-allauth.readthedocs.io/en/latest/configuration.html) in the django-allauth documentation.

#### ACCOUNT_LOGIN_ON_EMAIL_CONFIRMATION

Default: `True`

Designates if new users are automatically logged in after validating their email address.


Django REST framework settings
------------------------------

#### REST_FRAMEWORK

Default:

```
{
    'DEFAULT_THROTTLE_CLASSES': (
        'rest_framework.throttling.ScopedRateThrottle',
    ),
    'DEFAULT_THROTTLE_RATES': {
        'query.create': '10/second'
    }
}
```

Configuration object for the Django REST framework. Used to adjust the maximum rate in which queries are allowed to be submitted (by anyone).

See also [Throttling](https://www.django-rest-framework.org/api-guide/throttling/) in the Django REST framework documentation.


django-sendfile settings
------------------------

#### SENDFILE_BACKEND

Default: `'sendfile.backends.simple'`

Sets the backend used for the sendfile integration to deliver static files. On a production system (using Apache), this will be `sendfile.backends.xsendfile`. For testing/demonstration `sendfile.backends.simple` can be used.

See also [Django Sendfile](https://github.com/johnsensible/django-sendfile) GitHub readme.


Daiquiri settings
-----------------

#### ASYNC

Default: `False`

Designates if celery workers are used for asyncronous tasks like query execution. If set to `True`, three workers need to run: `default`, `query`, and `download`.

#### IPV4_PRIVACY_MASK

Default: `16`

Number of bits kept from IPv4 addresses for anonymity.

#### IPV6_PRIVACY_MASK

Default: `32`

Number of bits kept from IPv6 addresses for anonymity.


Daiquiri archive settings
-------------------------

#### ARCHIVE_ANONYMOUS

Default: `False`

Designates if the archive interface can be accessed by anonymus users.

#### ARCHIVE_SCHEMA

Default: `'daiquiri_archive'`

Sets the schema for the archive in the `data` database.

#### ARCHIVE_TABLE

Default: `'files'`

Sets the table for the archive in the schema set by `ARCHIVE_SCHEMA` the `data` database.

#### ARCHIVE_COLUMNS

Default:

```python
[
    {
        'name': 'id',
        'hidden': True
    },
    {
        'name': 'timestamp',
        'label': 'Timestamp'
    },
    {
        'name': 'file',
        'label': 'Filename',
        'ucd': 'meta.file'
    },
    {
        'name': 'collection',
        'hidden': True
    },
    {
        'name': 'path',
        'hidden': True
    }
]
```

Sets the additional columns for the archive in the table set by `ARCHIVE_TABLE` the `data` database.

#### ARCHIVE_BASE_PATH

Default: `os.path.join(BASE_DIR, 'files')`

Sets the absolute base path of the files served by the archive module in the local file system.

#### ARCHIVE_DOWNLOAD_DIR

Default: `os.path.join(BASE_DIR, 'download')`

Sets the absolute path where the zip files for downloads from the archive are located in the local file system.


Daiquiri auth settings
----------------------

#### AUTH_SIGNUP

Default: `False`

Designates if users can register for an account. If set to false, all users need to be created through the Django admin system.

#### AUTH_WORKFLOW

Default: `None`

Sets the workflow for user registration. Options are

* `None`: Newly registered users can log in after registartion.
* `'activation'`: Newly registered users need to be activated by a manager or admin.
* `'confirmation'`: Newly registered users need to be confirmed by a manager before they are activated by an admin.

#### AUTH_DETAIL_KEYS

Default: `[]`

Sets additional details to be asked from the users when registering. An example would be:

```python
AUTH_DETAIL_KEYS = [
    {
        'key': 'affiliation',
        'label': 'Affiliation',
        'data_type': 'text',
        'required': True,
        'options': []
    },

]
```

where:

* `key` is the internal identifier,
* `label` the text shown in the interface,
* `datatype` the type of the detail which affects the widget to be used (`'text'`, `'textarea'`, `'select'`, `'radio'`, `'multiselect'`, or `'checkbox'`),
* `required` whether this detail is required or not, and
* `options` a list of options for select, radio or checkboxes widgets of the form `[{'id': id, 'label': label}, ...]`.

#### AUTH_TERMS_OF_USE

Default: `False`

Designates whether terms of use are displayed on the signup page and need to be accepted to register.


Daiquiri cutout settings
------------------------

#### CONESEARCH_ADAPTER

Default: `'daiquiri.conesearch.adapter.SimpleConeSearchAdapter'`

Sets the adapter class to be used by the cone search api. The adapter class encapsulated all operations creating a cone search votable output from the api request. Custom adapter need to enherit from `daiquiri.conesearch.adapter.BaseConeSearchAdapter`.

#### CONESEARCH_ANONYMOUS

Default: `False`

Designates if cone searches can be done by by anonymus users.


Daiquiri cutout settings
------------------------

#### CUTOUT_ADAPTER

Default: `'daiquiri.cutout.adapter.SimpleCutOutAdapter'`

Sets the adapter class to be used by the cutout api. The adapter class encapsulated all operations creating a cut out from the api request. Custom adapter need to enherit from `daiquiri.cutout.adapter.BaseCutOutAdapter`.

#### CUTOUT_ANONYMOUS

Default: `False`

Designates if the cutout interface can be accessed by anonymus users.


Daiquiri files settings
-----------------------

#### FILES_BASE_PATH

Default: `os.path.join(BASE_DIR, 'files')`

Sets the absolute base path of the files served by the files module in the local file system.

#### FILES_BASE_URL

Default: `None`

Sets the absolute URL for this Daiquiri app to be prepended to file references when downloading query results. This is done for columns with the `meta.ref;meta.file`, `meta.ref;meta.image` or `meta.ref;meta.note` UCD.


Daiquiri meetings settings
--------------------------

#### MEETINGS_CONTRIBUTION_TYPES

Default:

```python
[
    ('talk', _('Talk')),
    ('poster', _('Poster'))
]
```

Sets the types of contributions to be selected by partcipants when registering.

#### MEETINGS_PAYMENT_CHOICES

Default:

```python
[
    ('cash', _('cash')),
    ('wire', _('wire transfer')),
]
```

Sets the different payment choices to be selected by partcipants when registering.

#### MEETINGS_PARTICIPANT_DETAIL_KEYS

Default: `[]`

Sets additional details to be asked from the participants when registering. An example would be:

```python
MEETINGS_PARTICIPANT_DETAIL_KEYS = [
    {
        'key': 'affiliation',
        'label': 'Affiliation',
        'data_type': 'text',
        'required': True
    },
    {
        'key': 'dinner',
        'label': 'Conference dinner',
        'data_type': 'radio',
        'required': True,
        'options': [
            {'id': 'yes', 'label': 'yes'},
            {'id': 'no', 'label': 'no'}
        ]
    }
]
```

where:

* `key` is the internal identifier,
* `label` the text shown in the interface,
* `datatype` the type of the detail which affects the widget to be used (`'text'`, `'textarea'`, `'select'`, `'radio'`, `'multiselect'`, or `'checkbox'`),
* `required` whether this detail is required or not, and
* `options` a list of options for select, radio or checkboxes widgets of the form `[{'id': id, 'label': label}, ...]`.

#### MEETINGS_ABSTRACT_MAX_LENGTH

Default: `2000`

Sets the maximum lenght of an abstract.


Daiquiri metadata settings
--------------------------

#### METADATA_COLUMN_PERMISSIONS

Default: `False`

Designates if permissions can be assigned to individual columns (in addition to tables and schemas). *This is an experimental feature.*

#### METADATA_BASE_URL

Default: `None`

Sets the absolute URL of the metadata module, e.g. http://example.com/metadata/. The URL is used to create links to the landing pages for schemas and tables in VOTables if a DOI is not set.


Daiquiri qyery settings
--------------------------

#### QUERY_ANONYMOUS

Default: `False`

Designates if the query interface can be accessed by anonymus users. The permissions on schemas and tables need to be configured using the metadata interface.

#### QUERY_USER_SCHEMA_PREFIX

Default: `'daiquiri_user_'`

Sets the prefix for user schemas in the `data` database. Each user has a private schema where the result table of successful queries are stored.

#### QUERY_QUOTA

Default:

```python
{
    'anonymous': '100Mb',
    'user': '10000Mb',
    'users': {},
    'groups': {}
}
```

Sets the maximum quota for tables in a user's personal schema. The quota need to be set for the `anonymous` user as well as regular loggen in users (`user`). Additionally, users or groups can be asigned individual quotas, e.g.:

```python
{
    'anonymous': '100Mb',
    'user': '10000Mb',
    'users': {
        'admin': '1000Gb'
    },
    'groups': {
        'collab': '100Gb'
    }
}
```

If more than one quota applies, the maximum is used.

#### QUERY_SYNC_TIMEOUT

Default: `5`

Sets the timeout for syncronous (TAP) queries in seconds.

#### QUERY_MAX_ACTIVE_JOBS

Default:

```python
{
    'anonymous': '1'
}
```

Sets the maximum of simultanous jobs for users. The setting work analog to 'QUERY_QUOTA'. If more than one maximum applies, the maximum is used. If no maximum is given, like the default setting for logged users, no maximum is enforced.

#### QUERY_QUEUES

Default:

```python
[
    {
        'key': 'default',
        'label': 'Default',
        'timeout': 10,
        'priority': 1,
        'access_level': 'PUBLIC',
        'groups': []
    }
]
```

Set the different queue, which can be selected by the users. Each queue is represented by a dictionary where:

* `key` is the internal identifier,
* `label` is the text shown in the interface,
* `timeout` the maximum excecution time in seconds,
* `priority` is a integer number indicationg the priority in which jobs are executed, and
* `access_level` and `groups` the usual restiction on who can use this queue.

Jobs in a queue with *higher* priority are selected first when a query worker becomes available.

#### QUERY_LANGUAGES

Default:

```python
[
    {
        'key': 'adql',
        'version': 2.0,
        'label': 'ADQL',
        'description': '',
        'quote_char': '"'
    }
]
```

Sets the different query languages, which can be selected by the users. Each query language is represented by a dictionary where:

* `key` is the internal identifier,
* `label` is the text shown in the interface,
* `description` is additional information shown, e.g. in TopCat, and
* `quote_char` the character used by the database system to quote identifiers.

Already implemented in Daiquiri are `adql`, `postgresql`, and `mysql`. Apart from ADQL, the query language must match the database system of the `data` database.

#### QUERY_FORMS

Default:

```python
[
    {
        'key': 'sql',
        'label': 'SQL query',
        'service': 'query/js/forms/sql.js',
        'template': 'query/query_form_sql.html'
    }
]
```

Sets the form available for the users in the query interface. Each form is represented by a dictionary where:

* `key` is the internal identifier,
* `label` is the text shown in the interface,
* `service` the path AngularJS service containing the client side logic, and
* `template` the path to the Django template with the markup for the form.

Included in Daiquiri are `sql` and (simplyfied) `cone` and `box` services.

#### QUERY_DROPDOWNS

Default:

```python
[
    {
        'key': 'simbad',
        'service': 'query/js/dropdowns/simbad.js',
        'template': 'query/query_dropdown_simbad.html',
        'options': {
            'url': 'http://simbad.u-strasbg.fr/simbad/sim-id'
        }
    },
    {
        'key': 'vizier',
        'service': 'query/js/dropdowns/vizier.js',
        'template': 'query/query_dropdown_vizier.html',
        'options': {
            'url': 'http://vizier.u-strasbg.fr/viz-bin/votable',
            'catalogs': ['I/322A', 'I/259']
        }
    }
]
```

Sets the additional drop down menus above the SQL query interface available for the users. Each drop down is represented by a dictionary where:

* `key` is the internal identifier,
* `service` the path AngularJS service containing the client side logic,
* `template` the path to the Django template with the markup, and
* `options` additional options specific to the drop down.

Included in Daiquiri are `simbad` and `vizier` services.

#### QUERY_DOWNLOAD_DIR

Default: `os.path.join(BASE_DIR, 'download')`

Sets the absolute base path of the download files served by the query module in the local file system.

#### QUERY_DEFAULT_DOWNLOAD_FORMAT

Default: `'votable'`

Sets the default download format.

#### QUERY_DOWNLOAD_FORMATS

Default:

```python
[
    {
        'key': 'votable',
        'extension': 'xml',
        'content_type': 'application/xml',
        'label': 'IVOA VOTable XML file - TABLEDATA serialization',
        'help': 'A XML file using the IVOA VOTable format. Use this option if you intend to use VO compatible software to further process the data.'
    },
    {
        'key': 'csv',
        'extension': 'csv',
        'content_type': 'text/csv',
        'label': 'Comma separated Values',
        'help': 'A text file with a line for each row of the table. The fields are delimited by a comma and quoted by double quotes. Use this option for a later import into a spreadsheed application or a custom script. Use this option if you are unsure what to use.'
    },
    {
        'key': 'fits',
        'extension': 'fits',
        'content_type': 'application/fits',
        'label': 'FITS',
        'help': 'Flexible Image Transport System (FITS) file format.'
    }
]
```

Sets the available default download formats. Each format is represented by a dictionary where:

* `key` is the internal identifier,
* `extension` the file extension,
* `content_type` the content type,
* `label` the text shown in the interface, and
* `help` a more verbose help text for the format.


Daiquiri serve settings
-----------------------

#### SERVE_DOWNLOAD_DIR

Default: `os.path.join(BASE_DIR, 'download')`

Sets the absolute base path of the download files served by the serve module in the local file system.

#### SERVE_RESOLVER

Default: `None`

Sets the a resolver class to handle references in tables served by the serve module.


Daiquiri stats settings
-----------------------

#### STATS_RESOURCE_TYPES

Default:

```python
[
    {
        'key': 'ARCHIVE_DOWNLOAD',
        'label': 'Archive downloads'
    },
    {
        'key': 'CONESEARCH',
        'label': 'Performed cone searches'
    },
    {
        'key': 'CUTOUT',
        'label': 'Performed cutouts'
    },
    {
        'key': 'FILE',
        'label': 'File downloads'
    },
    {
        'key': 'QUERY',
        'label': 'Queries'
    }
]
```

Sets the aggegated stats shown in the stats management overview.


Daiquiri uws settings
-----------------------

#### UWS_RESOURCES

Default: `[]`

Configures UWS services (in addition to the TAP module). E.g.:

```python
{
    'prefix': r'query',
    'viewset': 'daiquiri.query.viewsets.UWSQueryJobViewSet',
    'base_name': 'uws_query'
}
```

where:

* `prefix` is part of the base URL of the service (e.g. http://example.com/usw/query),
* `viewset` is the ViewSet class handling the requests (neets to enherit from `daiquiri.jobs.viewsets.AsyncJobViewSet`), and
* `base_name` a basename needed for the integration in the Django REST framework.


Daiquiri tap settings
-----------------------

#### TAP_SCHEMA

Default: `'TAP_SCHEMA'`

Sets the name of the TAP schema. If more than one Daiquiri application is using the same `data` database, they cannot use the same TAP schema. Therefore the schema name TAP_SCHEMA is replaced before queries are submitted.


Daiquiri wordpress settings
--------------------------

#### WORDPRESS_URL

Default: `'/cms/'`

Sets the base url of the WordPress instance.

#### WORDPRESS_CLI

Default: `'/opt/wp-cli/wp'`

Sets the location of the `wp-cli` script on the local file system.

#### WORDPRESS_PATH

Default: `'/opt/wordpress'`

Sets the location of the WordPress installation on the local file system.
