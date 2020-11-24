# strimzi

Once you have a Kafka cluster up and running

**Step 1: Download connector**

https://camel.apache.org/camel-kafka-connector/latest/connectors.html \

**Step 2: Build a docker image with the required connectors**

**Example Dockerfile**


*FROM strimzi/kafka-connect:0.11.4-kafka-2.1.0
USER root:root
COPY ./plugins/ /opt/kafka/plugins/
USER 1001* \
  
docker build . -t \<registry>/my_kafka_connectors:test1 \
docker push \<registry>/my_kafka_connectors:test1


**Step 3: Deploy Kafka Connect Cluster**

**Example YAML**

*apiVersion: kafka.strimzi.io/v1beta1 \
kind: KafkaConnect \
metadata: \
  name: my-connect-cluster \
  annotations: \
    strimzi.io/use-connector-resources: "true" \
spec: \
  version: 2.6.0 \
  replicas: 1 \
  bootstrapServers: my-cluster-kafka-bootstrap:9093 \
  image: fcr-nonprod.fmr.com/fmr-ap140435/my_kafka_connectors:test2 \
  tls: \
    trustedCertificates: \
      - secretName: my-cluster-cluster-ca-cert \
        certificate: ca.crt \
  config: \
    group.id: connect-cluster \
    offset.storage.topic: connect-cluster-offsets \
    config.storage.topic: connect-cluster-configs \
    status.storage.topic: connect-cluster-status \
    config.storage.replication.factor: 1 \
    offset.storage.replication.factor: 1 \
    status.storage.replication.factor: 1* \



**Step 4: Deploy Kafka Connector**

**Example YAML**

*apiVersion: kafka.strimzi.io/v1alpha1 \
kind: KafkaConnector \
metadata: \
  name: jira-source-connector \
  labels: \
    # The strimzi.io/cluster label identifies the KafkaConnect instance
    # in which to create this connector. That KafkaConnect instance
    # must have the strimzi.io/use-connector-resources annotation
    # set to true. \
    strimzi.io/cluster: my-connect-cluster \
spec: \
  class: org.apache.camel.kafkaconnector.jira.CamelJiraSourceConnector \
  tasksMax: 1 \
  topic: jira-topic \
  config: \
    camel.source.endpoint.jiraUrl: http://gvenkatx.atlassian.net/rest/api/2 \
    camel.component.jira.jiraUrl: http://gvenkatx.atlassian.net/rest/api/2 \
    camel.component.jira.username: gvenkatx@gmail.com \
    camel.component.jira.password: Myjira1718nc \
    camel.source.endpoint.delay: 60 \
    camel.component.jira.delay: 60 \
    camel.source.endpoint.jql: jql=project=GTES \
    camel.source.kafka.topic: jira-topic \
    camel.source.path.type: NEWISSUES \
    camel.proxyHost: http://http.proxy.fmr.com \
    camel.component.http.proxyPort: 8000* \

