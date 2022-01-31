# Prometheus exporter for Confluent Cloud Metrics API

A simple Prometheus exporter that can be used to extract metrics from [Confluent Cloud Metric API](https://docs.confluent.io/current/cloud/metrics-api.html).
By default, the exporter will be exposing the metrics on [port 2112].  When launching with Docker Compose, metrics are displayed via a Grafana dashboard on [http://localhost:3000](http://localhost:3000) (`admin`/`admin`).

To use the exporter, the following environment variables need to be specified:

* `CCLOUD_API_KEY`: The API Key created with `ccloud api-key create --resource cloud`
* `CCLOUD_API_SECRET`: The API Key Secret created with `ccloud api-key create --resource cloud`

`CCLOUD_API_KEY` and `CCLOUD_API_SECRET` environment variables will be used to invoke the https://api.telemetry.confluent.cloud endpoint.

The Docker Compose launc starts Prometheus on [http://localhost:9090] (http://localhost:9090)

In addition to the metrics exporter and Prometheus containers, the Docker Compose launch starts a [Grafana] on [http://localhost:3000](http://localhost:3000) (`admin`/`admin`).  The launch pre-provisions a Prometheus datasource for the Confluent Cloud metrics and a default dashboard.

The Docker Compose service definitions include data volumes for both Prometheus and Grafana, so metrics data will be retained following `docker-compose down` and restored when containers are started again. To remove these volumes and start with empty Prometheus and Grafana databases, run `docker-compose down --volumes`.
### Examples of configuration files

- A simple configuration to fetch metrics for a cluster: [simple.yaml](./config/config.simple.yaml)
- A configuration to fetch metrics at the partition granularity for a few topics: [partition.yaml](./config/config.partition.yaml)

### Default configuration

```yaml
config:
  http:
    baseurl: https://api.telemetry.confluent.cloud/
    timeout: 60
  listener: 0.0.0.0:2112
  noTimestamp: false
  delay: 60
  granularity: PT1M
  cachedSecond: 30
rules:
  - clusters:
      - $CCLOUD_CLUSTER
    connectors:
      - $CCLOUD_CONNECTOR
    ksqls:
      - $CCLOUD_KSQL
    schemaRegistries:
      - $CCLOUD_SCHEMA_REGISTRY
    metrics:
      - io.confluent.kafka.server/received_bytes
      - io.confluent.kafka.server/sent_bytes
      - io.confluent.kafka.server/received_records
      - io.confluent.kafka.server/sent_records
      - io.confluent.kafka.server/retained_bytes
      - io.confluent.kafka.server/active_connection_count
      - io.confluent.kafka.server/request_count
      - io.confluent.kafka.server/partition_count
      - io.confluent.kafka.server/successful_authentication_count
      - io.confluent.kafka.connect/sent_bytes
      - io.confluent.kafka.connect/received_bytes
      - io.confluent.kafka.connect/received_records
      - io.confluent.kafka.connect/sent_records
      - io.confluent.kafka.connect/dead_letter_queue_records
      - io.confluent.kafka.ksql/streaming_unit_count
      - io.confluent.kafka.schema_registry/schema_count
    labels:
      - kafka_id
      - topic
      - type
```

