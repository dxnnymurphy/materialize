# materialize
Using Materialize with adminer UI to load Kafka json messages into views which can then be queried and shown in a Grafana dashboard.

# Prerequisites
1) Requires a Kafka instance with json messages


# Usage

1) Logging in to the Adminer UI -
```
System -> PostgreSQL
Server -> materialized:6875
User -> materialize
Password ->
Database -> materialize
```

This SQL console is an easy way of trying out different SQL queries on the postgres database before they can either be provisioned or discarded. The postgres-sql psql cli tool can also be used for this purpose if preferred.

NB: Adminer v4.6.2 or lower must be used as any higher version does not allow connection to databases without a password, and materialize does not currently support changing the user from the default materialize user (with no password).

2) Creating a Kafka source in Materialize -
```
CREATE SOURCE $source_name
FROM KAFKA BROKER '$kafkabroker' TOPIC '$kafkatopic'
FORMAT BYTES;
```

3) Creating a materialized view of the raw json messages from the Kafka topic - 
```
SELECT convert_from(data, 'utf8')::jsonb AS json FROM $source_name;
```

4) Creating a materialized view of relevant parsed data from the raw topic - 
```
  SELECT DISTINCT ON (ID)
     json->'message'->'META'->'latestTrade'->'p' AS Price,
     json->'message'->'META'->'latestTrade'->'s' AS Qty,
     json->'message'->'META'->'latestTrade'->'i' AS ID,
     json->'message'->'META'->'latestTrade'->'x' AS Exchange,
     json->'message'->'META'->'latestTrade'->'t' AS Timestamp
  FROM $raw_json_view;
```

Steps 3 and 4 can be combined into one SQL Query but as it is likely that multiple views will be created from the messages in each topic it is easier to create a separate view and query from that.

5) Provisioning Sources and Views - 
There are likely sources and views that are important to the user which will need to be present even when the container is restarted, this provisioning is done via the execution of a bash script within the materialized-bootstrap container. 

This requires use of the postgresql-client, and is easily extendible by adding new lines to the CreateSource or CreateView functions as follows:
```
psql -U $USER -h $HOST -p $PORT $DB -c "INSERT SQL QUERY HERE"
```
The following arguments can be used to configure the use to match your postgres instance:
1) USER -> DB username
2) HOST -> DB hostname
3) PORT -> DB port
4) DB -> DB name