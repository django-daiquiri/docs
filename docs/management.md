Management
==========

Some of the functionalities of Daiquiri can be configured using a user friendly managment intreface. This interfacae is meant to be used not only by admins, who could also use the [Admin interface](http://localhost:8080/administration/#admin-interface), but also regular users, who are members of particular groups.

---

User management
---------------

URL path: */auth/users/*

Group: *user_manager*

Using the [AUTH_WORKFLOW setting](/settings/#daiquiriauthsettings), Daiquiri can be configured to use an `activation` or `confirmation` workflow. Using the `activation` workflow, after registration, users need to be manually activated. For the `confirmation` workflow, users need to be confirmed by through the interface, and then activated in a second step, usually ny a script through the API.

---

Contact messages
----------------

URL path: */contact/messages/*

Group: *contact_manager*

Users can send contact messages to the operators of a Daiquiri site. The coantact messages intreface can be used to respond to these messages or mark them as spam.

---

Metadata management
-------------------

URL path: */metadata/management/*

Group: *metadata_manager*

The metadata for database schemas, tables and columns, to be used with the query interface can be accessed through the metadata management. From here, all metadata entries can be edited. The metadata is automatically synced to the TAP_SCHEMA in the scientific database. Access to schemas and tables can be restricted to internal users (who are logged in) or made private and only available to certain groups.

---

Query examples
--------------

URL path: */metadata/management/*

Group: *metadata_manager*

The different examples, visible in the query interface can be created and edited using the example queries interface. As with the database schemas and tables, examples can be restricted to internal users (who are logged in) or made private and only available to certain groups.

---

Stats
-----

URL path: */stats/management/*

Group: *stats_manager*

A (very) basic overview over the statistics of the Daiquiri site is available in the stats management interface.
