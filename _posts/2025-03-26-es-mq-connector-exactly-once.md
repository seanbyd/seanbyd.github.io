# Kafka Connector Exactly-Once Delivery

Last updated: 25 March 2025

This post comtains the details to configure the Kafka Connector exactly-once delivery.

Full details can be found on the IBM page [Exactly-once message delivery semantics in IBM MQ source connector](https://github.com/ibm-messaging/kafka-connect-mq-source?tab=readme-ov-file#exactly-once-message-delivery-semantics).

More details can be found on [Exactly-once messaging](https://github.com/ibm-messaging/kafka-connect-mq-source?tab=readme-ov-file#exactly-once-message-delivery-semantics) in Github.

## Prerequisites

- Kafka Connect version 3.3.0 or later.

- Can only be used in distributed mode - that is with docker or kubernetes.

- [Kafka ACL requirements](https://kafka.apache.org/documentation/#connect_exactlyoncesource).

Refer to the IBM [running the mq source connector prerequisites](https://ibm.github.io/event-automation/es/connecting/mq/source/#prerequisites-1) page for full details.

## Settings

### Tasks max in the Kafka Connector

For exactly-once messaging, tasksMax must be set to 1 in the Kafka Connector yaml file.

``` yaml
tasksMax: 1
```

``` yaml
apiversion: eventstreams.ibm.com/vibeta2
kind: KafkaConnector
metadata:
  name: your-connector-name
  labels:
  :
spec:
  class: com.ibm.eventstreams.connect.mqsource.MQSourceConnector
  # For 'exactly-once' messaging, to avoid duplicates, the tasksMax must be set to 1.
  tasksMax: 1
  :
  config:
  :
```

### Replicas in the Kafka Connect runtime

While I haven't found this in the documentation, considering tasksMax in the Kafka Connector must be set to 1, by extension, the replicas value in the Kafka Connect runtime should be set to 1.

``` yaml
 replicas: 1
```

``` yaml
apiversion: eventstreams.ibm.com/v1beta2
kind: KafkaConnect
metadata:
  :
spec:
  authentication:
    :
  bootstrapServers: your-bootstrap-url
  config:
    :
  image: your-image
  replicas: 1
  resources :
    :
```

### Enable exactly-once in the Kafka Connect runtime

To enable exactly-once messaging, add the following to the Kafka Connect yaml file.

``` yaml
exactly.once.source.support: enabled
```
 
``` yaml
apiversion: eventstreams.ibm.com/v1beta2
kind: KafkaConnect
metadata:
  :
spec:
  authentication:
    :
  bootstrapServers: your-bootstrap-url
  config:
    # If you are using exactly-once messaging, set this to enabled.
    exactly.once.source.support: enabled
```

More details can be found in [enabling exactly-once prerequisites](https://ibm.github.io/event-automation/es/connecting/mq/source/#prerequisites-1
).

### Configure exactly-once in the Kafka Connector

Add the state queue name int the Kafka Connector yaml file.

``` yaml
mq.exactly.once.state.queue: specify-your-state-queue-name
```

``` yaml
apiversion: eventstreams.ibm.com/vibeta2
kind: KafkaConnector
metadata:
  :
spec:
  :
  config:
    # For exactly-once messaging, set the state queue name.
    # Define the queued as PERSISTENT with DEFSOPT(EXCL).
    mq.exactly.once.state.queue: specify-your-state-queue-name
    mq.queue.manager: your-queue-manager-name
```

Refer to [all connector configuration settings](https://github.com/ibm-messaging/kafka-connect-mq-source?tab=readme-ov-file#configuration) for a full list of the MQ source connector properties,.

### Transaction boundry

The transaction.boundary property must be set to poll. This is the default.

```
transaction.boundary: poll (default)
```

Refer to the [exactly-once delivery page](https://ibm.github.io/event-automation/es/connecting/mq/source/#enabling-exactly-once-delivery) for details.

Also refer to the [Kafka desription of transaction boundry](https://kafka.apache.org/documentation/#sourceconnectorconfigs_transaction.boundary).

## Full yaml files

Full yaml files have been provided below to help configure exactly-once messaging.

### Kafka Connect runtime

``` yaml
apiversion: eventstreams.ibm.com/v1beta2
kind: KafkaConnect
metadata:
  annotations:
    eventstreams.ibm.com/use-connector-resources: "true"
  labels:
    # Change the cluster name to a unique name in the Kafka environment
    # https://ibm.github.io/event-automation/es/connecting/setting-up-connectors/#configure-the-connector
    eventstreams.ibm.com/cluster: my-kafka-connect-01
  # Set the Kafka Connect runtime name.
  # This name appears in the Kafka Connect tab of the Event Streams Operator.
  # Ensure to use this as the prefix in the "productChargedContainers" field later in this file.
  name: my-kafka-connect-name-01
  # The namespace where you will run your connector.
  namespace: my-ocp-namespace-name
spec:
  authentication:
    # As the connector runs in a namespace in OpenShift, use the keys for authorization rather than SCRAM.
    # The user key details will be provided by your Kafka admin when they create your userid that will access Kafka.
    # Ensure the admin sends you the secrets in a yaml file and that the file contains the user key details (5 name-value key pairs) and not the SCRAM details (1 name-value key pair).
    # Create an OCP secret with these details.
    certificateAndKey:
      certificate: user.crt
      key: user.key
      # Set the below name to the same name you set when creating the secret containing the user key fields.
      secretName: my-user-keys-secret
    type: tls
  # Set the Kafka bootstrap URL.
  # Ensure the URL is the "internal" bootstrap URL. The URL can bne found in the "cluster connection" section of the Event Streams console.
  # The "internal" URL is used for namesopace-to-namespace communication.
  # The "external" URL is used when you are acceesing the Kafka cluster from outside of OpenShift.
  bootstrapServers: my=kafka-cluster-internal-bootstrap-url:9093
  config:
    # Uncomment the below if you are using exactly-once messaging.
    #exactly.once.source.support: enabled
    # If your Kafka cluster is set to use 3 brokers, set the factor values to 3.
    # The topic names must be unique within the Kafka environment. If the names are already
    # used, you'll get an error.
    # Set prefix of the 3 topics to the "eventstreams.ibm.com/cluster" value.
    # Ensure the Kafka admin granted your user access to created the below topics. If this is not
    # done, you'll see an authorization error in the Kafka Connect pod logs.
    config.storage.replication.factor: 3
    config.storage.topic: my-kafka-connect-01-config
    group.id: my-kafka-connect-01
    offset.storage.replication.factor: 3
    offset.storage.topic: my-kafka-connect-01-offset-storage
    status.storage.replication.factor: 3
    status.storage. topic: my-kafka-connect-01-status-storage
  # Set the image name to the one you want to use.
  # IN the below, my image name is "kc-mq-source-2.3.0" with tag "1.0".
  image: my-image-repo/kc-mq-source-2.3.0:1.0
  # The advice I received for the Kafka Connect runtime is to set replicas to 1.
  # This is particularly imortant when using 'exactly-once' messaging where taskaMax must be set to 1 to avoid duplicated messages. If replicas is set to 2, it would havwe the same effect as setting tasksMax to 2.
  # Ensure the limits and request sizes are <= the Quota values defined in OpenShift for your namespace. If these values are larger the pod won't start.
  replicas: 1
  resources :
    limits:
      cpu: 1
      memory: 2Gi
    requests:
      cpu: 1
      memory: 2Gi
  template:
    pod :
      metadata:
        annotations :
        cloudpakId: c8b82d189e7545f0892db9ef2731b90d
        cloudpakName: IBM Cloud Pak for Integration
        eventstreams.production.type: CloudPakForIntegrationNonProduction
        # Change the below to be the "name" from earlier in this file with the suffic "-connect".
        productChargedContainers: my-kafka-connect-name-01-connect
        productCloudpakRatio: "2:1"
        productID: 2a79e49111f44ec3acd89608e56138f5
        productMetric: VIRTUAL_PROCESSOR_CORE
        productName: IBM Event Streams for Non Production
        productVersion: 11.3.1
  tls:
    # To use TLS on the connection to the Kafka Connect cluster the CA cert of the cluster needs to be saved ni your namespace as a secret. The below name is the secret name.
    trustedCertificates:
    - certificate: ca.crt
      secretName: my-kafka-cluster-ca-cert```

### Kafka Connector

``` yaml
apiversion: eventstreams.ibm.com/vibeta2
kind: KafkaConnector
metadata:
  # Set the Kafka Connector name.
  # This name appears in the Kafka Connector tab of the Event Streams Operator.
  name: my-kafka-connector-name-01
  labels:
    # Set the the same value as "eventstreams.ibm.com/cluster" from the Kafka Connect yaml file.
    eventstreams.ibm.com/cluster: my-kafka-connect-01
spec:
  class: com.ibm.eventstreams.connect.mqsource.MQSourceConnector
  # For 'exactly once' messaging, to avoid duplicates, the value must be set to 1.
  #tasksMax: 1
  # Optionally add the autoRestarts setting to handle recovery situations.
  #autoRestart:
  #  enabled: true
  #  maxRestarts: 10
  config:
    # For exactly-once messaging, set the state queue name.
    # Define this queue as PERSISTENT with DEFSOPT(EXCL).
    # mq.exactly.once.state.queue: MY_STATE_QUEUE_NAME
    mq.queue.manager: MY_QMGR
    mq.connection.name.list: 10.1.1.1(1414)
    # For exactly-once messaging, set the HBINT(30) on the MQ channel name.
    mq.channel.name: MY_SVRCONN_CHANNEL
    mq.queue: MY_QUEUE_TO_GET_MESSAGES_FROM
    mq.user.name: ""
    mq.password: ""
    topic: MY-KAFKA-TOPIC-NAME
    mq.connection.mode: client
    mq.record.builder: com.ibm.eventstreams.connect.mqsource.builders.DefaultRecordBuilder
    # In this example we set the converter to ByteArrayConverter to ensre the message contents published to the topic is correct.
    # key.converter: org.apache.kafka.connect.storage.StringConverter
    # value.converter: org.apache.kafka.connect.storage.StringConverter
    key.converter: org.apache.kafka.connect.converters.ByteArrayConverter
    value.converter: org. apache.kafka. connect.converters.ByteArrayConverter
    # You can control the state of the connecter with the below vallues.
    #state: running
    #state: paused
    #state: stopped
```

## MQ config

Set the HBINT on the MQ chnnel to 30 seconds.

It is recommended to set the DEFSOPT option on the state queue to EXCL. 

Ensure the source queue (containing the messages that will be sent to Kafka) is set as persistent.

Ensure the state queue is set as persistent.

Refer to [creating a state queue in IBM MQ](https://ibm.github.io/event-automation/es/connecting/mq/source/#creating-a-state-queue-in-ibm-mq-by-using-the-runmqsc-tool) for the IBM MQ details.

## MQ message

Ensure the expiry time on the messges are set to unlimited.

Ensure all messages put to the source queue and state queue are persistent. If an app puts to the source queue, ensure the application doesn't override the queue default by setting the messages as non-persistent.

## Consuming apps

Ensure all consumers of the topic set the [Kafka isolation level](https://kafka.apache.org/documentation/#consumerconfigs_isolation.level) to read_committed. This ensures they are only consuming transactionally committed messages.

## Startup

Ensure the exatly once 'state' queue is empty before starting the connector with exactly-once enabled.

## URL

- [Exactly-once message delivery semantics in IBM MQ source connector](https://github.com/ibm-messaging/kafka-connect-mq-source?tab=readme-ov-file#exactly-once-message-delivery-semantics)

- [Exactly-once messaging in Github](https://github.com/ibm-messaging/kafka-connect-mq-source?tab=readme-ov-file#exactly-once-message-delivery-semantics)

- [Exactly-once Kafka reference](https://kafka.apache.org/documentation/#connect_exactlyoncesource)

- [Kafka ACL requirements](https://kafka.apache.org/documentation/#connect_exactlyoncesource)

- [Kafka isolation level](https://kafka.apache.org/documentation/#consumerconfigs_isolation.level)

- [Prerequisites](https://ibm.github.io/event-automation/es/connecting/mq/source/#prerequisites-1)

- [Enabling exactly-once delivery](https://ibm.github.io/event-automation/es/connecting/mq/source/#enabling-exactly-once-delivery)

- [All connector configuration settings](https://github.com/ibm-messaging/kafka-connect-mq-source?tab=readme-ov-file#configuration)