# Working Kafka Connect YAML file to load an IBM MQ SSL keystore

Last updated: 6 April 2025

```yaml
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
    # MQ SSL
    # Refer: https://github.com/dalelane/mq-kafka-connect-tutorial/blob/master/08-setup-kafka-connect/resources/connect-cluster.yaml
    # Allow the connector to read the config from secrets
    config.providers: file
    config.providers.file.class: org.apache.kafka.common.config.provider.DirectoryConfigProvider
    # MQ SSL
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
        # Change the license as required
        productName: IBM Event Streams for Non Production
        productVersion: 11.3.1
  tls:
    # To use TLS on the connection to the Kafka Connect cluster the CA cert of the cluster needs to be saved ni your namespace as a secret. The below name is the secret name.
    trustedCertificates:
    - certificate: ca.crt
      secretName: my-kafka-cluster-ca-cert
  # MQ JKS file
  # Refer: https://github.com/dalelane/mq-kafka-connect-tutorial/blob/master/08-setup-kafka-connect/resources/connect-cluster.yaml
  externalConfiguration:
    volumes:
      - name: mq-client-keystore
    secret:
      secretName: mq-client-keystore-yyyymmdd
  # MQ JKS file
```
