# strimzi

Once you have a Kafka cluster up and running

**Step 1: Build a docker image with the required connectors**

**Example Dockerfile**


*FROM strimzi/kafka:0.17.0-kafka-2.4.0\
USER root:root\
RUN mkdir -p /opt/kafka/plugins/camel\
COPY .strimzi-plugins/camel-telegram-kafka-connector-0.1.0-package.tar.gz /opt/kafka/plugins/camel/ \
RUN tar -xvzf /opt/kafka/plugins/camel/camel-telegram-kafka-connector-0.1.0-package.tar.gz --directory /opt/kafka/plugins/camel \
RUN rm /opt/kafka/plugins/camel/camel-telegram-kafka-connector-0.1.0-package.tar.gz \
USER 1001* \
  
docker build . -t \<registry>/my_kafka_connectors:test1 \
docker push \<registry>/my_kafka_connectors:test1


**Step 2: Deploy Kafka Connect Cluster**

**Example YAML**

*apiVersion: kafka.strimzi.io/v1beta1 \
kind: KafkaConnect \
metadata: \
  name: my-connect-cluster \
  annotations: \
    strimzi.io/use-connector-resources: "true" \
spec: \
  image: \<registry>/my_kafka_connectors:test1 \
  replicas: 1 \
  bootstrapServers: my-cluster-kafka-bootstrap:9093 \
  tls: \
    trustedCertificates: \
      - secretName: my-cluster-cluster-ca-cert \
        certificate: ca.crt \
  config: \
    config.storage.replication.factor: 1 \
    offset.storage.replication.factor: 1 \
    status.storage.replication.factor: 1 \
    config.providers: file \
    config.providers.file.class:  org.apache.kafka.common.config.provider.FileConfigProvider \
  externalConfiguration: \
    volumes: \
      - name: connector-config \
        secret: \
          secretName: telegram-credentials* \



**Step 3: Deploy Kafka Connector**

**Example YAML**

*apiVersion: kafka.strimzi.io/v1alpha1 \
kind: KafkaConnector \
metadata: \
  name: telegram-connector \
  labels: \
    strimzi.io/cluster: my-connect-cluster \
spec: \
  class: org.apache.camel.kafkaconnector.CamelSourceConnector \
  tasksMax: 1 \
  config: \
    key.converter: org.apache.kafka.connect.storage.StringConverter \
    value.converter: org.apache.kafka.connect.storage.StringConverter \
    camel.source.kafka.topic: telegram-topic \
    camel.source.url: telegram:bots \
    camel.component.telegram.authorizationToken: ${file:/opt/kafka/external-configuration/connector-config/telegram.properties:token} \
    transforms: telegram \
    transforms.telegram.type: org.apache.camel.kafkaconnector.transforms.CamelTypeConverterTransform$Value \
    transforms.telegram.target.type: java.lang.String* \

