
### Overview

In the following, we show the API of the Daiquiri OAI for the [Datacite](https://schema.datacite.org/meta/kernel-4.4/) metadata schema.

The entry values for OAI is generated from the four different resources:

 - Metadata model `metadata.models`
 - SETTINGS
 - constant values, hard encoded in the oai renderer
 - Datalink (Tables and dynamic datalink),which is not implemented in the default daiquiri and is specific for every app.

Note: Since in most cases `metadata.models.Schema` and `metadata.models.Table` 
have the same attributes, the source given for a Schema is also valid for the Tables, 
unless otherwise noted.

| ID    | DataCite-Property         | Source                            | Notes
|-------|---------------------------|-----------------------------------|-------
| 1     | Identifier                | `metadata.models.Schema.doi`      | `DOI` or default `obj/pk`
| 1.a   | identifierType            | constant `DOI`                    |
| 2     | Creator                   | `metadata.models.Schema.creator`  | JSON field
| 2.1   | creatorName               | `..creator.name`                  | If not present `creator.last_name, creator.first_name` 
| 2.2.a | nameType                  | `..creator.name_type`             | must be `Organizational` or `Personal`
| 2.2   | givenName                 | `..creator.first_name`            |
| 2.3   | familyName                | `..creator.last_name`             |
| 2.4   | nameIdentifier            | `..creator.orcid`                 |
| 2.4.a | nameIdentifierScheme      | constant `ORCID`                  |
| 2.4.b | schemeURI                 | constant `http://orcid.org/`      |
| 2.5   | affiliation               | `..creator.affiliation`           | List of Dict
| 2.5.a | affiliationIdentifier     | `..affiliation.affiliation_identifier`           | e.g., `https://ror.org/03mrbr458`
| 2.5.b | affiliationIdentifierScheme | `..affiliation.affiliation_identifier_scheme`  | e.g., `ROR`
| 2.5.c | schemeURI                 | `..affiliation.scheme_uri`        | e.g., `https://ror.org/`
| 3     | Title                     | `metadata.models.Schema.title`    |
| 4     | Publisher                 | `core.settings.daiquiri.SITE_PUBLISHER` |
| 5     | PublicationYear           | `metadata.models.Schema.published.year` |
| 6     | Subject                   | `core.setttings.daiquiri.SITE_SUBJECTS.subject`    |
| 6.a   | subjectScheme             | `..SITE_SUBJECTS.subjectScheme`   |
| 6.b   | schemeURI                 | `..SITE_SUBJECTS.schemeURI`       |
| 6.c   | valueURI                  | `..SITE_SUBJECTS.valueURI`        |
| 7     | Contributor               | `metadata.models.Schema.contributors` | JSON Field (list of dict)
| 7.a   | contributorType           | `..contributor.contributor_type`  | If not present, `DataManager`
| 7.1   | creatorName               | `..contributor.name`              | If not present `..contributor.last_name, ..contributor.first_name`
| 7.2.a | nameType                  | `..contributor.name_type`         | must be `Organizational` or `Personal`
| 7.2   | givenName                 | `..contributor.first_name`        |
| 7.3   | familyName                | `..contributor.last_name`         |
| 7.4   | nameIdentifier            | `..contributor.orcid`             |
| 7.4.a | nameIdentifierScheme      | constant `ORCID`                  |
| 7.4.b | schemeURI                 | constant `http://orcid.org/`      |
| 7.5   | affiliation               | `..contributor.affiliation`       | List of Dict
| 7.5.a | affiliationIdentifier     | `..affiliation.affiliation_identifier` | e.g., `https://ror.org/03mrbr458`
| 7.5.b | affiliationIdentifierScheme |`..affiliation.affiliation_identifier_scheme`| e.g., `ROR`
| 7.5.c | schemeURI                 | `..affiliation.scheme_uri`        | e.g., `https://ror.org/`
| 8     | Date                      | `metadata.models.Schema.updated` and `..published` |
| 8.a   | dateType                  | constant `Updated` and `Issued`   | the used value corresponds to `Date`
| 9     | Language                  | `.core.settings.daiquiri.SITE_LANGUAGE` |
| 10    | ResourceType              | constant `Database table` or `Database schema` |
| 10.a  | resourceTypeGeneral       | constant `Dataset`                |
| 11    | alternateIdentifier       | `DataciteSchemaSerializer.get_alternate_identifiers` | abs. url to the metadata of the Table/Schema created on the fly
| 11.a  | alternateIdentifierType   | constant `URL`                    |
| 12    | RelatedIdentifer          | `metadata.models.Schema.related_identifier` | JSON Field
| 12.a  | relatedIdentifierType     | `..related_identifier.related_identifer_type` |
| 12.b  | relationType              | `..related_identifier.relation_type` |
| 13    | Size                      | `DataciteSchemaSerializer.get_alternate_identifiers` | for schemas: `n tables`, for tables:  `<size>n columns</size><size>m rows</size>`
| 14    | Format                    | `query.settings.QUERY_DOWNLOAD_FORMATS.content_type`   |
| 15    | Version                   | supposed to be at `metadata.models.Schema.version` | Not implemented in metadata. Always empty.
| 16    | Rights                    | `DataciteSchemaSerializer.license_label` | created from `metadata.settings.LICENCE_CHOICES` via `metadata.models.Schema.license`
| 16.a  | rightURI                  | `DataciteSchemaSerializer.license_url`   | created from `metadata.settings.LICENCE_URL` via `metadata.models.Schema.license`
| 16.b  | rightsIdentifier          | `DataciteSchemaSerializer.license_identifiers`   | created from `metadata.settings.LICENCE_IDENTIFIERS` via `metadata.models.Schema.license` 
| 17    | Description               | `metadata.models.Schema.long_description` | If not present or `null`, `metadata.models.Schema.description`
| 17.a  | descriptionType           | constant `Abstract`                       | Additionally, it uses the attribute `xml:lang` which renders `core.settings.daiquiri.SITE_LANGUAGE`


