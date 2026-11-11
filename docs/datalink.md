Custom DataLink endpoints
=========================

This page explains how to add a custom DataLink endpoint to a Daiquiri
application. It focuses on the new service configuration and custom adapter
path.

It does not document the root DataLink service (`path: '/'`). The root service
currently combines several behaviors and is work in progress. Keep the root
service in your configuration when adding a custom service. For the current
root-service documentation, see the [DataLink settings](settings.md#daiquiridatalinksettings)
and the [administration notes](administration.md#notes-on-datalink).

DataLink connects an identifier from a discovery result with a file, related
resource, or service. Daiquiri returns these links using the IVOA DataLink
protocol. See the [official IVOA DataLink recommendation](https://www.ivoa.net/documents/DataLink/).

Configure a custom service
--------------------------

Add a new entry to `DATALINK_SERVICES`. Keep the root entry and add the custom
entry alongside it:

```python
DATALINK_SERVICES = [
    {
        'column_name': None,
        'path': '/',
        'adapter': 'daiquiri.datalink.adapter.DefaultDatalinkAdapter',
    },
    {
        'adapter': 'config.datalink.NewIdDatalinkAdapter',
        'path': '/new_id/',
        'column_name': 'new_id',
    },
]
```

The three fields in the custom entry have separate responsibilities:

* `adapter` is the import path of the adapter class used by the custom
  endpoint. The class must inherit from `BaseDatalinkAdapter` and is
  instantiated without arguments.
* `path` identifies the endpoint family handled by the adapter. A path such as
  `'/new_id/'` creates:

  * `/datalink/new_id/links`
  * `/datalink/new_id/view`
  * `/datalink/new_id/availability`
  * `/datalink/new_id/capabilities`

* `column_name` identifies the result column whose values are the identifiers
  served by this custom endpoint. A matching result column receives the
  DataLink service descriptor and identifier reference.

The endpoint family is independent of result-column linking. Setting
`column_name` to `None` leaves the result-column association unset, but the
configured endpoint and adapter still remain available.

Minimal custom adapter
----------------------

The following is an intentionally minimal adapter example. It contains only
the `get_datalink_rows` method and is shown as a Markdown example; no Python
file is added to the Daiquiri documentation repository.

```python
from daiquiri.datalink.adapter import BaseDatalinkAdapter


class NewIdDatalinkAdapter(BaseDatalinkAdapter):
    def get_datalink_rows(self, identifiers, **kwargs):
        return [
            [
                identifier,
                f'https://example.org/data/{identifier}.fits',
                None,
                None,
                'Example FITS file',
                '#this',
                'application/fits',
                None,
            ]
            for identifier in identifiers
        ]
```

`identifiers` contains the values requested through the DataLink `ID`
parameter. Each returned row currently contains these values in order:

| Value | Meaning |
| --- | --- |
| `ID` | Input identifier |
| `access_url` | URL of the linked data or service |
| `service_def` | Reference to a service descriptor, if applicable |
| `error_message` | Error text when the link cannot be created |
| `description` | Human-readable description of the link |
| `semantics` | DataLink semantic identifier |
| `content_type` | MIME type of the linked content |
| `content_length` | Size of the linked content in bytes, if known |

The `https://example.org/` URL is a placeholder. Replace it with the URL of
the resource or service provided by the application.

UCD metadata
------------

For a custom endpoint, `column_name` is the setting that associates a result
column with the service. The UCD `meta.id;meta.ref` is used for automatic
recognition of root-service identifier columns; root-service behavior is not
covered by this guide.

Calling the endpoint
--------------------

The custom links endpoint accepts one or more `ID` parameters:

```text
/datalink/new_id/links?ID=<identifier>
```

It returns a DataLink VOTable by default. JSON can be requested with
`RESPONSEFORMAT=application/json`:

```text
/datalink/new_id/links?ID=<identifier>&RESPONSEFORMAT=application/json
```

The `view` endpoint presents the links in a browser-oriented view. The
`availability` and `capabilities` endpoints provide the corresponding VO
service metadata.
