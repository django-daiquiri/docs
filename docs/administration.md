Administration
--------------

Admin interface
---------------

Daiquiri uses the [Django Admin interfaca](https://docs.djangoproject.com/en/2.2/ref/contrib/admin/) as an interface for admins to intereact with a particular Daiquiri site. Please refer to the Django documentation for a detailed guide. Here we summarize the most important options:

#### Site configuration

Daiquiri uses Django's [sites framework](https://docs.djangoproject.com/en/stable/ref/contrib/sites). It is therefore necesary to configure the **Domain name** and the **Display name** of your RDMO installation. This can be done in the admin interface under *SITE*.

#### Users and Groups

The users and groups of your Daiquiri site can be managed under **AUTHENTICATION AND AUTHORIZATION**. You can create and update users and set their password directly, but most of the time this will be done by the users themselves using the account menu.

The user created in the installation process can access all features of Daiquiri. In order to allow other users to access the management or the admin interface, they need to have the required permissions assigned to them. This can be done in two ways: through groups or using the superuser flag.

#### Social accounts

Using the [django-allauth](https://www.intenct.nl/projects/django-allauth/), social authentication using the OAuth2 protocol can be used with Daiquiri. For this `'allauth.socialaccount'` and, e.g. for GitHub, `'allauth.socialaccount.providers.github'` need to be added to `INSTALLED_APPS` in `base.py`. In addition, your app needs to be registered with the provider. The callback url needed for the registration is

```
<site url>/account/<provider>/login/callback/
```

Once the credentials are obtained, you need to enter them in the admin interface under SOCIAL ACCOUNTS. Click on Add social application and:

1. Select the corresponding provider
2. Enter a Name of your choice
3. Enter the Client id (or App ID) and the Secret key (or Client secret, Client key, App Secret)
4. Add your site to the chosen sites.
5. Click save.


Command line tools
------------------

In addition the web interface, Daiquiri provides several commands to inter, which can be

#### Setup groups

The permission system of Daiquiri utilizes a set of groups to manage access to the management interfaces and the WordPress integration. These groups can be created using:

```bash
python manage.py setup_groups
```

The groups creates are:

```
wordpress_editor
wordpress_admin
contact_manager
meetings_manager
metadata_manager
stats_manager
query_manager
user_manager
```

#### Setup TAP_SCHEMA

In order to setup the metadata for the TAP_SCHEMA run the following command (in your virtual environment):

```bash
python manage.py setup_tap_metadata
```

After this, */metadata/management/* should show a schema entry for the `tap_schema`.

#### Rebuild TAP_SCHEMA

The metadata is automatically synced between the metadata and the the TAP_SCHEMA. In order to rebuild the TAP_SCHEMA from scratch, you can use:

```bash
python manage.py rebuild_tap_schema
```

#### Rebuild DATALINK_TABLE

The global datalink table is partly used for metadata management. DOI and related identifier entries for public schemas and tables with a doi field are automatically generated.
Additionally all declared extra datalink tables (see `DATALINK_TABLES` setting variable) will be gathered and ingested into the global datalink table.
The global datalink table is in `TAP_SCHEMA`, is publically available and can be queried via TAP. The extra datalink table should stay `PRIVATE`.

To resync the global datalink table with the metadata, and new extra datalink tables, you can use:

```bash
python manage.py rebuild_datalink_table
```

#### Rebuild OAI Schema

Similar to the TAP_SCHEMA, the records for the OAI-PMH interface are located in a seperate schema, whithin the scientific database. In order to create the `oai_schema` from scratch, use:

```bash
python manage.py rebuild_oai_schema
```

Be aware that the OAI schema requires an up-to-date datalink table. So the datalink app should be activated, the global datalink table should exists: `setup_tap_schema`, and up-to-date: `rebuild_datalink_table`.

#### Dump Fixtures

```bash
# extract examples
python manage.py dumpdata daiquiri_query.example > examples.json

# extract files
python manage.py dumpdata daiquiri_files.directory > directories.json

# extract functions
python manage.py dumpdata daiquiri_metadata.function > functions.json

# extract groups
python manage.py dumpdata auth.group > groups.json

# extract users
python manage.py dumpdata auth.user daiquiri_auth.profile account.emailaddress > users.json
```

#### Load Fixtures

```bash
# extract examples
python manage.py loaddata examples.json

# extract files
python manage.py loaddata directories.json

# extract functions
python manage.py loaddata functions.json

# extract groups
python manage.py loaddata groups.json

# extract users
python manage.py loaddata users.json
```

#### Setup the scientific database

For helping the administrator in properly creating the scientific database, you can use:

```bash
python manage.py sqlcreate
```

Note: on the contrary to its name sqlcreate do not create anything, it just prints the SQL statement to properly create the science tables.

For the extra datalink tables, you can use:

```bash
python manage.py sqlcreate --datalink=<extra-datalink-tablename>
```

#### Notes on datalink

The extra datalink tables have four purposes:

1. improving the metadata with non-standard entries
2. creating relation between objects and groups
3. simplifying the discovery and access of files
4. declaring cross-table information for given object

##### Improving the metadata

Tables and schemas with a filled doi field will automatically get 3 datalink entries relating the schema/table name to its documentation, related identifier and doi url.
However not only tables and schema may have DOI, also objects or groups of objects may also diserve DOIs. In this case it is possible to use the extra datalink tables to 
create this relation: objects/doi.

The extra datalink tables should be named like: `<release_schema>.<object_category>_doi`.
The table should filled as follow:

1. the `ID` should be filled with a unique string identifying the object in the database: `<object_id>`
2. the `access_url` should be filled with the `<doi_url>`
3. the `semantics` field should be filled with `#doi`

This way after `rebuild_datalink_table` and `rebuild_oai_schema` the object will have entries in both the global datalink table and the oai schema.
The first ones can be queried either via `ivoa.TAP` or `ivoa.DATATLINK` protocol, the second via the `OAI-PMH` api.

The following table lists additional datalink entries that can be used for every objects to enrich the metadata for `OAI-PMH`

| __semantic__     | __OAI-PMH entry__                                                                                               |
|------------------|-----------------------------------------------------------------------------------------------------------------|
| #this            | `<relatedIdentifier relatedIdentifierType="URL" relationType="IsDescribedBy">access_url</relatedIdentifier>`    |
| #preview         | `<alternateIdentifier alternateIdentifierType="DOI Landing Page">access_url</alternateIdentifier>`              |
| #preview-image   | `<relatedIdentifier relatedIdentifierType="URL" relationType="IsSupplementedBy">access_url</relatedIdentifier>` |
| #detached-header | `<relatedIdentifier relatedIdentifierType="URL" relationType="IsSupplementedBy">access_url</relatedIdentifier>` |
| #documentation   | `<relatedIdentifier relatedIdentifierType="URL" relationType="IsDocumentedBy">access_url</relatedIdentifier>`   |
| #progenitor      | `<relatedIdentifier relatedIdentifierType="URL" relationType="IsDerivedFrom">access_url</relatedIdentifier>`    |
| #auxiliary       | `<relatedIdentifier relatedIdentifierType="URL" relationType="References">access_url</relatedIdentifier>`       |

The examples for the use of the semantics are 
| __semantic__     | __Used for...__                                                                                                 |
|------------------|-----------------------------------------------------------------------------------------------------------------|
| #this            | fits, hdf5, csv files, and resources holding the data                                                           |
| #preview         | viewers and doi-landing pages                                                                                   |
| #preview-image   | png, jpeg holding the represantation of the data                                                                |
| #detached-header | fits header, e.g., from previous releases                                                                       |
| #documentation   | related documentation pages                                                                                     |

More info on semantics: https://www.ivoa.net/rdf/datalink/core

##### Creating relation between object and groups

Several objects maybe part of a group of objects, i.e.: fields, observations, survey... In order to create a discoverable link it is possible to use the extra datalink tables.

The extra datalink table should names like: `<release_schema>.<group_category>_datalink`.
The table should be field as follow:

1. the `ID` field should be filled with `<group_name>/<group_id>`
2. the `access_url` should be filled with a url pointing to the linked ressource: datalink url of the object, viewer url of the object or doi url of the object.
3. the `semantics` field should be filled with `#auxilliary`

This way after `rebuild_datalink_table` and `rebuild_oai_schema` the groups will have entries in both the global datalink table and the oai schema.
The first ones can be queried either via `ivoa.TAP` or `ivoa.DATATLINK` protocol, the second via the `OAI-PMH` api.

Notes that a OAI Set with the group name is automatically created in the OAI schema, if the `ID` is build like `<group_name>/<group_id>`.

##### Simplifying file access

It is very frequent to not only provide table-data but also data in form of downloadable files. It is natural to link these downloadable files to a specific object.
In this case datalink is naturally useful:

The extra datalink table should be named like: `<release_schema>.<object_categoty>_files`.
The table should be filled as follow:

1. the `ID` field should be filled with `<object_id>`
2. the `access_url` should be filled with the downloadable url of the file
3. the `semantics` should be filled with the relevant value, i.e.: https://www.ivoa.net/rdf/datalink/core/2022-01-27/datalink.html
4. the `content_type` should be filled with the relevant value: i.e.: http://www.iana.org/assignments/media-types/media-types.xhtml
5. the `content_size` should be filled with the size of the file in bytes.

This way after `rebuild_datalink_table` and the object will have entries linking to the download links of related files in the global datalink table, queriable either via `ivoa.TAP` or `ivoa.DATALINK` protocol.

##### Declaring cross-tables information for given objects

Usually it may be relevant to split the information on object over sevral tables. Especially if the object somehow differ from nature and do not hold information in all tables.
Typically light sources in the sky may be either stellar objects, galaxies, ... These objects share informations in a main table, but specific information are spearded over various annex tables.

In this case it is usefull to provide in the main table flags to declare the existance of specific information in an annex table. Typically `stellar_flag`, `galaxy_flag`... these flags can be queried via TAP and convinienty helps the user to find his/her way through the database.

However, this still requires a knowledge about the naming and the existance of these flags. This is not sufficient for blind discovery. In order to allow user to blindly discover further specific information in annex tables, the extra datalink tables can be used:

The extra datalink table should named like: `<release_schema>.<object_category>_annex`
The table should be filled as follow:

1. the `ID` should be filled with the unique string identifying the object in the database: `<object _id>`
2. the `access_url` should be filled with the `<doi_url>` of the annex table, or its datalink url
3. the `semantics` field should be filled with `#auxilliary`

This way after `rebuild_datalink_table` the object will have blindly discoverable auxilliary information, that can be queried via TAP. The name of the table where the information can be found is present in the `access_url` and the `value` to query is the `ID` value.

Notes: for a large number of objects, this method must not be employed because the datalink table may become much to large and dynamic datalink feature (not yet implemented) should be used instead.

# Metadata trigerring features


## ucd

### meta.ref

Declares a column to be a reference. In the results tap of the query, this column will be rendered as a link toward the custom resolver (`resolver.py`).
The value of the column will be passed to the custom resolver to allow value specific rendering.

### meta.main

Declares a column as MAIN. This metadata identify the given column as a major contribution to the information of the table, this will be used by various
daiquiri services to present information: conesearch and default viewer.

#### Conesearch

The VO Conesearch service accepts to select either the entire tables dataset, a reduced sub-set, or just identifer and position information. The reduced sub-set is defined via the metadata `meta.main`.

#### Default viewer

In the default viewer, in case a table is provided as additional information in the datalink entries of a given object, a query on the columns marked as `meta.main` will be submitted.

### meta.id

This declares a column to be the datalink ID of the table. In combinaison with `meta.ref` and `meta.main` the column link will directly to the datalink viewer.

### meta.ref;meta.file

This declares a column to be the path to a FILE, it will be rendered as an URL and redirects to a download.

### meta.ref;meta.image

This declare a column to be the path to a PREVIEW, it will be rendered as an URL and redirects to a front-view of the image with a download button.

## datatype and arraysize

supported types:

* char
* boolean
* short
* int
* long
* float
* double
* spoint (ADQL)

supported array types:

* short
* int
* long
* float
* double
* boolean

### array of known dimension

For array of known dimension the metadata should be:
```
datatype: <type>[]
arraysize: <size>
```

### array of unknown dimension

For array of unknown dimension the metadata should be:
```
datatype: <type>[]
arraysize: None (null)
```

### Notes on Postgres array storage

On postgres side array should be stored as `ARRAY` of type `_<type>`.
