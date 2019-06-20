## Connect File Pulse (BETA)

Connect File Pulse is a multi-purpose [Kafka Connect](http://kafka.apache.org/documentation.html#connect) (source) for streaming files from a local filesystem to Kafka.

## Motivation

Lot of enterprises rely on files to export/import data between their systems either in near real time or daily.
Files can be in different textual formats such as CSV, XML, JSON and so one.
A common use case is to decouple those systems by streaming those files into Kafka.

Connect File Pulse  attends to be a simple solution to deal with those kind of files.

Connect File Pulse is largely inspired from functionality provided by both Elasticsearch and Logstash.

## Features

* Recursively scan local directories
* Reading and writing files into Kafka line by line
* Parsing and transforming data using built-in or custom filters
* Error handler definition
* Monitoring files while they are being written into Kafka
* Plugeable strategies for cleaning completed files

	
## Getting Started

### Installation

#### Download, Build & Package Connector

```bash
git clone https://github.com/streamthoughts/kafka-connect-file-pulse.git
cd kafka-connect-file-pulse
```

You can build kafka-connect-file-pulse with Maven using standard lifecycle.

```bash
mvn clean install -DskipTests
```


#### Install Connector Using Confluent Hub

Connector will be package into an archive file compatible with [confluent-hub client](https://docs.confluent.io/current/connect/managing/confluent-hub/client.html) :

```
./connect-file-pulse-plugin/target/components/packages/streamthoughts-kafka-connect-file-pulse-plugin-<FILEPULSE_VERSION>.zip
```

#### Install Connector Manually


## Demonstrations

### Start Docker Environment

**1 ) Run Confluent Platforms with Connect File Pulse**

```bash
docker-compose build
docker-compose up -d
```

**2 ) Check for Kafka Connect**
```
docker logs --tail="all" -f connect"
```

**3 ) Verify that Connect File Pulse plugins correctly loaded**
```bash
curl -sX GET http://localhost:8083/connector-plugins | grep FilePulseSourceConnector
```


### Example : Logs Parsing (Log4j)

This example starts a new connector instance to parse the Kafka Connect container log4j logs before writing them into a configured topic.


**1 ) Start a new connector instance**

```bash
curl -sX POST http://localhost:8083/connectors \
-d @config/connect-file-pulse-quickstart-log4j.json \
--header "Content-Type: application/json" | jq
```

**2 ) Check connector status**
```bash
curl -X GET http://localhost:8083/connectors/connect-file-pulse-quickstart-log4j | jq
```

**3 ) Consume output topics**
```bash
docker exec -it -e KAFKA_OPTS="" connect kafka-avro-console-consumer --topic connect-file-pulse-quickstart-log4j --from-beginning --bootstrap-server broker:29092 --property schema.registry.url=http://schema-registry:8081
```

(output)
```json
...
{"loglevel":{"string":"INFO"},"logdate":{"string":"2019-06-16 20:41:15,247"},"message":{"string":"[main] Scanning for plugin classes. This might take a moment ... (org.apache.kafka.connect.cli.ConnectDistributed)"}}
{"loglevel":{"string":"INFO"},"logdate":{"string":"2019-06-16 20:41:15,270"},"message":{"string":"[main] Loading plugin from: /usr/share/java/schema-registry (org.apache.kafka.connect.runtime.isolation.DelegatingClassLoader)"}}
{"loglevel":{"string":"INFO"},"logdate":{"string":"2019-06-16 20:41:16,115"},"message":{"string":"[main] Registered loader: PluginClassLoader{pluginLocation=file:/usr/share/java/schema-registry/} (org.apache.kafka.connect.runtime.isolation.DelegatingClassLoader)"}}
{"loglevel":{"string":"INFO"},"logdate":{"string":"2019-06-16 20:41:16,115"},"message":{"string":"[main] Added plugin 'org.apache.kafka.common.config.provider.FileConfigProvider' (org.apache.kafka.connect.runtime.isolation.DelegatingClassLoader)"}}
...
```

**4) Observe Connect status**

Connect File Pulse use an internal topic to track the current state of files being processing.

```bash
docker exec -it -e KAFKA_OPTS="" connect kafka-console-consumer --topic connect-file-pulse-status --from-beginning --bootstrap-server broker:29092
```

(output)
```json
{"hostname":"f51d45f96ed5","status":"SCHEDULED","metadata":{"name":"kafka-connect.log","path":"/var/log/kafka","size":172559,"lastModified":1560772525000,"inode":1705406,"hash":661976312},"offset":{"position":-1,"rows":0,"timestamp":1560772525527}}
{"hostname":"f51d45f96ed5","status":"STARTED","metadata":{"name":"kafka-connect.log","path":"/var/log/kafka","size":172559,"lastModified":1560772525000,"inode":1705406,"hash":661976312},"offset":{"position":-1,"rows":0,"timestamp":1560772525719}}
{"hostname":"f51d45f96ed5","status":"READING","metadata":{"name":"kafka-connect.log","path":"/var/log/kafka","size":172559,"lastModified":1560772525000,"inode":1705406,"hash":661976312},"offset":{"position":174780,"rows":1911,"timestamp":1560772535322}}
...
```

**5 ) Stop all containers**
```bash
docker-compose down
```

### Example : CSV File Parsing

This example starts a new connector instance that parse a CSV file and filter rows based on column's values before writing record into Kafka.

**1 ) Start a new connector instance**

```bash
curl -sX POST http://localhost:8083/connectors \
-d @config/connect-file-pulse-quickstart-csv.json \
--header "Content-Type: application/json" | jq
```

**2 ) Copy example csv file into container**

```bash
docker exec -it connect mkdir -p /tmp/kafka-connect/examples
docker cp examples/quickstart-musics-dataset.csv connect://tmp/kafka-connect/examples/quickstart-musics-dataset.csv
```

**3 ) Check connector status**
```bash
curl -X GET http://localhost:8083/connectors/connect-file-pulse-quickstart-csv | jq
```

**4 ) Check for task completion**
```
docker logs --tail="all" -f connect | grep "Orphan task detected"
```

**5 ) Consume output topics**
```bash
docker exec -it connect kafka-avro-console-consumer --topic connect-file-pulse-quickstart-csv --from-beginning --bootstrap-server broker:29092 --property schema.registry.url=http://schema-registry:8081
```

## Documentation

1. [Common Configuration](common-configuration)
2. [Scanning Files](scanning-files)
3. [File Readers](file-readers)
4. [Filters Chain definition](filters-chain-definitions)
5. [Acceding Data and Metadata](acceding-data-and-metadata)
6. [Conditional execution](conditional-execution)
7. [Handling Failures in Pipelines](handling-failires-in-pipelines)
8. [Filters](filters)
9. [Tracking file progression](tracking-file-progression)
10. [Cleaning completed files](cleaning-completed-files)
11. [FAQ](faq)

## Contributions

Any feedback, bug reports and PRs are greatly appreciated!

## Licence

Licensed to the Apache Software Foundation (ASF) under one or more contributor license agreements. See the NOTICE file distributed with this work for additional information regarding copyright ownership. The ASF licenses this file to you under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License