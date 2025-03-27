# How to create your first Kafka MQ source connector

Last updated: 27 March 2025

This posts describes the steps to create your first Kafka MQ source connector.

# Prerequisites

- The Kafka connect runtime has been deployed and started.

- The queue manager has been created and configured for a Kafka connector.

- The Kafka topic exists.

# Create the Kafka MQ source connector yaml file

Create a yaml file named kc-mq.yaml as follows.

Refer to the comments in the yaml file for additional details on configuring this file.

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
    # You can control the state of the connecter with the below values.
  #state: running
  #state: paused
  #state: stopped
```
# Deploy the Kafka MQ source connector

The below process uses the OC CLI.

### Log in to OpenShift

Log into OpenShift using the OC CLI.

``` sh
oc login -u YOUR_USER_ID --server=https://OPENSHIFT_URL:OPENSHIFT_PORT
```

``` sh
oc login -u sean --server=https://api.crc.testing:6443
```

### Select the correct project

List the projects.

``` sh
oc projects
```

```
/home/sean>oc projects
You have access to the following projects and can switch between them with ' project <projectname>':

  * my-kafka-dev
    my-kafka-qc

Using project "my-kafka-dev" on server "https://api.crc.testing:6443".
```

If you are not on the correct project (the one with the '*'), select it.

``` sh
oc project my-kafka-qc
```

``` sh
/home/sean>oc project my-kafka-qc
Now using project "my-kafka-qc" on server "https://api.crc.testing:6443".
```

### Deploy the Kafka MQ source connector

Issue the follopwing command to deploy the connector.

``` sh
oc apply -f kc-mq.yaml
```

``` sh
/home/sean>oc apply -f kc-mq.yaml
kafkaconnector.eventstreams.ibm.com/my-kafka-connector-name-01 created
```

To check the deployment status, you can log into the OpenShift console and view the pod logs.

Alternatively, you can use the OC CLI.

Get the pod name.

``` sh
oc get pods
```

```
/home/sean>oc get pods
NAME                                              READY   STATUS    RESTARTS   AGE
my-kafka-connect-name-01-connect-0                1/1     Running   0          31s
```

You can tail the logs as follows.

Check for any errors.

``` sh
oc logs -f my-kafka-connect-name-01-connect-0
```

### Verification

Assuming no errors in the pod logs, use IBM MQ Explorer to confirm the input handles on the MQ queue is 1.

## References

To configure a Kafka MQ source connector, refer to the following URL:

- [Set up a Kafka connector](https://ibm.github.io/event-automation/es/connecting/setting-up-connectors/#set-up-a-kafka-connector)

The following lists all the properties you can configure.

- [Configuration options for the Kafka MQ source and sink connectors](https://ibm.github.io/event-automation/es/connecting/mq/#configuration-options)

- [All connector configuration settings](https://github.com/ibm-messaging/kafka-connect-mq-source?tab=readme-ov-file#configuration)

