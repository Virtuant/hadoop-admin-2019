
# Data flow enrichment with NiFi part 1 : LookupRecord processor

## Introduction

## This is part 1 of a series of articles on Data Enrichment with NiFi:

* Part 1: Data flow enrichment with LookupRecord and SimpleKV Lookup Service is available here
* Part 2: Data flow enrichment with LookupAttribute and SimpleKV Lookup Service is available here
* Part 3: Data flow enrichment with LookupRecord and MongoDB Lookup Service is available here

Enrichment is a common use case when working on data ingestion or flow management. Enrichment is getting data from external source (database, file, API, etc) to add more details, context or information to data being ingested. It's common that data contains references (ex. IDs) rather than the actual information. These references can be used to query another data source (known as reference table in the relational world) to get other attributes of an entity (location, name, etc). Often, the enrichment is done in batch using the join operation. However, doing the enrichment on data streams in realtime is more interesting.

## Motivation
Enrichment in previous versions of NiFi was not natively supported. There were few workarounds to do it but they are not performant neither integrated. This is because NiFi is a data flow tool. In Data Flow logic, each flow file is an independent item that can be processed independently. Data enrichment involves correlating and joining two data sources at least which is not the sweet-spot of NiFi.

Starting from NiFi 1.3, it's possible to do data enrichment with new processors (LookupAttribute and LookupRecord) and new lookup services. This article explains how these new features can be used.

## Scenario
Let's take an example of real-time retail data ingestion coming from stores in different cities. Data is coming in JSON form with the following schema:

# Image goes here:

This JSON tells us that we have 45 units of product 889 in store 4. Details of store 4 such as city, adresse, capacity, etc are available on another data source. Let's say that we want to do some geographical analysis and show in a realtime dashboard all stores that have products which will be out of stock soon. To do so, we need information on the store locations. This can be achieved through data enrichment.

# Implementation
### Data generation
Let's use a GenerateFlowFile with the below configuration to simulate data coming from 5 different store (1 to 5).

# image goes here:

### Data enrichment
LookupRecord processor uses ServiceLookup services for data enrichment. You can see the lookup service as a Key-Value service that LookupRecord queries to get the value associated with a key. Currently, there are 6 available ServiceLookup:

# Image goes here:

PropertiesFileLookupService, SimpleCsvFileLookupService and IPLookupService are file-based lookup services. Your reference data should be sitting in a file (CSV,XML, etc) that NiFi will use to match a value to a key. ScriptedLookupService uses a script (Python, Ruby, Groovy, etc) to generate a value corresponding to a key. The SimpleKeyValueLookupService stores the key-value pairs in NiFi directly. It is very convenient to use if your reference data is not too big and don't evolve too much. This is the case in our scenario. Indeed, we don't add a new store each day. Other interesting lookup services are coming with the new versions of NiFi. These include MongoDB (NIFI-4345) and HBase (NIFI-4346).

To start the enrichment, add a LookupRecord processor to the flow and configure the following properties:

* Record Reader: Create a new JSONTreeReader and configure it. Use schema text property as a "Schema Access Strategy" and use the following Avro Schema

# Image goes here:

This tells the LookupRecord processor to serialize received JSON data with the provided schema. We don't use any schema registry here. I won't go into details of record oriented processors or Schema registries. If you are not familiar with these concepts, start by reading this article here

# Image goes here

* Record Writer: create a new JsonRecordSetWriter and configure it. Set the different attributes as follow and use this schema for the Schema Text property:

# Image goes here

# Another image goes here:

Note that the writer schema is slightly different from the reader schema. Indeed, I added a field called 'city' that the processor will populate.

* Lookup Service: create a new SimpleKeyValueLookupService and populate it with your reference data. Here, I added the city of each one of my stores. Store 1 is Paris, store 2 is in Lyon, and so on.

# Image goes here:

Finalize the configuration of the lookup processor. You need to add a custom property "key" and set it to the JSON path of the field that will be used for the lookup. Here, it's the Store ID so Key = /id_store. Result RecordPath tells the processor where to store the retrieved value. Finally, route to 'matched' or 'unmatched' strategy tells the processor what to do after the lookup.

# Image goes here:

Connect the LookupRecord to the next processor and start the flow. For demonstration, I'll be merging the encriched JSON event and pushing them to Solr to build my dashboard.

# Image goes here:

## Results
To verify that our enrichment is working, let's see the content of flow files using the data provenance feature.

# Image goes here:

First of all, you can notice that LookupRecord is adding an attribute called avro.schema. This is due to the write strategy that we are using. It's not useful here but just wanted to highlight this. By using a Schema Registry, we can add the name of the schema only.

Let's see the content of a flow file now. As you can see, a new field "city" is added to my JSON. Here the city is Toulouse since my Store ID is 4. It's worth noting that it's possible to write the file in other format (Avro for instance) to have enrichment and conversion with one step.

# Image goes here:

# Conclusion
Data enrichment is a common use case for ingestion and flow processing. With Lookup processors and services, we can now easily enrich data in NiFi. The existing Lookup services are convenient if reference data doesn't change often. Indeed, reference data is manually added or use through a file. In future NiFi releases, new databases lookup services will be available (ex. MongoDB and Hbase).
