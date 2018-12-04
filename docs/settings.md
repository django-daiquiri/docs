Settings
========

A Daiquiri application can be customised using various settings. Since Daiquiri is based on Django, we use its [build-in settings system](https://docs.djangoproject.com/en/2.1/topics/settings/). Almost every setting has a default value, which is set in the `daiquiri.core.settings` module ([Code on github](https://github.com/aipescience/django-daiquiri/tree/master/daiquiri/core/settings)). All settings can be changed for your particular `app` in `config/settings/base.py` (app specific) or `config/settings/local.py` (machine specific). Most of the settings can and should be customized in this way. In particular, the following settings can be overridden:


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

#### ARCHIVE_ANONYMOUS

Default: `False`

####ARCHIVE_SCHEMA

Default: `'daiquiri_archive'`

#### ARCHIVE_TABLE

Default: `'files'`

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

#### ARCHIVE_BASE_PATH

Default: `os.path.join(BASE_DIR, 'files')`

#### ARCHIVE_DOWNLOAD_DIR

Default: `os.path.join(BASE_DIR, 'download')`

#### AUTH_SIGNUP

Default: `False`

#### AUTH_WORKFLOW

Default: `None`

#### AUTH_DETAIL_KEYS

Default: `[]`

#### AUTH_TERMS_OF_USE

Default: `False`

#### CONESEARCH_ADAPTER

Default: `'daiquiri.conesearch.adapter.SimpleConeSearchAdapter'`

#### CONESEARCH_ANONYMOUS

Default: `False`

#### CUTOUT_ADAPTER

Default: `'daiquiri.cutout.adapter.SimpleCutOutAdapter'`

#### CUTOUT_ANONYMOUS

Default: `False`

#### FILES_BASE_PATH

Default: `os.path.join(BASE_DIR, 'files')`

#### FILES_BASE_URL

Default: `None`

#### MEETINGS_CONTRIBUTION_TYPES

Default:

```python
[
    ('talk', _('Talk')),
    ('poster', _('Poster'))
]
```

#### MEETINGS_PAYMENT_CHOICES

Default:

```python
[
    ('cash', _('cash')),
    ('wire', _('wire transfer')),
]
```

#### MEETINGS_PARTICIPANT_DETAIL_KEYS

Default: `[]`

#### MEETINGS_ABSTRACT_MAX_LENGTH

Default: `2000`

#### METADATA_COLUMN_PERMISSIONS

Default: `False`

#### METADATA_BASE_URL

Default: `None`

#### QUERY_ANONYMOUS

Default: `False`

#### QUERY_USER_SCHEMA_PREFIX

Default: `'daiquiri_user_'`

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

#### QUERY_SYNC_TIMEOUT

Default: `5`

#### QUERY_MAX_ACTIVE_JOBS

Default:

```python
{
    'anonymous': '1'
}
```

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

#### QUERY_DOWNLOAD_DIR

Default: `os.path.join(BASE_DIR, 'download')`

#### QUERY_DEFAULT_DOWNLOAD_FORMAT

Default: `'votable'`

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

#### SERVE_DOWNLOAD_DIR

Default: `os.path.join(BASE_DIR, 'download')`


#### SERVE_RESOLVER

Default: `None`

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

#### UWS_RESOURCES

Default: `[]`

#### TAP_SCHEMA

Default: `'TAP_SCHEMA'`

#### WORDPRESS_URL

Default: `'/cms/'`

#### WORDPRESS_CLI

Default: `'/opt/wp-cli/wp'`

#### WORDPRESS_PATH

Default: `'/opt/wordpress'`
