###############################################################################
#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance
#  with the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
# limitations under the License.
###############################################################################

###############################################################################
# Build Click Count Job
###############################################################################

FROM maven:3.6-jdk-8-slim AS builder

###############################################################################
# Build SQL Playground Image
###############################################################################

FROM flink:1.13.2-scala_2.11
#FROM alexskyer/flink1.12:v1

# Copy sql-client script
COPY bin/* /opt/sql-client/
RUN mkdir -p /opt/sql-client/lib

# Download connector libraries
RUN wget -P /opt/sql-client/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-elasticsearch7_2.12/1.13.0/flink-sql-connector-elasticsearch7_2.12-1.13.0.jar; \
    wget -P /opt/sql-client/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-sql-connector-kafka_2.11/1.13.0/flink-sql-connector-kafka_2.11-1.13.0.jar; \
    wget -P /opt/sql-client/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-connector-jdbc_2.11/1.13.0/flink-connector-jdbc_2.11-1.13.0.jar; \
    wget -P /opt/sql-client/lib/ https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-mysql-cdc/2.0.2/flink-sql-connector-mysql-cdc-2.0.2.jar; \
    wget -P /opt/sql-client/lib/ https://repo1.maven.org/maven2/com/ververica/flink-sql-connector-postgres-cdc/2.0.2/flink-sql-connector-postgres-cdc-2.0.2.jar; \
    wget -P /opt/sql-client/lib/ https://repo1.maven.org/maven2/com/ververica/flink-connector-debezium/2.0.2/flink-connector-debezium-2.0.2.jar; \
    wget -P /opt/sql-client/lib/ https://repo1.maven.org/maven2/com/ververica/flink-connector-mysql-cdc/2.0.2/flink-connector-mysql-cdc-2.0.2.jar; \
    wget -P /opt/sql-client/lib/ https://repo1.maven.org/maven2/com/ververica/flink-connector-postgres-cdc/2.0.2/flink-connector-postgres-cdc-2.0.2.jar; \
    wget -P /opt/sql-client/lib/ https://repo1.maven.org/maven2/com/ververica/flink-format-changelog-json/2.0.2/flink-format-changelog-json-2.0.2.jar; \
    wget -P /opt/sql-client/lib/ https://repo.maven.apache.org/maven2/org/apache/flink/flink-shaded-hadoop-2-uber/2.8.3-10.0/flink-shaded-hadoop-2-uber-2.8.3-10.0.jar; \
    wget -P /opt/sql-client/lib/  https://repo.maven.apache.org/maven2/org/apache/hadoop/hadoop-mapreduce-client-core/2.7.3/hadoop-mapreduce-client-core-2.7.3.jar; \
    wget -P /opt/sql-client/lib/ https://repo.maven.apache.org/maven2/mysql/mysql-connector-java/8.0.19/mysql-connector-java-8.0.19.jar;

# Copy configuration
COPY conf/* /opt/flink/conf/

#COPY jars/* /opt/sql-client/lib/

WORKDIR /opt/sql-client
ENV SQL_CLIENT_HOME /opt/sql-client

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
