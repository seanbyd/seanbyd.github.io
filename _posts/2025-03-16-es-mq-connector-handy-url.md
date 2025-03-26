# IBM Event Streams - MQ Connector - Handy URL

Last updated: 16 March 2025

IBM has produced high quality documentation around [IBM Event Streams](https://www.ibm.com/products/event-streams).

This page contains the URLs I found most useful while learning to create my first IBM Event Streams Kafka 'source' Connector.

# Source and Sink connector starting point

## Youtube

If you are new to Kafka Connectors like I am, I'd suggest spending a few minutes watching the youtube video [Apache Kafka 101: Kafka Connect (2023)](https://www.youtube.com/watch?v=J6adhl3wEj4). I'd actually watch all the Kafka videos from this presenter. They are awesome!

## URLs to guide you through the process

### High level guides

Perhaps the best starting point is the IBM Event Automation connector URLs on github.io.

The landing URLs contain a high level process to follow when implementing Kafka Connect runtimes and associated Kafka Connectors such as the IBM MQ source and sink connectors.

- [Install an IBM MQ **source** connector](https://ibm.github.io/event-automation/connectors/kc-source-ibm-mq/installation)

- [Install an IBM MQ **sink** connector](https://ibm.github.io/event-automation/connectors/kc-sink-ibm-mq/installation)

A **source** connector consumes IBM MQ messages from a queue and publishes them to a Kafka topic.

A **sink** connector consumes from Kafka topics and puts IBM MQ messages to a queue.

The rest of this post focusses on the IBM MQ to Kafka **source** connector only.

### Releases

Before reading too much, I suggest you have a quick scan of the following URL.

These URLs contain the IBM provided **source** and **sink** connectors, including the connector source code (if your are interested) together with compiled jar files.

- [Kafka Connect mq source connector](https://github.com/ibm-messaging/kafka-connect-mq-source/releases/)

- [Kafka Connect mq sink connector](https://github.com/ibm-messaging/kafka-connect-mq-sink/releases)

The jar files for the source connector (as of writing this post) are for the v2.3.0 release. Be aware that the following URLs will initiate a download of the jar file.

- jar file containing all the dependencies excluding the MQ classes: [kafka-connect-mq-source-2.3.0-dependencies-exc-mq.jar](https://github.com/ibm-messaging/kafka-connect-mq-source/releases/download/v2.3.0/kafka-connect-mq-source-2.3.0-dependencies-exc-mq.jar)

- jar file with the dependencies including the MQ classes: [kafka-connect-mq-source-2.3.0-jar-with-dependencies.jar](https://github.com/ibm-messaging/kafka-connect-mq-source/releases/download/v2.3.0/kafka-connect-mq-source-2.3.0-jar-with-dependencies.jar)

- jar file without the dependencies: [kafka-connect-mq-source-2.3.0.jar](https://github.com/ibm-messaging/kafka-connect-mq-source/releases/download/v2.3.0/kafka-connect-mq-source-2.3.0.jar)

*From what I have heard, this is the new location to obain the (open source) connectors.*

You may be able to obtain older source and sink connectors  from [Fix Central](https://www.ibm.com/support/fixcentral/swg/selectFixes?parent=ibm%7EOther%20software&product=ibm/Other+software/IBM+Event+Automation&release=1.0.0.0&platform=All&function=all). However, I do see a comment "*Kafka Connect MQ Source old version. For new version visit https://ibm.github.io/event-automation/connectors/kc-source-ibm-mq/installation*" which suggests the location may have permanently moved.

If you are interested in working from the IBM MQ perspective refer ro following URLs:

- [You want to download the jar file for the IBM MQ Kafka Connector](https://www.ibm.com/support/pages/you-want-download-jar-file-ibm-mq-kafka-connector). You'll need an IBM account to download this file.

- [Download the IBM MQ Kafka Connector](https://ibm.biz/mq94kafkaconnectors). Look for the *IBM MQ Kafka Connectors* link.

## Kafka Connect Runtime for an IBM MQ source Connector

To create a Kafka Connect runtime for an IBM MQ source connector, start with the high level process URL:

- [Install an IBM MQ **source** connector](https://ibm.github.io/event-automation/connectors/kc-source-ibm-mq/installation)

This page contains URLs that take you to the appropriate sections to complete your acitvities. I actually found this a great starting point.

You can happily follow that page rather than read the rest of this post, however, the below adds a few comments to highlight what I focussed on.

You can optionally read the entire page about [setting up and running connectors](https://ibm.github.io/event-automation/es/connecting/setting-up-connectors/#using-kafka-connect).

For me, I focussed on building a Kafka Connect image and deploying to OpenShift.

- [Configure connection to Kafka](https://ibm.github.io/event-automation/es/connecting/setting-up-connectors/#configure-connection-to-kafka)

- [Manually add required connectors](https://ibm.github.io/event-automation/es/connecting/setting-up-connectors/#manually-add-required-connectors)

- [Configure workers](https://ibm.github.io/event-automation/es/connecting/setting-up-connectors/#configure-workers)

## MQ source connector

To configure an MQ source connector, refer to the following URL:

- [Set up a Kafka connector](https://ibm.github.io/event-automation/es/connecting/setting-up-connectors/#set-up-a-kafka-connector)

The following lists all the properties you can configure.

- [Configuration options for the MQ source and sink connectors](https://ibm.github.io/event-automation/es/connecting/mq/#configuration-options)

To know the Kafka Connect "source" Connector image dependencies check out the below URL. I faced some challenges building the image until I discovered this page.

- [Source connect dependencies](https://ibm.github.io/event-automation/es/es_11.4/connecting/mq/source/#adding-connector-dependencies)