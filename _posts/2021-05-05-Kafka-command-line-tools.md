---
title: "Kafka command-line tools"
toc: true
toc_label: "Contents"
tags:
  - Kafka

date: May 05, 2021
layout: single
classes: wide
excerpt: "Kafka command-line tools that you can use to troubleshoot your Kafka clusters."
---

## Introduction
Here is a summary of the most common Kafka commands that will help you debug your Kafka clusters. Apart from these commands, you can also use any UI tool like these: [KafkaTool](https://www.kafkatool.com/) or [Conduktor](https://www.conduktor.io/).

## Kafka topics
```bash
# List existing Kafka topics, excluding internal Kafka topics
$ kafka-topics.sh --bootstrap-server localhost:9092 --list --exclude-internal

# List existing Kafka topics with details
$ kafka-topics.sh --bootstrap-server localhost:9092 --describe

# Create a new Kafka topic
$ kafka-topics.sh --bootstrap-server localhost:9092 --topic kafka_topicX --create --replication-factor 3 --partitions 6

# Create a new Kafka topic if it does not exist
$ kafka-topics.sh --bootstrap-server localhost:9092 --topic kafka_topicX --create --replication-factor 3 --partitions 6 --if-not-exists

# Get information about a given topic
$ kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic kafka_topicX
```

### Alter some configs of a given topic:
```bash
$ kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic kafka_topicX --config retention.ms=7200000

$ kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic kafka_topicX --partitions 5
```

### Print topics with overridden configs:
```bash
$ kafka-topics.sh --bootstrap-server localhost:9092 --describe --topics-with-overrides
```

### Delete a given Kafka topic:
```bash
$ kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic kafka_topicX
```

## Produce Kafka records
### Opens a Keyboard Interactive Shell following '>'. Enter one record per line. Exit with Ctrl-C.
```bash
$ kafka-console-producer.sh --broker-list localhost:9092 --topic kafka_topicX
```

### Produce Kafka records from a file
```bash
$ kafka-console-producer.sh --broker-list localhost:9092 --topic kafka_topicX < topic-input.txt
```

### Produce Kafka records from stdin
```bash
$ cat "col1,col2,col3" | kafka-console-producer.sh --broker-list localhost:9092 --topic kafka_topicX
```

## Consume Kafka records
```bash
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic kafka_topicX
$ kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic kafka_topicX
```

## Getting information about consumer groups (CGs)
### Get all consumer groups:
```bash
$ kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

### List topic and partitions for a given CG:
```bash
$ kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group kafka_cgX --describe
```

### Reset CG offsets for a given topic:
```bash
$ kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group kafka_cgX --topic kafka_topicX --reset-offsets --to-earliest
```