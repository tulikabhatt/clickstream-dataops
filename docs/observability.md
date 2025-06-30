# Adding metrics with Prometheus and Grafana

Real-time pipelines are always on — and when they break, they often do so silently. Observability is how you detect, debug, and recover from failures fast.

This guide explains how to expose metrics, visualize them in Grafana, and configure alerts using Prometheus.



## 1. Why Observability?

- Detect lag, data loss, schema mismatches
- Monitor job health and throughput
- Alert on symptoms before users are impacted

## 2. Expose Metrics from Flink

Update `flink-conf.yaml`:

```yaml
metrics.reporter.prom.class: org.apache.flink.metrics.prometheus.PrometheusReporter
metrics.reporter.prom.port: 9250
```

Start Flink with this config. Prometheus will scrape metrics at `http://flink-jobmanager:9250/metrics`.



## 3. Prometheus Config (prometheus.yml)

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'flink'
    static_configs:
      - targets: ['jobmanager:9250']

  - job_name: 'kafka'
    static_configs:
      - targets: ['kafka:9101']
```


## 4. Visualize in Grafana

- Login: http://localhost:3000 (default user: admin/admin)
- Add Prometheus as a data source
- Create dashboards:
  - Kafka consumer lag
  - Flink job latency & checkpoint time
  - Data freshness or throughput by event type



## 5. Alerting

Use Grafana or Prometheus alert rules:

```yaml
alert:
  - alert: HighLag
    expr: kafka_consumer_lag > 5000
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "Kafka consumer lag too high"
```

