# How to set up Apicurio Schema Registry

This guide walks you through setting up [Apicurio Registry](https://www.apicur.io/) to manage versioned schemas for real-time data pipelines. Schema registries ensure producers and consumers share a consistent contract, and prevent breaking changes from silently affecting downstream systems.

## Why Use a Schema Registry?

* **Version control for schemas** (like Avro or Protobuf)
* **Enforce compatibility rules** (backward, forward, full)
* **Prevent silent data corruption** in Kafka/Flink pipelines
* **Promote trust and observability** in data contracts

## 1. Run Apicurio Schema Registry (Docker)

To start the registry locally (in-memory version):

```bash
docker run -d \
  --name apicurio-registry \
  -p 8081:8080 \
  apicurio/apicurio-registry-mem:2.3.2.Final
```

To run with Kafka using Docker Compose, include this in your `docker-compose.yml`:

```yaml
apicurio:
  image: apicurio/apicurio-registry-mem:2.3.2.Final
  ports:
    - "8081:8080"
  environment:
    QUARKUS_PROFILE: prod
```

Then start the services:

```bash
docker-compose up -d
```

## 2. Access the Registry UI and API

Once running:

* Web UI: [http://localhost:8081/ui](http://localhost:8081/ui)
* REST API Base: `http://localhost:8081/apis/registry/v2`

You can verify it's live with:

```bash
curl http://localhost:8081/health
```

## 3. Register an Avro Schema

Assume you have a file called `click_event.avsc`:

```json
{
  "type": "record",
  "name": "ClickEvent",
  "fields": [
    {"name": "user_id", "type": "long"},
    {"name": "event_type", "type": "string"},
    {"name": "timestamp", "type": "long"}
  ]
}
```

To register it via CLI:

```bash
curl -X POST http://localhost:8081/apis/registry/v2/groups/default/artifacts \
  -H "Content-Type: application/json" \
  -H "X-Registry-ArtifactId: click-event" \
  --data-binary @schemas/click_event.avsc
```

## 4. Enable Compatibility Checks (Optional)

Set schema compatibility to prevent breaking changes:

```bash
curl -X PUT http://localhost:8081/apis/registry/v2/groups/default/artifacts/click-event/rules/COMPATIBILITY \
  -H "Content-Type: application/json" \
  --data '"BACKWARD"'
```

Other options: `"FORWARD"`, `"FULL"`, `"NONE"`


## 5. Check Compatibility in CI

You can use this in a GitHub Actions or CI script:

```bash
curl -X POST http://localhost:8081/apis/registry/v2/groups/default/artifacts/click-event/versions \
  -H "Content-Type: application/json" \
  --data-binary @schemas/click_event.avsc
```

This POST will fail if the schema breaks the configured compatibility rule — blocking the merge.

## More Resources

* [Apicurio Registry Docs](https://www.apicur.io/registry/docs/)
* [REST API Reference](https://www.apicur.io/registry/docs/apicurio-registry/2.3.x/api/registry-rest-api.html)
* [Confluent Compatibility Modes (for reference)](https://docs.confluent.io/platform/current/schema-registry/avro.html#compatibility)
