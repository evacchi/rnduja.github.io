---
layout:     post
title:      "Confluent Platform"
subtitle:   "An initial evaluation"
date:       2016-02-15 09:00:00
author:     "Fabio Fumarola"
header-img: "img/2015-10-26-deep_learning_with_torch_step_7_optim/cover.jpg"
comments:   true
tags:       [reactive-programming]
---

# Confluent Platform Initial Evaluation

The goal of this blog post is to evaluate the [Confluent Platform](http://www.confluent.io/).

The Confluent Platform is built around Apache Kafka and adds to it:

- Clients: native c/c++ kafka client
- Schema registry
- Rest Proxy
- Connectors to: JDBC databases, HDFS and Hive

 In particular, we are interested on analyzing the [Schema-Registry](http://docs.confluent.io/2.0.0/schema-registry/docs/index.html) and the [REST Proxy](http://docs.confluent.io/2.0.0/kafka-rest/docs/index.html) added on top of [apache kafka](http://kafka.apache.org/).

## Setup
The examples below can be executed by starting [from this gist](https://gist.github.com/fabiofumarola/858f7140c29b875b5864).

<script src="https://gist.github.com/fabiofumarola/858f7140c29b875b5864.js"></script>


Requirements: docker, docker-compose

- starts
  ```bash
  docker-compose up -d
  ```

- export the env variable for curl:

	mac:
	```bash
	DOCKER_IP=`docker-machine ip dev`
	```

	linux:
	```bash
	DOCKER_IP=localhost
	```


## Schema Registry

The Confluent Schema Registry provides a serving layer for your metadata. It provides a RESTful interface for storing and retrieving Avro schemas. It stores a versioned history of all schemas, provides multiple compatibility settings and allows evolution of schemas according to the configured compatibility setting. It provides serializers that plug into Kafka clients that handle schema storage and retrieval for Kafka messages that are sent in the Avro format.

The main concepts related to the schema registry are: **subject, schema and version**.

1. A subject is a textual label that will be associated to a kafka topic
2. A schema is a avro compatible schema that describes the data model used to store data into a topic
3. a version is a numeric id used to identify a schema used in a subject.


### Api tutorial

The first step is to understand how to list subjects and get their associated schemas.

#### List Subjects

Request:

~~~~~~~~bash
curl -X GET -i http://$DOCKER_IP:8081/subjects
~~~~~~~~

Return a list of subjects

Response:

~~~~~~~~bash
HTTP/1.1 200 OK
Content-Length: 71
Content-Type: application/vnd.schemaregistry.v1+json
Server: Jetty(8.1.16.v20140903)

["complex"]
~~~~~~~~

#### Version associated to a subject

~~~~~~~~bash
curl -X GET -i http://$DOCKER_IP:8081/subjects/<subject>/versions
~~~~~~~~


#### Get schemas associated to a subject

~~~~~~~~bash
curl -X GET -i http://$DOCKER_IP:8081/subjects/<subject>/versions/<id>
~~~~~~~~

#### Post a Subject

Register a new schema under the specified subject. If successfully registered, this returns the unique identifier of this schema in the registry. The returned identifier should be used to retrieve this schema.
> If the same schema is registered under a different subject, the same identifier will be returned. However, the version of the schema **may be different** under different subjects.

Example 1: kafka-key subject

~~~~~~~~bash
curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
http://$DOCKER_IP:8081/subjects/kafka-key/versions
~~~~~~~~

Response 1:

~~~~~~~~bash
HTTP/1.1 200 OK
Content-Length: 8
Content-Type: application/vnd.schemaregistry.v1+json
Server: Jetty(8.1.16.v20140903)

{"id":1}%
~~~~~~~~

Example 2: kafka-value with the same schema of kafka-key

~~~~~~~~bash
curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
http://$DOCKER_IP:8081/subjects/kafka-value/versions
~~~~~~~~

Response 2

~~~~~~~~bash
HTTP/1.1 200 OK
Content-Length: 8
Content-Type: application/vnd.schemaregistry.v1+json
Server: Jetty(8.1.16.v20140903)

{"id":1}%   
~~~~~~~~
Here we can see that the id associated to the subject is the same.


#### Post and Update a Subject

An easy way to test the put several version of the same subject is to be enabled the `FORWARD` compatibility configuration:

~~~~~~~~bash
curl -X PUT -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"compatibility": "FORWARD"}' \
	http://$DOCKER_IP:8081/config
~~~~~~~~

Now, we can create the subject *complex* with the schema **complex**:

~~~~~~~~bash
curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{ \"type\": \"record\",\"name\": \"complex\",\"fields\":[{\"type\": \"string\",\"name\": \"field1\"}]}"}' \
http://$DOCKER_IP:8081/subjects/complex/versions
~~~~~~~~

Then we update the subject by updating the schema content of the subject:

~~~~~~~~bash
curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{ \"type\": \"record\",\"name\": \"complex\",\"fields\":[{\"type\": \"string\",\"name\": \"field1\"},{\"name\": \"value\", \"type\": \"long\"}]}"}' \
http://$DOCKER_IP:8081/subjects/complex/versions
~~~~~~~~

Here we can see that the id associated are different.

If we lists the stored subjects we get `["complex","kafka-key","kafka-value"]`

~~~~~~~~bash
curl -X GET -i http://$DOCKER_IP:8081/subjects
~~~~~~~~

and if we query for the version of the subject `complex` we get `[1,2]`

~~~~~~~~bash
curl -X GET -i http://$DOCKER_IP:8081/subjects/complex/versions
~~~~~~~~

Finally, we can get the name, version and schema of the returned versions by:

~~~~~~~~bash
curl -X GET -i http://$DOCKER_IP:8081/subjects/complex/versions/1

curl -X GET -i http://$DOCKER_IP:8081/subjects/complex/versions/2
~~~~~~~~


#### Check if a schema exists under a subject

Check if a schema has already been registered under the specified subject. If so, this returns:

- subject
- version
- id
- schema

~~~~~~~~bash
curl -X POST -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    --data '{"schema": "{\"type\": \"string\"}"}' \
http://$DOCKER_IP:8081/subjects/kafka-key
~~~~~~~~

Response:

~~~~~~~~bash
HTTP/1.1 200 OK
Content-Length: 64
Content-Type: application/vnd.schemaregistry.v1+json
Server: Jetty(8.1.16.v20140903)

{"subject":"kafka-key","version":1,"id":1,"schema":"\"string\""}%
~~~~~~~~

### Schema Registry Discussion

The goal of the schema registry is immediately such as its data model. I spent some time to fully understand how to test the concept of version associated with a **subject** especially because it strongly depends on the base compatibility configuration of the registry [("BACKWARD")](http://docs.confluent.io/2.0.0/schema-registry/docs/api.html#compatibility). I think that the rationale here should be to use Avro optional fields to support base schema evolution.

#### Data Model

- A subject has a schema associated,
- Each schema has an id,
- Each schema can be associated with multiple subjects,
- It is possible to associate multiple schema to the same subject. If compatibility is **Forward** we can set different schema (or breaking evolutions of the same schema) into the same subject. Each update will have a different version id.

## REST Proxy

The Kafka REST Proxy provides a RESTful interface to a Kafka cluster. It makes it easy to produce and to consume messages (in json, avro and binary), to view the state of the cluster, and to perform administrative actions without using the native Kafka protocol or clients.


**Examples**

~~~~~~~~bash

# Get a list of topics
$ curl "http://$DOCKER_IP:8082/topics"

# Get info about one topic
$ curl "http://$DOCKER_IP:8082/topics/test"

# Get info about a topic's partitions
$ curl "http://$DOCKER_IP:8082/topics/test/partitions
~~~~~~~~

**Produce and Consume Avro Messages**

~~~~~~~~bash
# Produces a message using Avro embedded data, including the schema which will
# be registered with the schema registry and used to validate and serialize
# before storing the data in Kafka
$ curl -X POST -H "Content-Type: application/vnd.kafka.avro.v1+json" \
      --data '{"value_schema": "{\"type\": \"record\", \"name\": \"User\", \"fields\": [{\"name\": \"name\", \"type\": \"string\"}]}", "records": [{"value": {"name": "testUser"}}]}' \
      "http://$DOCKER_IP:8082/topics/avrotest"


# create a consumer
curl -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
      --data '{"name": "my_consumer_instance", "format": "avro", "auto.offset.reset": "smallest"}' \
      http://$DOCKER_IP:8082/consumers/my_avro_consumer

 curl -X GET -H "Accept: application/vnd.kafka.avro.v1+json" \
      http://$DOCKER_IP:8082/consumers/my_avro_consumer/instances/my_consumer_instance/topics/avrotest     
~~~~~~~~

If we query the schema registry we will see a new subject named `avrotest-value`

~~~~~~~~
curl -X GET -i http://$DOCKER_IP:8081/subjects   

#with this command we can get its schema
curl -X GET -i http://$DOCKER_IP:8081/subjects/avrotest-value/versions/1
~~~~~~~~

Thus, when we create a new topic, using the confluent clients or the http proxy, it would be add in the schema-registry a new mapping for the topic key and its value.

### Features

The Rest PROXY exposes all the functionalities of the Java producers, consumers, and command-line tools:

- Metadata: metadata of the state of the cluster
- Producers and Consumers: expose creation and communication
- Data Formats: can write JSON, raw bytes base64 and JSON-encoded Avro.

The design of the API resembles the [Eventstore API](http://docs.geteventstore.com/http-api/3.4.0/)

## Java Clients

The [Confluent Kafka](https://github.com/confluentinc/kafka) extends the base Apache Kafka by adding:

1. HDFS and JDBC connection
2. connection to schema registry and topic validation
3. Camus for Kafka to HDFS pipelines.

As we are interested on the registry interaction with the Clients Api we evaluated the [example](https://github.com/confluentinc/examples) provided by confluent.

The producer and consumer api are similar to the Apache foundation version. The interaction with the schema registry is done via a configuration add to the producer/consumer.

~~~~~~~~java
Properties props = new Properties();
props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", schemaUrl);
~~~~~~~~

This makes everything transparent to the final user. What is not clear is if it is possible to restrict the creation/modification of topics from the client.
In kafka broker configuration we can setup `auto.create.topics.enable=false` to disable automatic topic creation, but it is not clear from Confluent documentation how this settings interact with the registry and the api.
