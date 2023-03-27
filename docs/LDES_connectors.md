---
sort: 6
---

# LDES CONNECTORS

## Version Object Creator

The Version Object Creator building block is responsible for converting a state object into a version object. 

## Version Materializer
The Version Materializer building block is responsible for converting a version object back into a state object. 

## NGSI-V2 to NGSI-LD
The NGSI-V2 to NGSI-LD building block is responsible for converting NGSI-V2 data to NGSI-LD format.

## SPARQL Construct Transfomer
SPARQL Construct is a query language used in semantic web technologies to create RDF (Resource Description Framework) graphs from existing RDF data. It allows users to specify a pattern of data they wish to extract from the RDF data and construct a new graph based on that pattern.

The SPARQL Construct query language provides a powerful way to create new RDF data by using existing data as the input. It can be used to transform RDF data into different formats, as well as to simplify the structure of RDF data by aggregating or filtering data. 

This SPARQL Construct Transfomer building block can be used to execute model transformations.

## "RDF4J PUT"-processor
The RDF4J PUT Processor building block is responsible for writing RDF data to triple stores that support the RDF4J API. This enables the import of LDES data into triple stores such as GraphDB.