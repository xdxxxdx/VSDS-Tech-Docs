---
sort: 6
---

# LDES CONNECTORS

## Version Object Creator

The Version Object Creator building block is responsible for converting a state object into a version object. 
To support the creation of version objects, e.g. when transforming data in the [NGSI LD format](https://vloca-kennishub.vlaanderen.be/NGSI_(LD)) to LDES.

## Version Materializer
The Version Materializer building block is responsible for converting a version object back into a state object. 

## NGSI-V2 to NGSI-LD
The NGSI-V2 to NGSI-LD building block is responsible for converting NGSI-V2 data to NGSI-LD format.
To support the creation of version objects, e.g. when transforming data in the [NGSI LD format](https://vloca-kennishub.vlaanderen.be/NGSI_(LD)) to LDES.


```
 "destination": {
                            "id": "daca1054-11e5-386b-bcad-9378ede61cd8",
                            "type": "PROCESSOR",
                            "groupId": "c07e45bb-d97c-33c0-af68-6de36e584fa5",
                            "name": "Translate NGSI-v2 to NGSI-LD",
                            "comments": "",
                            "instanceIdentifier": "6a3d76b3-86c9-325a-cba6-1030f968b6c5"
                        },
```

Here you can find a [workflow of the IoW case](https://github.com/Informatievlaanderen/VSDS-LDES-E2E-testing/blob/6228fe5add3f8f47354b53b723ded69a945ec4d3/use-cases/iow/workflow.json) where this processor is used.

## SPARQL Construct Transfomer
SPARQL Construct is a query language used in semantic web technologies to create RDF (Resource Description Framework) graphs from existing RDF data. It allows users to specify a pattern of data they wish to extract from the RDF data and construct a new graph based on that pattern.

The SPARQL Construct query language provides a powerful way to create new RDF data by using existing data as the input. It can be used to transform RDF data into different formats, as well as to simplify the structure of RDF data by aggregating or filtering data. 

This SPARQL Construct Transfomer building block can be used to execute model transformations.

[Here](https://github.com/Informatievlaanderen/VSDS-Linked-Data-Interactions/blob/6bf77a8078ff0eefd31478b93eec193b8a74fd7e/ldi-nifi/ldi-nifi-processors/sparql-interactions-processor/src/main/java/be/vlaanderen/informatievlaanderen/ldes/ldi/processors/SparqlConstructProcessor.java) you can find the Java file where the Sparql construct is implemented.

## "RDF4J PUT"-processor
The RDF4J PUT Processor building block is responsible for writing RDF data to triple stores that support the RDF4J API. This enables the import of LDES data into triple stores such as GraphDB.