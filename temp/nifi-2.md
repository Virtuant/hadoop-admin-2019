# Data flow enrichment with NiFi part 2 : LookupAttribute processor  

## Introduction
This is part 3 of a series of articles on Data Enrichment with NiFi:

* Part 1: Data flow enrichment with LookupRecord and SimpleKV Lookup Service is available here
* Part 2: Data flow enrichment with LookupAttribute and SimpleKV Lookup Service is available here
* Part 3: Data flow enrichment with LookupRecord and MongoDB Lookup Service is available here
Enrichment is a common use case when working on data ingestion or flow management. Enrichment is getting data from external source (database, file, API, etc) to add more details, context or information to data being ingested. In Part 1 I showed how to use LookupRecord to enrich the content of a flow file. This is a powerful feature of NiFi based on the record based paradigm. For some scenarios we want to enrich the flow file by adding the result of the lookup as an attribute and not to the content of the flow file. For this, we can use LookupAttribute with a LookupService.

## Scenario
We will be using the same retail scenario of the previous article. However, we will be adding the city of the store as an attribute to each flow file. This information will be used inside NiFi for data routing for instance. Let's see how we can use LookupAttribute to do it.

## Implementation
We will be using the same GenerateFlowFile processor to generate data as well as the same SimpleKeyValueLookupService. In order to add the city of a store as an attribute, we will use a LookupAttribute with the follwing configuration:

# Image goes here:

The LookupAttribute processor will use the value of the attribute id_store as a key and query the lookup service. The returned value will be added as the 'city' attribute. To make this work, the flow files should have an attribute 'id_store' before entering the lookup processor. Currently, this information is only in the content. We can use an EvaluateJsonPath to get this information from the content to attribute.

# Image goes here:

The final flow looks as the following:

# Image goes here:

## Results
To verify that our enrichment is working, let's see the attribute of after the EvaluateJsonPath and then the LookupAttribute:

# Image goes here:

# Another image goes here:


