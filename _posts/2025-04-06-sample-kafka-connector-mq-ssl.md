# Working Kafka MQ source connector YAML file to connect to IBM MQ using SSL

Last updated: 6 April 2025

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
    # For exactly-once messaging, set the state queue name, plus the reconnect option.
    # Define this queue as PERSISTENT with DEFSOPT(EXCL).
    # mq.exactly.once.state.queue: MY_STATE_QUEUE_NAME
    # mq.client.reconnect.options: QMGR
    mq.queue.manager: MY_QMGR
    mq.connection.name.list: 10.1.1.1(1414)
    # For exactly-once messaging, set the HBINT(30) on the MQ channel name.
    mq.channel.name: MY_SVRCONN_CHANNEL
    mq.queue: MY_QUEUE_TO_GET_MESSAGES_FROM
    mq.user.name: ""
    mq.password: ""
    topic: MY-KAFKA-TOPIC-NAME
    # MQ SSL
    # Refer: https://github.com/dalelane/mq-kafka-connect-tutorial/blob/master/08-setup-kafka-connect/resources/connector.yaml
    mq.ssl.use.ibm.ciper.mappings: false
    mq.ssl.cipher.suite: TLS_AES_128_GCM_SHA256
    mq.ssl.truststore.location: /opt/kafka/external-configuration/mq-client-keystore/client.jks
    mq.ssl.truststore.password: ${file:/opt/kafka/external-configuration/mq-client-keystore:password}
    mq.ssl.keystore.location: /opt/kafka/external-configuration/mq-client-keystore/client.jks
    mq.ssl.keystore.password: ${file:/opt/kafka/external-configuration/mq-client-keystore:password}
    # MQ SSL
    mq.connection.mode: client
    mq.record.builder: com.ibm.eventstreams.connect.mqsource.builders.DefaultRecordBuilder
    # In this example we set the converter to ByteArrayConverter to ensre the message contents published to the topic is correct.
    # key.converter: org.apache.kafka.connect.storage.StringConverter
    # value.converter: org.apache.kafka.connect.storage.StringConverter
    key.converter: org.apache.kafka.connect.converters.ByteArrayConverter
    value.converter: org. apache.kafka. connect.converters.ByteArrayConverter
    # You can control the state of the connector with the below values.
    # The default state is running so you don't need to uncomment this field. I
    # choose to uncomment as I find this makes it easier for me when changing
    # the running states as the name/value pair appears when you do an
    # "oc edit kafkaconnector CONNECTOR_NAME".
    state: running
    #state: paused
    #state: stopped
```
