FROM flink:1.13.2-scala_2.11

# Download connector libraries
RUN wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-elasticsearch7_2.12/1.13.2/flink-sql-connector-elasticsearch7_2.12-1.13.2.jar; \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-kafka_2.11/1.13.2/flink-sql-connector-kafka_2.11-1.13.2.jar; \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-connector-jdbc_2.11/1.13.2/flink-connector-jdbc_2.11-1.13.2.jar; \
    wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.0.2/flink-sql-connector-mysql-cdc-2.0.2.jar; \
    wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-postgres-cdc/2.0.2/flink-sql-connector-postgres-cdc-2.0.2.jar; \
    wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/com/ververica/flink-connector-debezium/2.0.2/flink-connector-debezium-2.0.2.jar; \
    wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/com/ververica/flink-connector-mysql-cdc/2.0.2/flink-connector-mysql-cdc-2.0.2.jar; \
    wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/com/ververica/flink-connector-postgres-cdc/2.0.2/flink-connector-postgres-cdc-2.0.2.jar; \
    wget -P /opt/flink/lib/ https://repo1.maven.org/maven2/com/ververica/flink-format-changelog-json/2.0.2/flink-format-changelog-json-2.0.2.jar; \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-shaded-hadoop-2-uber/2.8.3-10.0/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar; \
    wget -P /opt/flink/lib/  https://repo.maven.apache.org/maven2/org/apache/hadoop/hadoop-mapreduce-client-core/2.7.3/hadoop-mapreduce-client-core-2.7.3.jar; \
    wget -P /opt/flink/lib/ https://repo.maven.apache.org/maven2/mysql/mysql-connector-java/8.0.19/mysql-connector-java-8.0.19.jar;

RUN apt-get update; \
apt-get install -y vim --fix-missing;

# Copy configuration
COPY conf/* /opt/flink/conf/

COPY jars/* /opt/flink/lib/

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
