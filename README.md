[![Gitter chat](https://badges.gitter.im/gitterHQ/gitter.png)](https://gitter.im/big-data-europe/Lobby)

# docker-hive

This is a docker container for Apache Hive 2.3.2. It is based on https://github.com/big-data-europe/docker-hadoop so check there for Hadoop configurations.
This deploys Hive and starts a hiveserver2 on port 10000.
Metastore is running with a connection to postgresql database.
The hive configuration is performed with HIVE_SITE_CONF_ variables (see hadoop-hive.env for an example).

To run Hive with postgresql metastore:
```
    docker-compose up -d
```

To deploy in Docker Swarm:
```
    docker stack deploy -c docker-compose.yml hive
```

To run a PrestoDB 0.181 with Hive connector:

```
  docker-compose up -d presto-coordinator
```

This deploys a Presto server listens on port `8080`

## Testing
Count number of stations:
```
  $ docker-compose exec hive-server bash
  # hive --database climate
  > SELECT station, count(station) AS cnt FROM data GROUP BY station;
```

## Example Queries

Get the average temperature for each station in a given month

```
SELECT station, avg(hourlydrybulbtemperature) FROM data WHERE from_unixtime(unix_timestamp(time), "YYYY-MM") = "2019-12" GROUP BY station;
```

Get the average temperature for each station for each month

```
SELECT station, from_unixtime(unix_timestamp(time), "YYYY-MM") AS month, avg(hourlydrybulbtemperature) FROM data GROUP BY 2, station CLUSTER BY month;
```

Get the average temperature for a given station for each month

```
SELECT from_unixtime(unix_timestamp(time), "YYYY-MM"), avg(hourlydrybulbtemperature) FROM data WHERE station = "16201" GROUP BY 1;
```

Get the nearest 10 stations to a given city

```
select name, latitude, longitude, SQRT(POW(69.1 * (latitude - result.lat), 2) + POW(69.1 * (result.lng - longitude) * COS(latitude / 57.3), 2)) AS distance
FROM stations LEFT JOIN
(SELECT lat, lng FROM cities WHERE city = "London" AND country = "United Kingdom") result
CLUSTER BY distance LIMIT 10;
```

## Contributors
* Ivan Ermilov [@earthquakesan](https://github.com/earthquakesan) (maintainer)
* Yiannis Mouchakis [@gmouchakis](https://github.com/gmouchakis)
* Ke Zhu [@shawnzhu](https://github.com/shawnzhu)
