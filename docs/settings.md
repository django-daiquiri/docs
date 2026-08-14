Settings
========

A Daiquiri application can be customised using various settings. Since Daiquiri is based on Django, we use its [built-in settings system](https://docs.djangoproject.com/en/5.2/topics/settings/). Almost every setting has a default value, which is set in the Daiquiri library. The core settings are defined in the `daiquiri.core.settings` module:

* [daiquiri.core.settings.django](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/core/settings/django.py)
* [daiquiri.core.settings.daiquiri](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/core/settings/daiquiri.py)
* [daiquiri.core.settings.logging](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/core/settings/logging.py)

Additional settings, which concern only a single Daiquiri module are defined in the `settings` module of the particular module:

* [daiquiri.auth.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/auth/settings.py)
* [daiquiri.conesearch.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/conesearch/settings.py)
* [daiquiri.contact.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/contact/settings.py)
* [daiquiri.cutout.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/cutout/settings.py)
* [daiquiri.files.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/files/settings.py)
* [daiquiri.metadata.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/metadata/settings.py)
* [daiquiri.oai.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/oai/settings.py)
* [daiquiri.query.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/query/settings.py)
* [daiquiri.serve.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/serve/settings.py)
* [daiquiri.stats.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/stats/settings.py)
* [daiquiri.tap.settings](https://github.com/django-daiquiri/daiquiri/blob/main/daiquiri/tap/settings.py)

All settings can be changed for your particular `app` in `config/settings/base.py` (app specific) or `config/settings/local.py` (machine specific, and ignored by git). In addition, selected settings can be configured as environment variables, in particular using the `.env` file in the `app` directory:

Setting        | Default
-------------- | -----------------------
SECRET_KEY     | `None` (must be supplied)
SITE_URL       | `None`
DEBUG          | `None`
ASYNC          | `None`
ALLOWED_HOSTS  | `['localhost', '127.0.0.1', '::1']`
TIME_ZONE      | `'UTC'`
PROXY          | `None`
ADMINS         | `[]`
DATABASE_APP   | `None` (must be supplied)
DATABASE_DATA  | `None` (must be supplied)
TAP_SCHEMA     | `tap_schema`
TAP_UPLOAD     | `tap_upload`
OAI_SCHEMA     | `oai_schema`

In the `.env` file, lists are separated by commas, e.g. `ADMINS=Anna Admin <admin@example.com>, Manni Manager <manager@example.com>`. Boolean values accept `1`, `t`, `true`, `y`, `yes`, and `on` for true; other non-empty values are false. Paths read from the environment are made absolute. Since `SECRET_KEY`, `DATABASE_APP`, and `DATABASE_DATA` have no usable default value, they need to be set.

`DATABASE_APP` and `DATABASE_DATA` are database URLs understood by
`dj-database-url`. The `default` database uses `DATABASE_APP`; the `data`,
`tap`, and `oai` aliases use `DATABASE_DATA`, with the TAP and OAI namespaces
selected by `TAP_SCHEMA`, `TAP_UPLOAD`, and `OAI_SCHEMA`. `PROXY=True` enables
the forwarded-host and forwarded-protocol handling needed behind a reverse
proxy.

In the following all settings, which can be changed from their default values to customize you particular Daiquiri app are described in detail:


daiquiri.core.settings.django
-----------------------------

#### SECRET_KEY

Default: `None`

Secret key for Django. It must be set in `.env` and must not be committed.

See also [SECRET_KEY](https://docs.djangoproject.com/en/5.2/ref/settings/#std-setting-SECRET_KEY) in the Django documentation.

---

#### DEBUG

Default: `None`

Debug mode. Set it to `False` in production. See also [DEBUG](https://docs.djangoproject.com/en/5.2/ref/settings/#std-setting-DEBUG) in the Django documentation. This setting can be set in `.env`.

---

#### BASE_URL

Default: `/`

Base URL for the Daiquiri app. Set if your Daiquiri app runs under an alias path on your web server, e.g. `/daiquiri/`. The value is normalized to include a trailing slash.

---

#### ALLOWED_HOSTS

Default: `['localhost', '127.0.0.1', '::1']`

List of allowed hosts for this app.

See also [ALLOWED_HOSTS](https://docs.djangoproject.com/en/5.2/ref/settings/#std-setting-ALLOWED_HOSTS) in the Django documentation. This setting can be set in `.env` as a comma-separated list.

---

#### INSTALLED_APPS

The list of Django apps for this Daiquiri site. Usually it is set to

```python
INSTALLED_APPS = DJANGO_APPS + [
    # the list of Daiquiri modules for this site
    ...
] + ADDITIONAL_APPS
```
See also [INSTALLED_APPS](https://docs.djangoproject.com/en/5.2/ref/settings/#std-setting-INSTALLED_APPS) in the Django documentation.

---

#### USER_TABLESPACE

Default: `'pg_default'` for PostgreSQL, otherwise `None`

The tablespace where to store the user query-job tables. This setting is only used for the PostgreSQL scientific database.

---

#### TIME_ZONE

Default: `'UTC'`

The time zone for this Daiquiri app (e.g. `Europe/Berlin`). It can be set in `.env`.

---

#### ACCOUNT_LOGOUT_ON_GET

Default: `True`

Designates if a GET request is sufficient to log out. It is enabled by the current source configuration.

---

#### ACCOUNT_USERNAME_MIN_LENGTH

Default: 4

The minimal length of usernames.

---

#### ACCOUNT_PASSWORD_MIN_LENGTH

Default: 8

The minimal length of passwords.

---

#### ACCOUNT_EMAIL_VERIFICATION

Default: `'mandatory'`

Designates if new users need to verify their email addresses before logging in. Options are

* `'mandatory'`
* `'optional'`
* `'none'`

See also [ACCOUNT_EMAIL_VERIFICATION](https://docs.allauth.org/en/latest/account/configuration.html) in the django-allauth documentation.

---

#### ACCOUNT_LOGIN_ON_EMAIL_CONFIRMATION

Default: `True`

Designates if new users are automatically logged in after validating their email address.

---

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

Configuration object for the Django REST framework. Used to adjust the maximum rate in which queries are allowed to be submitted (by anyone). See also [Throttling](https://www.django-rest-framework.org/api-guide/throttling/) in the Django REST framework documentation.

---

#### CELERY_BROKER_URL

Default: `amqp://`

URL of the RabbitMQ server to be used. For a more sophisticated setup use:

```
CELERY_BROKER_URL=amqp://<user>:<password>@<host>:<port>/<vhost>
```

See also [deployment](deployment.md#asyncronous-workers). This setting can be set in `.env`.

---

#### EMAIL_BACKEND

Default: `'django.core.mail.backends.console.EmailBackend'`

Sets the backend used for email delivery. On a production system, this will be `django.core.mail.backends.smtp.EmailBackend`. For testing/demonstration `django.core.mail.backends.console.EmailBackend` can be used. See also [Sending email](https://docs.djangoproject.com/en/5.2/topics/email/) in the Django documentation. This setting can be set in `.env`.

---

#### EMAIL_HOST

Default: `None`

Hostname of the SMTP server. Can be set in `.env`.

---

#### EMAIL_PORT

Default: `'25'`

Port of the SMTP server. Can be set in `.env`.

---

#### EMAIL_HOST_USER

Default: `None`

User for the SMTP server. Can be set in `.env`.

---

#### EMAIL_HOST_PASSWORD

Default: `None`

Password for the SMTP server. Can be set in `.env`.

---

#### EMAIL_USE_TLS

Default: `None`

Designates whether `STARTTLS` is used (usually on port 587). This setting can be set in `.env`.

---

#### DEFAULT_FROM_EMAIL

Default: `'info@example.com'`

Sets the `From` field for emails sent by Daiquiri. This setting can be set in `.env`.

---

#### SENDFILE_BACKEND

Default: `'django_sendfile.backends.simple'`

Sets the backend used for the sendfile integration to deliver static files. On a production system using Apache with `mod_xsendfile`, this can be `django_sendfile.backends.xsendfile`. The simple backend is suitable for testing and demonstration.

See also [Django Sendfile](https://github.com/johnsensible/django-sendfile) GitHub readme.

---

#### MEMCACHE_KEY_PREFIX

Default: `None`

Key prefix to use for caching with [memcached](https://memcached.org/). Enables caching with memcached. See also [deployment](deployment.md#caching). Should be set in `.env`.

---

daiquiri.core.settings.daiquiri
-------------------------------

#### ASYNC

Default: `None`

Designates if Celery workers are used for asynchronous tasks like query execution. The current non-query queues are `default` and `download`; query workers use the queues configured by `QUERY_QUEUES`, whose names are prefixed with `query_`.

---

#### IPV4_PRIVACY_MASK

Default: `16`

Number of bits kept from IPv4 addresses (e.g. to produce query stats) for anonymity.

---

#### IPV6_PRIVACY_MASK

Default: `32`

Number of bits kept from IPv6 addresses (e.g. to produce query stats) for anonymity.

---

#### SITE_URL

Default: `None`

Public URL of the Daiquiri site. Used for VO and OAI metadata.

---

#### SITE_IDENTIFIER

Default: `None`

Identifier for the Daiquiri site. Usually the URL without the protocol. Used for VO and OAI metadata.

---

#### SITE_TITLE

Default: `None`

The title for the Daiquiri site. Used for VO registry and service metadata.

---

#### SITE_DESCRIPTION

Default: `None`

The description for the Daiquiri site. This value is defined by the current
settings module but is not read directly by the current core VO/OAI serializers.

---

#### SITE_LICENSE

Default: `None`

A license identifier for the Daiquiri site. The available resource-license
constants are defined in `daiquiri.metadata.settings`; the current core
serializers use the resource license settings rather than reading this value
directly.

---

#### SITE_CREATOR

Default: `None`

Creator name of the Daiquiri site. It is used in VO curation metadata. The current source expects the creator name here; the logo is configured separately by the application.

```
'Daiquiri project'
```

---

#### SITE_CONTACT

Default: `None`

Contact information for the Daiquiri site. Used in the VO and OAI metadata. It has to be a dictionary of the following form:

```
{
    'name': 'Anna Admin',
    'address': 'Beispielstr. 3, 12345 Berlin',
    'email': 'contact@example.com',
    'telephone': '+49 30 123467'
}
```

---

#### SITE_PUBLISHER

Default: `None`

Publisher of the Daiquiri site. Used for VO and OAI metadata.

---

#### SITE_CREATED

Default: `None`

Date of the creation of the Daiquiri site. Used for VO and OAI metadata. Has to be of the form `YYYY-MM-DD`.

---

#### SITE_UPDATED

Default: `None`

Date of the last update of the Daiquiri site. Used for VO and OAI metadata. Has to be of the form `YYYY-MM-DD`.

---

#### SITE_SUBJECTS

Default:
```
[
    {
        'subject': 'Astronomy',
        'subjectScheme': 'Library of Congress Subject Headings (LCSH)',
        'schemeURI': 'http://id.loc.gov/authorities/subjects',
        'valueURI': 'http://id.loc.gov/authorities/subjects/sh85009003',
    }
]
```

Subjects for this Daiquiri app. Used for VO and OAI metadata.

---

#### SITE_TYPE

Default: `'service'`

Type for this Daiquiri site. Used in the VO registry entry.

---

daiquiri.core.settings.logging
------------------------------

#### LOG_LEVEL

Default: `None`

Level of logging. Set to `DEBUG` to get more verbose logging. Should be set in `.env`.

---

#### LOG_DIR

Default: `None`

Directory for log files. In development `log` is suitable, in production `/var/log/django/daiquiri/` might be a sensible choice. Note that this directory must exist for Daiquiri to work. Should be set in `.env`.

---

daiquiri.auth.settings
----------------------

#### AUTH_SIGNUP

Default: `None`

Designates if users can register for an account. If it is false, all users need to be created through the Django admin system. This setting is read from `.env`.

---

#### AUTH_WORKFLOW

Default: `None`

Sets the workflow for user registration. Options are

* `None`: Newly registered users can log in after registration.
* `'activation'`: Newly registered users need to be activated by a manager or admin.
* `'confirmation'`: Newly registered users need to be confirmed by a manager before they are activated by an admin.

---

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
* `data_type` is the type of the detail, which affects the widget to be used (`'text'`, `'textarea'`, `'select'`, `'radio'`, `'multiselect'`, or `'checkbox'`),
* `required` whether this detail is required or not, and
* `options` a list of options for select, radio or checkboxes widgets of the form `[{'id': id, 'label': label}, ...]`.

---

#### AUTH_TERMS_OF_USE

Default: `False`

Designates whether terms of use are displayed on the signup page and need to be accepted to register. The text to be displayed needs to be configured in a template `account/terms_of_use.html`.

---

daiquiri.conesearch.settings
------------------------

#### CONESEARCH_ADAPTER

Default: `'daiquiri.conesearch.adapter.BaseConeSearchAdapter'`

Sets the adapter class to be used by the cone search API. The adapter class encapsulates all operations for creating a cone-search VOTable output from an API request. A custom adapter should inherit from `daiquiri.conesearch.adapter.BaseConeSearchAdapter`.

---

#### CONESEARCH_RESOURCES

Default: `{}`

Defines which database resources are available for the
cone-search service. Keys must be "<schema.table>" identifiers. Each entry
contains:
- schema_name: schema where the table is located
- table_name:  table to query
- column_names: columns exposed in the cone search
- coordinates_columns: maps `RA`, `DEC`, and optionally other coordinate names to table columns

Example:
```
CONESEARCH_RESOURCES = {
        'schema_name.table_name': {
            'schema_name': '<SCHEMA_NAME>',
            'table_name': '<TABLE_NAME>',
            'column_names': ['source_id', 'ra', 'dec'],
            'coordinates_columns': {'RA': 'ra', 'DEC': 'dec'},
        },
        'schema_name.other_table': {
            'schema_name': '<SCHEMA_NAME>',
            'table_name': '<OTHER_TABLE_NAME>',
            'column_names': ['source_id', 'ra_component', 'dec_component'],
            'coordinates_columns': {'RA': 'ra_component', 'DEC': 'dec_component'},
        }
}
```
---

#### CONESEARCH_ANONYMOUS

Default: `False`

Designates if cone searches can be done by anonymous users.

---

#### CONESEARCH_SUBJECTS

Default: ['cone search']

Defines the list of subjects associated with cone-search functionality.

---

#### CONESEARCH_RANGES

Default: {
            'RA': {
                'min': 0,
                'max': 360
            },
            'DEC': {
                'min': -90,
                'max': 90
            },
            'SR': {
                'min': 0,
                'max': 10
            }
        }

Defines the maximum and minimum value for RA, DEC and SR.

---

#### CONESEARCH_DEFAULTS

Default: {
            'RA': 0.0,
            'DEC': 0.0,
            'SR': 1.0,
         }

Defines the default values for RA, DEC and SR.

---

daiquiri.contact.settings
------------------------

#### ANNOUNCEMENT_MESSAGE_FILTER

Default: `daiquiri.contact.filters.DefaultMessageFilter`

Sets the filter class to be used by announcement messages.
The filter class encapsulates the filter methods used to show announcements to a filtered set of users.
A custom filter class must inherit from `daiquiri.contact.filters.DefaultMessageFilter`.
All custom filter methods are required to have a single parameter `request`.

The announcement messages can be added in the Django admin site. In order to show
the messages on a webpage, one has to load the announcement tags in the template
`{% load announcement_tags %}` and then place `{% show_announcements %}` where
the messages should be shown.

---

daiquiri.cutout.settings
------------------------

#### CUTOUT_ADAPTER

Default: `'daiquiri.cutout.adapter.SimpleCutOutAdapter'`

Sets the adapter class to be used by the cutout API. The adapter class encapsulates all operations for creating a cutout from an API request. A custom adapter should inherit from `daiquiri.cutout.adapter.BaseCutOutAdapter`.

---

#### CUTOUT_ANONYMOUS

Default: `False`

Designates if the cutout interface can be accessed by anonymous users.

---

daiquiri.files.settings
-----------------------

#### FILES_BASE_PATH

Default: `None`

Sets the absolute base path of the files served by the files module in the local file system. Should be set in `.env`.

---

#### FILES_BASE_URL

Default: `None`

Sets the absolute URL for this Daiquiri app to be prepended to file references when downloading query results. This is done for columns with the `meta.ref;meta.file`, `meta.ref;meta.image` or `meta.ref;meta.note` UCD. Should be set in `.env`.

---

#### FILES_DOCS_REL_PATH

Default: `None`

Sets the relative file path to the documentation files on which the file search will be executed.
The path is given relative to the `FILES_BASE_PATH`.

In order to use the search function for the files, one has to include the search field
into the template.
```
{% include 'files/search-input.html' %}
```
The layout of the results page can be changed by overriding the template `files/search-results.html` in the Daiquiri app.

---

#### FILES_SEARCH_RESULTS_PER_PAGE

Default: `5`

Sets the number of search results per page.

---

daiquiri.metadata.settings
--------------------------

#### METADATA_COLUMN_PERMISSIONS

Default: `False`

Designates if permissions can be assigned to individual columns (in addition to tables and schemas). *This is an experimental feature.*

---

#### METADATA_BASE_URL

Default: `None`

Sets the absolute URL of the metadata module, e.g. http://example.com/metadata/. The URL is used to create links to the landing pages for schemas and tables in VOTables if a DOI is not set.

---

#### ARCHIVE_BASE_PATH

Default: `None`

Sets the absolute base path value for archive-related integrations. It is read
from the `ARCHIVE_BASE_PATH` environment variable. The setting remains defined
in `daiquiri.metadata.settings`, but the current Daiquiri source does not
provide a separate `daiquiri.archive` module or archive-specific consumers.

---

daiquiri.oai.settings
---------------------

#### OAI_SCHEMA

Default: `'oai_schema'`

Sets the name of the schema or database namespace for OAI records. If more than one Daiquiri application is using the same `data` database, they cannot use the same OAI schema. The default is `oai_schema`; this setting can be changed in `.env`.

---

#### OAI_ADAPTER

Default: `'daiquiri.oai.adapter.DefaultOaiAdapter'`

Sets the adapter class to be used by the OAI-PMH API. The adapter class encapsulates the operations used to set up and retrieve OAI records. A custom adapter should inherit from `daiquiri.oai.adapter.BaseOaiAdapter`. Usually this will be done in the Daiquiri app, e.g.

```
OAI_ADAPTER = 'app.adapter.OaiAdapter'
```
---

#### OAI_PAGE_SIZE

Default: `500`

Number of records returned in a OAI-PMH request.

---

daiquiri.query.settings
-----------------------

#### QUERY_ANONYMOUS

Default: `False`

Designates if the query interface can be accessed by anonymous users. The permissions on schemas and tables need to be configured using the metadata interface.

---

#### QUERY_USER_SCHEMA_PREFIX

Default: `'daiquiri_user_'`

Sets the prefix for user schemas in the `data` database. Each user has a private schema where the result table of successful queries are stored.

---

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

Sets the maximum quota for tables in a user's personal schema. The quota needs to be set for the `anonymous` user as well as regular logged-in users (`user`). Additionally, users or groups can be assigned individual quotas, e.g.:

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

---

#### QUERY_SYNC_TIMEOUT

Default: `5`

Sets the timeout for synchronous (TAP) queries in seconds.

---

#### QUERY_MAX_ACTIVE_JOBS

Default:

```python
{
    'anonymous': '1'
}
```

Sets the maximum number of simultaneous jobs for users. The setting works analogously to `QUERY_QUOTA`: if more than one maximum applies, the largest applicable value is used. If no maximum is given, no maximum is enforced.

---

#### QUERY_QUEUES

Default:

```python
[
    {
        'key': 'short',
        'label': '30 seconds',
        'timeout': 30,
        'access_level': 'PUBLIC',
        'groups': [],
    },
    {
        'key': 'long',
        'label': '1 Hour',
        'timeout': 3600,
        'access_level': 'PUBLIC',
        'groups': [],
    },
]
```

Sets the different query queues that can be selected by users. Each queue is represented by a dictionary where:

* `key` is the internal identifier,
* `label` is the text shown in the interface,
* `timeout` is the maximum execution time in seconds, and
* `access_level` and `groups` are the usual restrictions on who can use this queue.

Celery queue names are generated as `query_<key>`; for example, the defaults
become `query_short` and `query_long`.

---

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
* `version` is the version reported as part of the query-language identifier,
* `label` is the text shown in the interface,
* `description` is additional information shown, e.g. in TopCat, and
* `quote_char` is the character used by the database system to quote identifiers.

The default configuration provides `adql`. Additional entries can be configured
for supported database query processors; apart from ADQL, the query language
must match the database system of the `data` database.

---

#### QUERY_FORMS

Default:

```python
[
    {
        'key': 'sql',
        'label': 'SQL query',
        'template': 'query/query_form_sql.html'
    },
    {
        'key': 'conesearch',
        'label': 'Cone search',
        'submit': 'Submit new cone search',
        'template': 'query/query_form_cone.html',
        'adapter': 'daiquiri.query.adapter.ConeSearchQueryFormAdapter',
    },
    {
        'key': 'upload',
        'label': 'Upload VOTable',
        'template': 'query/query_form_upload.html',
    },
]
```

Sets the form available for the users in the query interface. Each form is represented by a dictionary where:

* `key` is the internal identifier,
* `label` is the text shown in the interface,
* `submit` is the submit-button label when a form provides one,
* `template` is the path to the Django template with the markup for the form, and
* `adapter` optionally names the class that prepares form-specific query data.

Included in Daiquiri are the `sql`, `conesearch`, and `upload` forms.

---

#### QUERY_PLOTS

Default:

```python
[
    {
        'key': 'scatter_plot',
        'label': 'Scatter',
        'is_active': True,
    },
    {
        'key': 'scatter_cmap_plot',
        'label': 'Scatter (color coded)',
        'is_active': True,
    },
    {
        'key': 'histogram',
        'label': 'Histogram',
        'is_active': True,
    }
]
```

Sets the visibility of the available plot types in the web interface.

* `key` is the internal identifier,
* `label` is the type name shown in the interface, and
* `is_active` controls whether the plot type is used in the interface.

---

#### QUERY_DROPDOWNS

Default:

```python
[
    {
        'key': 'schemas',
        'label': 'Database',
    },
    {
        'key': 'columns',
        'label': 'Columns',
    },
    {
        'key': 'functions',
        'label': 'Functions',
    },
    {
        'key': 'simbad',
        'label': 'Simbad',
        'options': {'url': 'http://simbad.u-strasbg.fr/simbad/sim-id'},
    },
    {
        'key': 'vizier',
        'label': 'VizieR',
        'options': {
            'url': 'http://vizier.u-strasbg.fr/viz-bin/votable',
            'catalogs': ['I/322A', 'I/259'],
        },
    },
    {
        'key': 'examples',
        'label': 'Examples',
        'classes': 'ms-auto',
    },
]
```

Sets the additional dropdown menus above the SQL query interface available for users. Each dropdown is represented by a dictionary where:

* `key` is the internal identifier,
* `label` is the text shown in the interface,
* `options` contains additional options specific to the dropdown, and
* `classes` can add CSS classes to the dropdown entry.

Included in Daiquiri are database, column, function, Simbad, VizieR, and
example dropdowns.

---

#### QUERY_DOWNLOAD_DIR

Default: `None`

Sets the absolute base path of the download files served by the query module in the local file system. It is read from `.env`; the query application requires a download directory to be configured.

---

#### QUERY_DEFAULT_DOWNLOAD_FORMAT

Default: `'votable'`

Sets the default download format.

---

#### QUERY_DOWNLOAD_FORMATS

Default:

```python
[
    {
        'key': 'votable',
        'extension': 'xml',
        'content_type': 'application/xml',
        'label': 'IVOA VOTable',
        'help': 'A XML file using the IVOA VOTable format. Use this option if you intend to use VO compatible software to further process the data.',
    },
    {
        'key': 'csv',
        'extension': 'csv',
        'content_type': 'text/csv',
        'label': 'Comma-separated Values',
        'help': 'A text file with a line for each row of the table. The fields are delimited by a comma and quoted by double quotes.',
    },
    {
        'key': 'fits',
        'extension': 'fits',
        'content_type': 'application/fits',
        'label': 'FITS',
        'help': 'Flexible Image Transport System (FITS) file format.'
    },
    {
        'key': 'parquet',
        'extension': 'parquet',
        'content_type': 'application/parquet',
        'label': 'Parquet',
        'help': 'Apache Parquet file format.',
    },
]
```

Sets the available default download formats. Each format is represented by a dictionary where:

* `key` is the internal identifier,
* `extension` the file extension,
* `content_type` the content type,
* `label` the text shown in the interface, and
* `help` a more verbose help text for the format.

**NOTE**: If you want faster `parquet` file generation, get the latest binary of [`pg2parquet`](https://github.com/exyi/pg2parquet) and put it somewhere covered by your `PATH` variable. This Rust binary is up to four times faster than the Python-based `fastparquet` fallback.

---

#### QUERY_UPLOAD

Default: `True`

Enables the upload functionality in the query interface.

---

#### QUERY_UPLOAD_LIMIT

Default:

```
{
    'anonymous': '10Mb',
    'user': '100Mb',
    'users': {},
    'groups': {}
}
```

Sets the maximum quota for uploads by a user. The quota needs to be set for the `anonymous` user as well as regular logged-in users (`user`). Users or groups can be assigned individual quotas, much like with `QUERY_QUOTA`.

---

daiquiri.serve.settings
-----------------------

#### SERVE_DOWNLOAD_DIR

Default: `None`

Sets the optional base path for download files served by the serve module. The
current serve settings module defines this value, but the current serve code
does not read it directly; configure application-specific serving code if a
separate download directory is needed.

---

#### SERVE_RESOLVER

Default: `None`

Sets a resolver class to handle references in tables served by the serve module. When set, the class is imported and instantiated by the serve utilities.

---

daiquiri.stats.settings
-----------------------

#### STATS_RESOURCE_TYPES

Default:

```python
[
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
        'label': 'Downloaded static files'
    },
    {
        'key': 'CREATE_ZIP',
        'label': 'Created zip files'
    },
    {
        'key': 'CREATE_FILE',
        'label': 'Created files for download'
    },
    {
        'key': 'DOWNLOAD',
        'label': 'Downloaded files'
    },
    {
        'key': 'UPLOAD',
        'label': 'Uploaded files'
    },
    {
        'key': 'QUERY',
        'label': 'Queries'
    }
]
```

Sets the aggregated resource types shown in the stats management overview.

---

daiquiri.tap.settings
---------------------

#### TAP_SCHEMA

Default: `'tap_schema'`

Sets the name of the TAP metadata schema or database namespace. If more than one Daiquiri application is using the same `data` database, they cannot use the same TAP schema. The configured name is used when queries are submitted. This setting can be set in `.env`.

---

#### TAP_UPLOAD

Default: `'tap_upload'`

Sets the name of the namespace for TAP uploads. For PostgreSQL this is a schema;
for MySQL it is a database. If more than one Daiquiri application is using the
same `data` database, they cannot use the same upload namespace. This setting
can be set in `.env`.

---

daiquiri.uws integration
------------------------

#### UWS_RESOURCES

There is no default value; `daiquiri.uws.urls` reads this optional setting
directly from Django's settings. If it is absent, no additional UWS resources
are registered.

Configures UWS services in addition to the TAP module. For example:

```python
{
    'prefix': r'query',
    'viewset': 'daiquiri.query.viewsets.UWSQueryJobViewSet',
    'base_name': 'uws_query'
}
```

Each entry is a dictionary where:

* `prefix` is the URL prefix of the service (e.g. `query`),
* `viewset` is the import path of the viewset class handling the requests, and
* `base_name` is the basename used to register routes with the Django REST framework.
