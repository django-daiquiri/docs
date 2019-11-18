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

#### Rebuild OAI Schema

Similar to the TAP_SCHEMA, the records for the OAI-PMH interface are located in a seperate schema, whithin the scientific database. In order to create the `oai_schema` from scratch, use:

```bash
python manage.py rebuild_oai_schema
```
