# Daiquiri

Daiquiri is a framework for the publication of scientific databases.

Home page
: <https://django-daiquiri.github.io>

GitHub organization
: <https://github.com/django-daiquiri>

PyPI releases
: <https://pypi.org/project/django-daiquiri>

## Introduction

Today, the publication of research data plays an important role in astronomy and astrophysics. On the one hand, dedicated surveys like SDSS and RAVE, data intensive instruments like LOFAR, or massive simulations like Millennium and MultiDark are initially planned to release their data for the community. On the other hand, more traditionally oriented research projects strive to publish their data as a key requirement demanded by the funding agencies.

The common approach is to publish this data via dedicated web sites. This includes rather simple HTML forms as well as complex query systems such as SDSS-CAS. Most of these web sites are tailor made for the particular case and are therefore not easily transferable to future projects.

At [Leibniz-Institute for Astrophysics Potsdam (AIP)](http://www.aip.de/), we gained experience with both the maintenance and the development of such applications. It became, however, apparent that already the current plethora of applications constitutes a major challenge for maintenance expenses and scalability. In order to address these issues, we developed the **Daiquiri framework**, which is particularly designed to allow for different highly customizable web applications based on a common easily maintainable code base.

## Features

Daiquiri enables collaboration and institutions to create customized websites, comprising of the following features:

* An interactive Query interface enabling users to perform SQL/ADQL queries against catalog databases. The queries are analyzed using the [queryparser](https://github.com/django-daiquiri/queryparser) and permissions are checked depending on user accounts and groups.
* Asynchronous database queries, which can take minutes or even hours.
* Download of the query results in different formats and visualization of the data.
* A programmatic interface to the database implementing the [IVOA TAP](http://www.ivoa.net/documents/TAP/20180830/PR-TAP-1.1-20180830.html) protocol.
* A cone search API based on the [IVOA Simple Cone Search](http://www.ivoa.net/documents/latest/ConeSearch.html) recommendation.
* An integration into to [IVOA registry of registry](http://rofr.ivoa.net/) to make the VO endpoints available in applications like, e.g. [topcat](http://www.star.bris.ac.uk/~mbt/topcat/).
* A metadata management backend containing information about the database schemas and tables including DOI and UCD.
* The download of files connected to the database tables, including access restrictions.
* An OAI-PMH2 endpoint to make the metadata stored in the system available to harvesters.
* A cut-out API for multi-dimensional data (e.g. data cubes).
* A sophisticated user management system with customizable registration and confirmation workflows.
* A contact form connected to the management backend.
* A meeting module to organize workshops and smaller conferences.
* An integrated [WordPress](https://wordpress.org/) content management system for documentation and/or the presentation of the project.

## Requirements

Daiquiri is based on [Django](https://www.djangoproject.com/) and is written in Python. The following requirements are mandatory:

* Python `>=3.9`
* PostgreSQL `>=13` (MariaDB `>=10.1` or MySQL `>=5.6` not actively supported anymore)
* RabbitMQ `>=3.8` (for asyncronous tasks like the query queue)

For demonstration, development or testing purposes, Daiquiri can be installed on Linux, macOS, or even Windows. If you, however, intent to set up a production environment, serving Daiquiri over a Network or the Internet, we strongly suggest that you use a recent Linux distribution, namely:

* Debian 12
* Ubuntu 20.04


## Usage

Daiquiri is currently used on several sites hosted and maintained by the [Leibniz-Institute for Astrophysics Potsdam (AIP)](http://www.aip.de/):

* [Gaia@AIP Services](https://gaia.aip.de)
* [APPLAUSE archives](https://www.plate-archive.org)
* [MUSE-Wide survey](https://musewide.aip.de)
* [GREGOR project and archive](https://gregor.aip.de)
* [CLUES – Constrained Local UniversE Simulations project](https://www.clues-project.org)
* [CosmoSim database](https://www.cosmosim.org) (legacy version)
* [RAVE Survey](https://www.rave-survey.org) (legacy version)

## License

Daiquiri is Licensed unter the **Apache License 2.0**. The full license text is part of the source code repository at [github.com/django-daiquiri/daiquiri](https://github.com/django-daiquiri/daiquiri/blob/master/LICENSE).
