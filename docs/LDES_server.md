---
sort: 4
---

# LDES SERVER

The Linked Data Event Stream (LDES) [server](https://github.com/Informatievlaanderen/VSDS-LDESServer4J) is a configurable component that can be used to ingest, store, and (re-)publish an LDES. The LDES server was built in the context of the VSDS project to exchange Open Data easily.


<p align="center"><img src="/VSDS-Tech-Docs/images/LDES%20server.png" width="60%" text-align="center"></p>

The server can tailor its functionality to meet the organisation's specific needs. Functionalities include **retention policy**, **fragmentation** and **pagination**  for managing and processing large amounts of data more efficiently and ensuring the efficient use of storage. 

![](../images/scalableApplications.png)

## Ingesting sources (HTTP in)

The LDES server is able to receive data via HTTP ingestion. Specifically, the server expects an LDES-compliant member to be sent as input via a POST request. This member must be a version object of an entity that adheres to the LDES standard.

However, if the data stream is not yet LDES compliant, such as a state object data stream, it must first be converted to a version object. This essential step ensures data compatibility and adherence to the LDES standards.

Once the data stream is converted to a version object and the LDES-compliant member is received by the LDES server, the server can effortlessly publish the LDES member as part of the LDES.

When starting up the LDES server, the server checks if a SHACL shape (Shapes Constraint Language) is provided before proceeding with the ingestion process. A SHACL shape is used to validate members against predefined validation rules and to verify the conformity of RDF data with these rules.

The SHACL shape specifies the properties of an RDF resource and the rules that must be followed to ensure the data adheres to the expected structure and semantics. It defines properties such as required properties, allowed property values, and the data types expected for the properties.

For more information about the SHACL shape and its structure, go to [here](https://informatievlaanderen.github.io/VSDS-Tech-Docs/docs/Specification.html#shacl).

Example HTTP Ingest-Fetch Configuration:


```yaml
server.port: { http-port }
ldes:
  collection-name: { short name of the collection, cannot contain characters that are used for url interpretation, e.g.’ $’, ‘=’ or ‘&’}
  host-name: { hostname of LDES Server }
  member-type: { Defines which syntax type is used to define the member id e.g. “https://data.vlaanderen.be/ns/mobiliteit#Mobiliteitshinder”}
  timestamp-path: { SHACL property path to the timestamp when the version object entered the event stream. }
  version-of: { SHACL property path to the non-versioned identifier of the entity. }
  validation:
    shape: { URI to defined shape }
    enabled: { Enables/Disables shacl validation on ingested members }
rest:
  max-age: { time in seconds that a mutable fragment can be considered up-to-date, default when omitted: 60 }
  max-age-immutable: { time in seconds that an immutable fragment should not be refreshed, default when omitted: 604800 }
```

## Fragmentation

To reduce the amount of data consumers need to replicate or to speed up certain queries, the LDES server can be configured with several fragmentations. Fragmentations are similar to indexes in databases but then published on the Web. The RDF predicate on which the fragmentation must be applied is defined through configuration.

<p align="center"><img src="/VSDS-Tech-Docs/images/fragmentation.png" width="60%" text-align="center"></p>

The fragmenting of a Linked Data Event Stream (LDES) is a crucial technique for managing and processing large amounts of data more efficiently. There are three main methods of fragmentation: **geospatial**, **time-based**, and **substring** fragmentation.

<br>

<br>

---

### Partitioning

When applying partitioning, the LDES server will create fragments based on the order of arrival of the LDES member. This fragmentation is considered to be the most basic and default fragmentation.
The members that arrive first on the LDES server are added to the first page, while the latest members are always included on the latest page.



```yaml
name: “pagination”
config:
  memberLimit: { Mandatory: member limit > 0 }
```


**Algorithm**

1. The fragment to which the member should be added is determined.
   - The currently open fragment is retrieved from the database.
   - If this fragment contains members equal to or exceeding the member limit or no fragment can be found, a new fragment is created instead.
2. If a new fragment is created, the following steps are taken. 
   - The new fragment becomes the new open fragment and the previous fragment becomes immutable<sup>1</sup>.
   - This newly created fragment and the previous fragment are then linked with each other by 2 generic relationships<sup>1</sup>.
   - The pagenumber of the new fragment is determined based on the old fragment or is set to 1 in case of the first fragment.

<sup>1</sup> In case of the first fragment, a previous fragment does not exist so these steps are skipped.

![](content/paginationAlgorithm.png)

**Example properties**


  ```yaml
  name: "pagination"
  config:
    memberLimit: 10
  ```

<br>

<br>

---

### Substring fragmentation

[Substring fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-substring) involves dividing the data stream into smaller pieces based on specific substrings, or patterns, within the data. 

Example of substring fragmentation configuration file

```yaml
name: “substring”
config:
  fragmenterProperty: { Defines which property will be used for bucketizing }
  memberLimit: { member limit > 0 }
```

**Example**

Example properties:

  ```yaml
  name: "substring"
  config:
    fragmenterProperty: "https://data.vlaanderen.be/ns/adres#volledigAdres"
    memberLimit: 10
  ```

With following example input:

```ttl
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

<https://data.vlaanderen.be/id/adres/1781020/2023-02-15T10:14:36.002Z>
  <https://data.vlaanderen.be/ns/adres#isVerrijktMet> [ 
      <https://data.vlaanderen.be/ns/adres#volledigAdres> "Kazernestraat 15, 9160 Lokeren"@nl 
   ] ;
  prov:generatedAtTime "2023-02-15T10:14:36.002Z"^^xsd:dateTime ;
  a <https://data.vlaanderen.be/ns/adres#Adres> .
```

The selected object would be "Kazernestraat 15, 9160 Lokeren".

The bucket of substrings would be:
- K
- Ka
- Kaz
- ...
- Kazernestraat 15, 9160 Lokeren

If this is the first member of the collection it would be added to fragment 'k' and available at http://localhost:8080/addresses/by-name?substring=k and not in ka or any other substring fragment.

In a scenario where there are already 10 addresses starting with 'k' and only 2 with 'ka', it would be added to http://localhost:8080/addresses/by-name?substring=ka

```note
Note that this is all lowercase.
```


<br>

<br>

---

### Time-based fragmentation
[Time-based fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-timebased) has not yet been implemented.

<p align="center"><img src="/VSDS-Tech-Docs/images/temporal.png" width="60%" text-align="center"></p>

Example of a time-based fragmentation configuration file



```yaml
name: “timebased”
config:
  memberLimit: { member limit > 0 }
```

**Algorithm**
This fragmentiser will create an initial fragment with the current timestamp when processing a member.
Members are added to the fragment until the member limit is reached. When the fragment member limit is reached, a
next fragment is created with a new current timestamp.


**Reasons for deprecating this fragmentiser:**
1. This fragmentiser follows the algorithm of pagination but without the semantics.
2. For a correct timebased fragmentation, members of the fragment should be checked and their value 
for a given property should be used to create the correct relations. This is not the case, and there is currently no demand to have this implemented.

<br>

<br>

---

### Geospatial fragmentation

[Geospatial fragmentation](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/tree/main/ldes-fragmentisers/ldes-fragmentisers-geospatial) involves dividing the data stream into smaller pieces based on geographical information, allowing organisations to process and analyse data within specific geographic areas in real-time. 

<p align="center"><img src="/VSDS-Tech-Docs/images/geospatial.png" width="60%" text-align="center"></p>

The geospatial fragmentation follows the "Slippy Maps" principle. A zoom level is set through configuration, and the "world" is divided into tiles based on this zoom level. The number of tiles is 2^2n^ (where n = zoom level). An RDF predicate must also be configured for this fragmentation, determining on which property of the LDES member the fragmentation should be applied ....

The required configuration for this fragmentation is:

1. RDF predicate on which the fragmentation should be based

2. Zoom level

Example of geospatial fragmentation configuration file
```yaml
name: “geospatial”
config:
  maxZoomLevel: { Required zoom level }
  fragmenterProperty: { Defines which property will be used for bucketizing }
```

**Algorithm**

1. The fragmentationObjects of the member are determined
   - We filter the RDF statements where the predicate matches the `fragmenterProperty`
   - If an optional regex is provided through the `fragmenterSubjectFilter` property, we filter on subjects that match this regex.
   - We select all the object that pass the above filters.
2. A bucket of tiles is created using the coordinates and provided zoomLevel. [This is done using the Slippy Map algorithm.](https://wiki.openstreetmap.org/wiki/Slippy_map)  
3. The tiles are iterated. The member is added to every tile, or sub-fragmentations of these tiles<sup>1</sup>. Taking into account:
   - A new fragment is created if no fragment exists for the given tile.
   - There is no `memberLimit` or max size for a fragment. They do not become immutable.
   - The member is added to every related fragment

<sup>1</sup> If the geospatial fragmentation is not the lowest fragmentation level, the member is not added to the tile but to a subfragment on this tile. This case is included in the [example below](#when-we-have-a-timebased-sub-fragmentation-below-geospatial-fragmentation).

![](content/geospatial_algorithm.drawio.png)


**Example**

Example properties:

  ```yaml
  name: "geospatial"
  config:
    maxZoomLevel: 15
    fragmenterProperty: "http://www.opengis.net/ont/geosparql#asWKT"
  ```

With following example input:

```ttl
@prefix dc: <http://purl.org/dc/terms/> .
@prefix ns0: <http://semweb.mmlab.be/ns/linkedconnections#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix ns1: <http://vocab.gtfs.org/terms#> .
@prefix prov: <http://www.w3.org/ns/prov#> .
@prefix ns2: <http://www.opengis.net/ont/geosparql#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix geo: <http://www.w3.org/2003/01/geo/wgs84_pos#> .

<http://njh.me/original-id#2022-09-28T17:11:28.520Z>
  dc:isVersionOf <http://njh.me/original-id> ;
  ns0:arrivalStop <http://example.org/stops/402161> ;
  ns0:arrivalTime "2022-09-28T07:14:00.000Z"^^xsd:dateTime ;
  ns0:departureStop <http://example.org/stops/402303> ;
  ns0:departureTime "2022-09-28T07:09:00.000Z"^^xsd:dateTime ;
  ns1:dropOffType ns1:Regular ;
  ns1:pickupType ns1:Regular ;
  ns1:route <http://example.org/routes/Hasselt_-_Genk> ;
  ns1:trip <http://example.org/trips/Hasselt_-_Genk/Genk_-_Hasselt/20220928T0909> ;
  a ns0:Connection ;
  prov:generatedAtTime "2022-09-28T17:11:28.520Z"^^xsd:dateTime .

<http://example.org/stops/402161>
  ns2:asWKT "POINT (5.47236 50.9642)"^^ns2:wktLiteral ;
  a ns1:Stop ;
  rdfs:label "Genk Brug" ;
  geo:lat 5.096420e+1 ;
  geo:long 5.472360e+0 .

<http://example.org/stops/402303>
  ns2:asWKT "POINT (5.49661 50.9667)"^^ns2:wktLiteral ;
  a ns1:Stop ;
  rdfs:label "Genk Station perron 11" ;
  geo:lat 5.096670e+1 ;
  geo:long 5.496610e+0 .
```

The selected objects would be

`"POINT (5.47236 50.9642)"^^ns2:wktLiteral` and `"POINT (5.49661 50.9667)"^^ns2:wktLiteral`

When we convert these [coordinates to tiles](https://wiki.openstreetmap.org/wiki/Slippy_map_tilenames#Lon..2Flat._to_tile_numbers_2), the bucket of tiles would be:
- "15/16884/10974"
- "15/16882/10975"

<ins>When geospatial fragmentation is the lowest level</ins>

After ingestion the member will be part of the following two fragments
- http://localhost:8080/addresses/by-zone?tile=15/16884/10974
- http://localhost:8080/addresses/by-zone?tile=15/16882/10975

<ins>When we have a timebased sub-fragmentation below geospatial fragmentation</ins>

After ingestion the member will be part of the following two fragments
- http://localhost:8080/addresses/by-zone-and-time?tile=15/16884/10974&generatedAtTime=2023-02-15T10:14:28.262Z
- http://localhost:8080/addresses/by-zone-and-time?tile=15/16882/10975&generatedAtTime=2023-02-15T10:14:28.262Z

Note that the `generatedAtTime=2023-02-15T10:14:28.262Z` is an example, this can be any other fragmentation.


<br>

<br>

---

## Retention policy


A [retention policy](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-retention) determines how long data will be kept and stored. Its purpose is to ensure the efficient use of storage resources by controlling data growth over time. Setting a retention policy per view to minimise storage fill-up is possible.

<p align="center"><img src="/VSDS-Tech-Docs/images/retention_policy.png" width="60%" text-align="center"></p>

Implementing a retention policy helps organisations maintain control over their data growth and ensure that storage resources are used optimally. The policy specifies the maximum duration that data should be kept. Currently, the only retention policy supported is time-based, which can be configured using the [ISO 8601](https://tc39.es/proposal-temporal/docs/duration.html) duration format. This time-based policy ensures that data is automatically deleted after a specified period, freeing up valuable storage space for new data.

```yaml
views:
  - name: “firstView”
    retentionPolicies:
      - name: “timebased”
        config:
          duration: “PT5M”
```
duration:  "PT5M"

As an example, the time-based retention configuration example above is set up to ensure that data is automatically deleted after 5 minutes (PT5M).

## Hosting the LDES stream SHACL shape

LDES server facilitates hosting an LDES stream SHACL shape. SHACL Shapes Constraint Language is a language used to validate RDF graphs against a set of conditions provided as shapes and other constructs in an RDF graph. SHACL shapes can be used to describe data graphs that satisfy these conditions, beyond validation, such as for user interface building, code generation, and data integration.

## Hosting DCAT metadata

DCAT is a standardised RDF vocabulary for data catalogues on the Web, allowing easy interoperability between catalogues. Using a standard schema, DCAT enhances discoverability and facilitates federated search across multiple catalogues.

LDES server facilitates hosting DCAT metadata when publishing an LDES. The DCAT metadata can be represented as linked data events and published on an LDES server. The LDES server can then serve as a real-time data source for DCAT metadata.


More information about this can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#example-serving-dcat-metadata)

**Example Serving DCAT Metadata**

Supported file formats: .ttl, .rdf, .nq and .jsonld Templates for configuring the DCAT metadata can be found [here](https://github.com/Informatievlaanderen/VSDS-LDESServer4J/blob/main/templates/dcat).


```yaml
ldes:
 dcat: { path of file or url containing DCAT metadata, e.g. “dcat/metadata.ttl”}
```

## (Re)publish an LDES stream

An LDES server facilitate the possibility to publish and republish an LDES stream.

<p align="center"><img src="/VSDS-Tech-Docs/images/LDES%20server.png" width="60%" text-align="center"></p>


## Authentication and authorisation


Authentication and authorisation enable secure data publication with restricted access.

In an OAuth2 gateway scenario, the gateway acts as an intermediary, handling authentication and authorisation, issuing access tokens, and granting access based on permissions.

<p align="center"><img src="/VSDS-Tech-Docs/images/authorisation.png" width="60%" text-align="center"></p>

The user is prompted to enter their login credentials, which are authenticated by the [OAuth2](https://oauth.net/2/) gateway. If authentication is successful, the user can access the LDES stream.

For governmental organisations [Access (ACM) and User Management (IDM)](https://www.vlaanderen.be/acm-idm-standaard-aansluitingsproces) can be used.
<br>

---

<br>



## Setup of the LDES Server

A detailed description on how to setup the LDES Server can be found in the [README.md](https://github.com/Informatievlaanderen/VSDS-LDESServer4J#ldes-server) of the LDES Server GitHub repository.