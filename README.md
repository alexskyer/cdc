##### 1.Enter mysql’s container and initialize data
```shell
docker-compose exec mysql mysql -uroot -p123456
```
```sql
CREATE DATABASE mydb;
USE mydb;
CREATE TABLE products (
  id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description VARCHAR(512)
);
ALTER TABLE products AUTO_INCREMENT = 101;

INSERT INTO products
VALUES (default,"scooter","Small 2-wheel scooter"),
       (default,"car battery","12V car battery"),
       (default,"12-pack drill bits","12-pack of drill bits with sizes ranging from #40 to #3"),
       (default,"hammer","12oz carpenter's hammer"),
       (default,"hammer","14oz carpenter's hammer"),
       (default,"hammer","16oz carpenter's hammer"),
       (default,"rocks","box of assorted rocks"),
       (default,"jacket","water resistent black wind breaker"),
       (default,"spare tire","24 inch spare tire");

CREATE TABLE orders (
  order_id INTEGER NOT NULL AUTO_INCREMENT PRIMARY KEY,
  order_date DATETIME NOT NULL,
  customer_name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 5) NOT NULL,
  product_id INTEGER NOT NULL,
  order_status BOOLEAN NOT NULL -- Whether order has been placed
) AUTO_INCREMENT = 10001;

INSERT INTO orders
VALUES (default, '2020-07-30 10:08:22', 'Jark', 50.50, 102, false),
       (default, '2020-07-30 10:11:09', 'Sally', 15.00, 105, false),
       (default, '2020-07-30 12:00:30', 'Edward', 25.25, 106, false);
```
##### 2.Enter Postgres’s container and initialize data
```shell script
docker-compose exec postgres psql -h localhost -U postgres
```
```sql
CREATE TABLE shipments (
  shipment_id SERIAL NOT NULL PRIMARY KEY,
  order_id SERIAL NOT NULL,
  origin VARCHAR(255) NOT NULL,
  destination VARCHAR(255) NOT NULL,
  is_arrived BOOLEAN NOT NULL
);
ALTER SEQUENCE public.shipments_shipment_id_seq RESTART WITH 1001;
ALTER TABLE public.shipments REPLICA IDENTITY FULL;

INSERT INTO shipments
VALUES (default,10001,'Beijing','Shanghai',false),
       (default,10002,'Hangzhou','Shanghai',false),
       (default,10003,'Shanghai','Hangzhou',false);
```
##### 3.Launch a Flink cluster, then start a Flink SQL CLI and execute following SQL statements inside
```shell script
docker exec -it cdc_sql-client_1 /bin/bash
```
```sql
SET execution.checkpointing.interval = 3s;

CREATE TABLE products (
    id INT,
    name STRING,
    description STRING,
    PRIMARY KEY (id) NOT ENFORCED
  ) WITH (
    'connector' = 'mysql-cdc',
    'hostname' = 'mysql',
    'port' = '3306',
    'username' = 'root',
    'password' = '123456',
    'database-name' = 'mydb',
    'table-name' = 'products'
  );

CREATE TABLE orders (
   order_id INT,
   order_date TIMESTAMP(0),
   customer_name STRING,
   price DECIMAL(10, 5),
   product_id INT,
   order_status BOOLEAN,
   PRIMARY KEY (order_id) NOT ENFORCED
 ) WITH (
   'connector' = 'mysql-cdc',
   'hostname' = 'mysql',
   'port' = '3306',
   'username' = 'root',
   'password' = '123456',
   'database-name' = 'mydb',
   'table-name' = 'orders'
 );

CREATE TABLE shipments (
   shipment_id INT,
   order_id INT,
   origin STRING,
   destination STRING,
   is_arrived BOOLEAN,
   PRIMARY KEY (shipment_id) NOT ENFORCED
 ) WITH (
   'connector' = 'postgres-cdc',
   'hostname' = 'postgres',
   'port' = '5432',
   'username' = 'postgres',
   'password' = 'postgres',
   'database-name' = 'postgres',
   'schema-name' = 'public',
   'table-name' = 'shipments'
 );

CREATE TABLE enriched_orders (
   order_id INT,
   order_date TIMESTAMP(0),
   customer_name STRING,
   price DECIMAL(10, 5),
   product_id INT,
   order_status BOOLEAN,
   product_name STRING,
   product_description STRING,
   shipment_id INT,
   origin STRING,
   destination STRING,
   is_arrived BOOLEAN,
   PRIMARY KEY (order_id) NOT ENFORCED
 ) WITH (
     'connector' = 'elasticsearch-7',
     'hosts' = 'http://elasticsearch:9200',
     'index' = 'enriched_orders'
 );

 INSERT INTO enriched_orders
 SELECT o.*, p.name, p.description, s.shipment_id, s.origin, s.destination, s.is_arrived
 FROM orders AS o
 LEFT JOIN products AS p ON o.product_id = p.id
 LEFT JOIN shipments AS s ON o.order_id = s.order_id;
```
##### 4.Make some changes in MySQL and Postgres, then check the result in Elasticsearch:
```sql
--MySQL
INSERT INTO orders
VALUES (default, '2020-07-30 15:22:00', 'Jark', 29.71, 104, false);

--PG
INSERT INTO shipments
VALUES (default,10004,'Shanghai','Beijing',false);

--MySQL
UPDATE orders SET order_status = true WHERE order_id = 10004;

--PG
UPDATE shipments SET is_arrived = true WHERE shipment_id = 1004;

--MySQL
DELETE FROM orders WHERE order_id = 10004;

--PG
DELETE FROM shipments WHERE shipment_id = 1004;
```
##### 5.Kafka Changelog JSON format
```shell script
docker exec cdc_sql-client_1 sql-client.sh
```
```sql
CREATE TABLE kafka_gmv (
   day_str STRING,
   gmv DECIMAL(10, 6)
 ) WITH (
     'connector' = 'kafka',
     'topic' = 'kafka_gmv',
     'scan.startup.mode' = 'earliest-offset',
     'properties.bootstrap.servers' = 'kafka:9094',
     'format' = 'changelog-json'
 );

CREATE TABLE kafka_shipments (
   shipment_id INT,
   order_id INT,
   origin STRING,
   destination STRING,
   is_arrived BOOLEAN
 ) WITH (
    'connector' = 'kafka',
    'topic' = 'kafka_shipments',
    'scan.startup.mode' = 'earliest-offset',
    'properties.bootstrap.servers' = 'kafka:9094',
    'format' = 'changelog-json'
 );
INSERT INTO kafka_shipments
 SELECT shipment_id,order_id,origin,destination,is_arrived
 FROM shipments;


INSERT INTO kafka_gmv
 SELECT DATE_FORMAT(order_date, 'yyyy-MM-dd') as day_str, SUM(price) as gmv
 FROM orders
 WHERE order_status = true
 GROUP BY DATE_FORMAT(order_date, 'yyyy-MM-dd');

-- Consumer changelog data from Kafka, and check the result of materialized view: 
SELECT * FROM kafka_gmv;
```
```shell script
docker-compose exec kafka bash -c 'kafka-console-consumer.sh --topic user_behavior --bootstrap-server kafka:9094 --from-beginning --max-messages 10'
```
```sql
--mysql
UPDATE orders SET order_status = true WHERE order_id = 10001;
UPDATE orders SET order_status = true WHERE order_id = 10002;
UPDATE orders SET order_status = true WHERE order_id = 10003;

INSERT INTO orders
VALUES (default, '2020-07-30 17:33:00', 'Timo', 50.00, 104, true);

INSERT INTO orders
VALUES (default, '2020-08-31 17:33:00', 'Timos', 50.00, 102, true);


UPDATE orders SET price = 40.00 WHERE order_id = 10005;

DELETE FROM orders WHERE order_id = 10005;
```
##### 6.hudi test
```sql
--mysql
create table users
(
    id bigint auto_increment primary key,
    name varchar(20) null,
    birthday timestamp default CURRENT_TIMESTAMP not null,
    ts timestamp default CURRENT_TIMESTAMP not null
);
 
// 随意插入几条数据
insert into users (name) values ('hello');
insert into users (name) values ('world');
insert into users (name) values ('iceberg');
insert into users (id,name) values (4,'spark');
insert into users (name) values ('hudi');
 
select * from users;
update users set name = 'hello spark'  where id = 5;
delete from users where id = 5;
```
```sql
CREATE TABLE kafka_users (
   name STRING,
   birth TIMESTAMP(3)
 ) WITH (
     'connector' = 'kafka',
     'topic' = 'kafka_users',
     'scan.startup.mode' = 'earliest-offset',
     'properties.bootstrap.servers' = 'kafka:9094',
     'format' = 'changelog-json'
 );

INSERT INTO kafka_users
SELECT name,birthday from mysql_users;
```
```sql
--flink table
CREATE TABLE mysql_users (
                             id BIGINT PRIMARY KEY NOT ENFORCED ,
                             name STRING,
                             birthday TIMESTAMP(3),
                             ts TIMESTAMP(3)
) WITH (
      'connector' = 'mysql-cdc',
      'hostname' = 'mysql',
      'port' = '3306',
      'username' = 'root',
      'password' = '123456',
      'server-time-zone' = 'Asia/Shanghai',
      'database-name' = 'mydb',
      'table-name' = 'users'
      );
 
// 2.创建hudi表
CREATE TABLE hudi_users2
(
    id BIGINT PRIMARY KEY NOT ENFORCED,
    name STRING,
    birthday TIMESTAMP(3),
    ts TIMESTAMP(3),
    `partition` VARCHAR(20)
) PARTITIONED BY (`partition`) WITH (
    'connector' = 'hudi',
    'table.type' = 'MERGE_ON_READ',
    'path' = 'hdfs://namenode:8020/hudi/hudi_users2',
    'read.streaming.enabled' = 'true',
    'read.streaming.check-interval' = '1' 
);

INSERT INTO hudi_users2 SELECT *, DATE_FORMAT(birthday, 'yyyyMMdd') FROM mysql_users;


 #查询表数据，设置一下查询模式为tableau
set execution.result-mode=tableau;

select * from t1;
```

```sql
INSERT INTO hudi_users2(id,name,birthday,ts, `partition`) SELECT id,name,birthday,ts,DATE_FORMAT(birthday, 'yyyyMMdd') FROM mysql_users;
```