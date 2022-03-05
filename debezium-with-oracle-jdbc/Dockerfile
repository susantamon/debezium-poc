ARG DEBEZIUM_VERSION
FROM debezium/connect:$DEBEZIUM_VERSION
ENV KAFKA_CONNECT_JDBC_DIR=$KAFKA_CONNECT_PLUGINS_DIR/kafka-connect-jdbc

ARG JMX_AGENT_VERSION
RUN mkdir /kafka/etc && cd /kafka/etc &&\
        curl -so jmx_prometheus_javaagent.jar \
        https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/$JMX_AGENT_VERSION/jmx_prometheus_javaagent-$JMX_AGENT_VERSION.jar

COPY config.yml /kafka/etc/config.yml

USER kafka

# Add Oracle client and drivers
COPY oracle_instantclient/xstreams.jar /kafka/libs
COPY oracle_instantclient/ojdbc8.jar /kafka/libs

# Add Confluent kafka connect for jdbc & elastic sink connector
COPY confluentinc-kafka-connect-jdbc-10.3.3/lib/*.jar /kafka/connect/kafka-connect-jdbc/
COPY confluentinc-connect-transforms-1.4.2/lib/*.jar /kafka/connect/kafka-connect-transforms/

# Add filter dependencies
COPY debezium-scripting/*.jar /kafka/connect/debezium-connector-oracle/
COPY groovy/*.jar /kafka/connect/debezium-connector-oracle/
