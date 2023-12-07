
Install Docker with docker.sh

    Enter the Portainer UI: 

    https://192.168.56.125:9443/

    Create Portainer Admin User **

    Inspect Portainer **

    Docker Compose Up


akHQ: http://192.168.56.125:28107/
kafka-connect-ui: http://192.168.56.125:28103/
jupyter: http://192.168.56.125:28888
ksqldb: http://192.168.56.125:8088/info



ubuntu@kafka:~/my_kafka_workshop$ -----------------------------------

# Enter kafka-1 Container
sudo docker exec -ti kafka-1 bash

[appuser@kafka-1 ~]$ ------------------------------------------------

kafka-topics --create --topic my_topic --bootstrap-server kafka-1:19092,kafka-2:19093 --partitions 3

kafka-topics --list --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094

kafka-topics --delete --topic my_topic --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094

kafka-console-consumer --bootstrap-server kafka-1:19092,kafka-2:19093 --topic movie-json --consumer-property group.id=CG1

kafka-console-producer --bootstrap-server kafka-1:19092,kafka-2:19093 --topic movie-json


Alternatives:
---------------------------------------------------
# CREATE TOPIC
sudo docker compose exec kafka-1 kafka-topics \
  --create \
  --topic msg_delivery_app \
  --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094 \
  --partitions 1 \
  --replication-factor 1

# LIST TOPICS
sudo docker compose exec kafka-1 kafka-topics \
  --list \
  --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094

# DELETE TOPIC
sudo docker compose exec kafka-1 kafka-topics \
  --delete \
  --topic msg_delivery_app \
  --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094

# Python App Topic
sudo docker compose exec kafka-1 kafka-topics \
  --create \
  --topic wordcount-record \
  --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094 \
  --partitions 3 \
  --replication-factor 1

# KafkaCat Query --------------
/ # kcat -C -b kafka-1:19092,kafka-2:19093,kafka-3:19094 -t wordcount-record -o -200 -e
  Kafka is awesome!
  % Reached end of topic wordcount-record [1] at offset 1
  % Reached end of topic wordcount-record [2] at offset 0
  % Reached end of topic wordcount-record [0] at offset 0: exiting

------------------------------------
Exercise: Using Consumer Group
------------------------------------
In this exercise you will create a consumer group and observe what happens when the number of Kafka consumers changes.

TIP: Use kafkacat for more control (than kafka-console-producer that does not support sending messages to a given partition).

Procedure

    Create a topic t1 with 3 partitions

./bin/kafka-topics.sh --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094 --create --topic t1 --partitions 3

    Start a new consumer c1 in a consumer group CG1

./bin/kafka-console-consumer.sh --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094 --topic t1 --consumer-property group.id=CG1

    Start a Kafka producer that is attached to partition 0. Send a couple of messages with the key 0 for easier identification what producer sends what messages.

kafkacat -P -b kafka-1:19092,kafka-2:19093,kafka-3:19094 -t t1 -p 0 -K :
(-K is for a key-value separator)

    Start another Kafka producer to send messages to partition 2 (with 2 key)

kafkacat -P -b kafka-1:19092,kafka-2:19093,kafka-3:19094 -t t1 -p 2 -K :

    At this point you should have 3 partitions, 2 producers and 1 consumer. Observe what and how messages are consumed. Simply send messages so you can identity what message used what partition.

    Start a new consumer c2 in the CG1 consumer group

./bin/kafka-console-consumer.sh --bootstrap-server :9092 --topic t1 --consumer-property group.id=CG1
    
    Observe the logs in the Kafka broker

    Send a couple of message to observe if and how messages are distributed across the two consumers

    Start a new consumer c3 in the CG1 consumer group

./bin/kafka-console-consumer.sh --bootstrap-server :9092 --topic t1 --consumer-property group.id=CG1
    
    Send a couple of message to observe if and how messages are distributed across the consumers

    Shut down any of the running consumers and observe which consumer takes over the "abandoned" partition


Open 4 terminal:
    1 kcat (producer)
    3 consumer

# Send Messages to Specific Partition with Kafkacat 

/# echo "new_message_0" | kcat -P -b kafka-1:19092,kafka-2:19093 -t t1 -p 0 -K :

/# echo "new_message_2" | kcat -P -b kafka-1:19092,kafka-2:19093 -t t1 -p 2 -K :


------------------------------------
Exercise: Configuring Multi-Broker Kafka Cluster
------------------------------------

sudo docker compose exec kafka-1 kafka-topics \
  --create \
  --topic msg_delivery_app_p3 \
  --bootstrap-server kafka-1:19092,kafka-2:19093,kafka-3:19094 \
  --partitions 3 \
  --replication-factor 1

ubuntu@kafka:~/my_kafka_workshop$ sudo docker exec -it ksqldb-server-1 /bin/bash
[appuser@ksqldb-server-1 ~]$
[appuser@ksqldb-server-1 ~]$ ksql
SLF4J: A number (123) of logging calls during the initialization phase have been intercepted and are
SLF4J: now being replayed. These are subject to the filtering rules of the underlying logging system.
SLF4J: See also http://www.slf4j.org/codes.html#replay

                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =        The Database purpose-built       =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2022 Confluent Inc.

CLI v0.29.0, Server v<unknown> located at http://localhost:8088

WARNING: Could not identify server version.
         Non-matching CLI and server versions may lead to unexpected errors.

Server Status: <unknown>

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!


*************************************ERROR**************************************
Remote server at http://localhost:8088 does not appear to be a valid KSQL
server. Please ensure that the URL provided is for an active KSQL server.

The server responded with the following error:
Error issuing GET to KSQL server. path:/info
Caused by: io.netty.channel.AbstractChannel$AnnotatedConnectException:
        Connection refused: localhost/127.0.0.1:8088
Caused by: Could not connect to the server. Please check the server details are
        correct and that the server is running.
********************************************************************************

ksql>
CREATE STREAM joined_movies_stream AS
    SELECT s.production_companies AS production_companies,
		   s.id AS id,
           s.title AS title,
           s.release_year AS release_year,
           s.country AS country,
           s.genres AS genres,
           s.actors AS actors,
           s.director AS director,
           s.composers AS composers,
           s.screenwriters AS screenwriters,
           s.cinematographers AS cinematographers
    FROM MOVIES_JSON s
    LEFT JOIN MOVIES_JSON_SET2 t
    ON s.id = t.id;

 Message
----------------
 Stream created
----------------
ksql>
ksql> SELECT * FROM movies_json;
+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+
|ID                        |TITLE                     |RELEASE_YEAR              |COUNTRY                   |GENRES                    |ACTORS                    |DIRECTOR                  |COMPOSERS                 |SCREENWRITERS             |CINEMATOGRAPHERS          |PRODUCTION_COMPANIES      |
+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+--------------------------+
|1                         |Inception                 |2010                      |United States             |[Action, Sci-Fi, Thriller]|[Leonardo DiCaprio, Joseph|Christopher Nolan         |[Hans Zimmer]             |[Christopher Nolan]       |[Wally Pfister]           |Warner Bros.              |
|                          |                          |                          |                          |                          | Gordon-Levitt, Ellen Page|                          |                          |                          |                          |                          |
|                          |                          |                          |                          |                          |]                         |                          |                          |                          |                          |                          |
|2                         |The Shawshank Redemption  |1994                      |United States             |[Drama]                   |[Tim Robbins, Morgan Freem|Frank Darabont            |[Thomas Newman]           |[Frank Darabont]          |[Roger Deakins]           |Columbia Pictures         |
|                          |                          |                          |                          |                          |an]                       |                          |                          |                          |                          |                          |
|3                         |The Godfather             |1972                      |United States             |[Crime, Drama]            |[Marlon Brando, Al Pacino,|Francis Ford Coppola      |[Nino Rota]               |[Mario Puzo, Francis Ford |[Gordon Willis]           |Paramount Pictures        |
|                          |                          |                          |                          |                          | James Caan]              |                          |                          |Coppola]                  |                          |                          |
|4                         |Pulp Fiction              |1994                      |United States             |[Crime, Drama]            |[John Travolta, Uma Thurma|Quentin Tarantino         |[Various Artists]         |[Quentin Tarantino, Roger |[Andrzej Sekuła]          |Miramax Films             |
|                          |                          |                          |                          |                          |n, Samuel L. Jackson]     |                          |                          |Avary]                    |                          |                          |
|5                         |The Dark Knight           |2008                      |United States             |[Action, Crime, Drama]    |[Christian Bale, Heath Led|Christopher Nolan         |[Hans Zimmer, James Newton|[Jonathan Nolan, Christoph|[Wally Pfister]           |Warner Bros.              |
|                          |                          |                          |                          |                          |ger, Aaron Eckhart]       |                          | Howard]                  |er Nolan]                 |                          |                          |
|6                         |Forrest Gump              |1994                      |United States             |[Drama, Romance]          |[Tom Hanks, Robin Wright, |Robert Zemeckis           |[Alan Silvestri]          |[Eric Roth]               |[Don Burgess]             |Paramount Pictures        |
|                          |                          |                          |                          |                          |Gary Sinise]              |                          |                          |                          |                          |                          |
|7                         |The Matrix                |1999                      |United States             |[Action, Sci-Fi]          |[Keanu Reeves, Laurence Fi|Lana Wachowski            |[Don Davis]               |[Lana Wachowski, Lilly Wac|[Bill Pope]               |Warner Bros.              |
|                          |                          |                          |                          |                          |shburne, Carrie-Anne Moss]|                          |                          |howski]                   |                          |                          |
|8                         |Schindler's List          |1993                      |United States             |[Biography, Drama, History|[Liam Neeson, Ben Kingsley|Steven Spielberg          |[John Williams]           |[Steven Zaillian]         |[Janusz Kamiński]         |Universal Pictures        |
|                          |                          |                          |                          |]                         |, Ralph Fiennes]          |                          |                          |                          |                          |                          |
|9                         |Titanic                   |1997                      |United States             |[Drama, Romance]          |[Leonardo DiCaprio, Kate W|James Cameron             |[James Horner]            |[James Cameron]           |[Russell Carpenter]       |Paramount Pictures        |
|                          |                          |                          |                          |                          |inslet, Billy Zane]       |                          |                          |                          |                          |                          |
|10                        |The Silence of the Lambs  |1991                      |United States             |[Crime, Drama, Thriller]  |[Jodie Foster, Anthony Hop|Jonathan Demme            |[Howard Shore]            |[Ted Tally]               |[Tak Fujimoto]            |Orion Pictures            |
|                          |                          |                          |                          |                          |kins, Scott Glenn]        |                          |                          |                          |                          |                          |


------------------- Jupyter App -----------------------------------
# Get Token from logs, Enter the Browser (http://192.168.56.125:28888)
sudo docker logs jupyter

http://192.168.56.125:28888/login?next=%2Flab%3F

[I 2023-12-06 16:00:05.155 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 2023-12-06 16:00:05.156 ServerApp]

    To access the server, open this file in a browser:
        file:///home/jovyan/.local/share/jupyter/runtime/jpserver-20-open.html
    Or copy and paste one of these URLs:
        http://jupyter:8888/lab?token=31c7212c30e917dd1db0cc7733ddda963c644bd9330b4799
        http://127.0.0.1:8888/lab?token=31c7212c30e917dd1db0cc7733ddda963c644bd9330b4799
[I 2023-12-06 16:00:06.965 ServerApp] Skipped non-installed server(s): bash-language-server, dockerfile-language-server-nodejs, javascript-typescript-langserver, jedi-language-server, julia-language-server, pyright, python-language-server, python-lsp-server, r-languageserver, sql-language-server, texlab, typescript-language-server, unified-language-server, vscode-css-languageserver-bin, vscode-html-languageserver-bin, vscode-json-languageserver-bin, yaml-language-server
[I 2023-12-06 16:04:29.380 ServerApp] 302 GET / (@192.168.56.1) 0.35ms
[I 2023-12-06 16:04:29.383 LabApp] 302 GET /lab? (@192.168.56.1) 0.52ms
[W 2023-12-06 16:04:58.993 ServerApp] 401 POST /login?next=%2Flab%3F (@192.168.56.1) 0.98ms referer=http://192.168.56.125:28888/login?next=%2Flab%3F
....
....




Exercise: Word Count Per Record
--------------------------------------
Write a new Kafka application WordCountPerLineApp (using Kafka Producer and Consumer APIs) that does the following:

* Consumes records from a topic, e.g. input
* Counts words (in the value of a record)
* Produces records with the unique words and their occurences (counts)
    A record key -> hello hello world gives a record with the following value hello -> 2, world -> 1 (and the same key as in the input record)
    (EXTRA) Produces as many records as there are unique words in the input record with their occurences (counts)
    A record key -> hello hello world gives two records in the output, i.e. (hello, 2) and (world, 1 (as (key, value))

------------------- KAFKA CONNECT UI --------------------------------------

http://192.168.56.125:28103/#/

Working with Kafka Connect and Change Data Capture (CDC)

    Creating Postgresql Database cdc_demo 

docker exec -ti postgresql psql -d postgres -U postgres

CREATE SCHEMA IF NOT EXISTS cdc_demo;

SET search_path TO cdc_demo;

DROP TABLE IF EXISTS address;
DROP TABLE IF EXISTS person;

CREATE TABLE "cdc_demo"."person" (
    "id" integer NOT NULL,
    "title" character varying(8),
    "first_name" character varying(50),
    "last_name" character varying(50),
    "email" character varying(50),
    "modifieddate" timestamp DEFAULT now() NOT NULL,
    CONSTRAINT "person_pk" PRIMARY KEY ("id")
);

CREATE TABLE "cdc_demo"."address" (
    "id" integer NOT NULL,
    "person_id" integer NOT NULL,
    "street" character varying(50),
    "zip_code" character varying(10),
    "city" character varying(50),
    "modifieddate" timestamp DEFAULT now() NOT NULL,
    CONSTRAINT "address_pk" PRIMARY KEY ("id")
);

ALTER TABLE ONLY "cdc_demo"."address" ADD CONSTRAINT "address_person_fk" FOREIGN KEY (person_id) REFERENCES person(id) NOT DEFERRABLE;

    Create the necessary topics
    Let's create a topic for the change records of the two tables 'personandaddress`:

*******************************
    How we can use the JDBC Source Connector to perform polling-based CDC on the two tables.
*******************************

docker exec -ti kafka-1 kafka-topics --bootstrap-server kafka-1:19092 --create --topic priv.person.cdc.v1 --partitions 8 --replication-factor 3

docker exec -ti kafka-1 kafka-topics --bootstrap-server kafka-1:19092 --create --topic priv.address.cdc.v1 --partitions 8 --replication-factor 3

    Now let's start two consumers, one on each topic, in separate terminal windows

    Create some initial data in Postgresql
    Let's add a first person and address as a sample. In a terminal connect to Postgresql

docker exec -ti postgresql psql -d postgres -U postgres

    execute the following SQL INSERT statements

INSERT INTO cdc_demo.person (id, title, first_name, last_name, email)
VALUES (1, 'Mr', 'Peter', 'Muster', 'peter.muster@somecorp.com');

INSERT INTO cdc_demo.address (id, person_id, street, zip_code, city)
VALUES (1, 1, 'Somestreet 10', '9999', 'Somewhere');

    Create a JDBC Source connector instance
    Now let's create and start the JDBC Source connector

curl -X "POST" "$DOCKER_HOST_IP:8083/connectors" \
     -H "Content-Type: application/json" \
     -d $'{
  "name": "person.jdbcsrc.cdc",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "tasks.max": "1",
    "connection.url":"jdbc:postgresql://postgresql/postgres?user=postgres&password=abc123!",
    "mode": "timestamp",
    "timestamp.column.name":"modifieddate",
    "poll.interval.ms":"10000",
    "table.whitelist":"cdc_demo.person,cdc_demo.address",
    "validate.non.null":"false",
    "topic.prefix":"priv.",
    "key.converter":"org.apache.kafka.connect.storage.StringConverter",
    "key.converter.schemas.enable": "false",
    "value.converter":"org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false",
    "name": "person.jdbcsrc.cdc",
     "transforms":"createKey,extractInt,addSuffix",
     "transforms.createKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
     "transforms.createKey.fields":"id",
     "transforms.extractInt.type":"org.apache.kafka.connect.transforms.ExtractField$Key",
     "transforms.extractInt.field":"id",
     "transforms.addSuffix.type":"org.apache.kafka.connect.transforms.RegexRouter",
     "transforms.addSuffix.regex":".*",
     "transforms.addSuffix.replacement":"$0.cdc.v1"
  }
}'

    The key get's extracted using two Single Message Transforms (SMT), the ValueToKey and the ExtractField. They are chained together to set the key for data coming from a JDBC Connector. During the transform, ValueToKey copies the message id field into the message key and then ExtractField extracts just the integer portion of that field.

        "transforms.createKey.type":"org.apache.kafka.connect.transforms.ValueToKey",
        "transforms.createKey.fields":"id",
        "transforms.extractInt.type":"org.apache.kafka.connect.transforms.ExtractField$Key",
        "transforms.extractInt.field":"id",

    To also add a suffix to the topic (we have added a prefix using the topic.prefix configuration) we can use the RegexRouter SMT

        "transforms.addSuffix.type":"org.apache.kafka.connect.transforms.RegexRouter",
        "transforms.addSuffix.regex":".*",
        "transforms.addSuffix.replacement":"$0.cdc.v1"

    Add more data

    Add a new address to person with id=1.

INSERT INTO cdc_demo.address (id, person_id, street, zip_code, city)
VALUES (2, 1, 'Holiday 10', '1999', 'Ocean Somewhere');

    Now let's add a new person, first without an address

INSERT INTO cdc_demo.person (id, title, first_name, last_name, email)
VALUES (2, 'Ms', 'Karen', 'Muster', 'karen.muster@somecorp.com');

    and now also add the address

INSERT INTO cdc_demo.address (id, person_id, street, zip_code, city)
VALUES (3, 2, 'Somestreet 10', '9999', 'Somewhere');

    Remove the connector
    We have successfully tested the query-based CDC using the JDBC Source connector. So let's remove the connector.

curl -X "DELETE" "http://$DOCKER_HOST_IP:8083/connectors/person.jdbcsrc.cdc"


*******************************
Log-based CDC using Debezium and Kafka Connect 
*******************************


Let's add a first person and address as a sample. In a terminal connect to Postgresql

docker exec -ti postgresql psql -d postgres -U postgres
and execute the following SQL TRUNCATE followed by INSERT statements

TRUNCATE cdc_demo.person CASCADE;

INSERT INTO cdc_demo.person (id, title, first_name, last_name, email)
VALUES (1, 'Mr', 'Peter', 'Muster', 'peter.muster@somecorp.com');

INSERT INTO cdc_demo.address (id, person_id, street, zip_code, city)
VALUES (1, 1, 'Somestreet 10', '9999', 'Somewhere');

    Create the Debezium connector
    Now with the connector installed and the Kafka Connect cluster restarted, let's create and start the connector:

curl -X PUT \
  "http://$DOCKER_HOST_IP:8083/connectors/person.dbzsrc.cdc/config" \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -d '{
  "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
  "tasks.max": "1",
  "database.server.name": "postgresql",
  "database.port": "5432",
  "database.user": "postgres",
  "database.password": "abc123!",  
  "database.dbname": "postgres",
  "schema.include.list": "cdc_demo",
  "table.include.list": "cdc_demo.person, cdc_demo.address",
  "plugin.name": "pgoutput",
  "tombstones.on.delete": "false",
  "database.hostname": "postgresql",
  "topic.creation.default.replication.factor": 3,
  "topic.creation.default.partitions": 8,
  "topic.creation.default.cleanup.policy": "compact"
}'