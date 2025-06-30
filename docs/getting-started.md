# Getting Started with Clickstream DataOps Demo

This guide helps you set up and run a simplified DataOps pipeline using open-source tools. You’ll simulate real-time clickstream events, validate schemas, run a streaming job, and monitor it using Prometheus and Grafana.

## Prerequisites

- Docker and Docker Compose installed
- Git (optional for version control)
- Python 3 (for event simulation)


## Step 1: Clone and Launch the Stack

```bash
git clone https://github.com/your-org/clickstream-dataops.git
cd clickstream-dataops
docker-compose up -d
```

This launches:
- Kafka + Zookeeper
- Flink JobManager + TaskManager
- Apicurio Schema Registry
- Prometheus + Grafana

## Step 2: Register Your Schema

Use the CLI or `curl` to register your Avro schema:

```bash
curl -X POST http://localhost:8081/apis/registry/v2/groups/default/artifacts   -H "Content-Type: application/json"   -H "X-Registry-ArtifactId: click-event"   --data-binary @schemas/click_event.avsc
```

## Step 3: Simulate Clickstream Events

Use the Python script in `test/data/` to publish events to Kafka:

```bash
python test/data/simulate_clickstream.py
```

Or manually run:

```bash
kcat -b localhost:9092 -t clickstream -P < test/data/sample-clickstream.json
```



## ⚙️ Step 4: Run Flink SQL Job

Access the Flink UI at [http://localhost:8082](http://localhost:8082)  
Submit your SQL or JAR job using the schema fields from `ClickEvent`.


## Step 5: CI/CD Validation (Optional)

Push a change and verify schema + test validation via GitHub Actions.  
Workflow defined in `.github/workflows/ci.yml`.

## Step 6: Monitor and Alert

- Visit Prometheus: [http://localhost:9090](http://localhost:9090)
- Visit Grafana: [http://localhost:3000](http://localhost:3000)
  - Username: `admin` | Password: `admin`

Check dashboards for:
- Kafka lag
- Flink latency
- Job health


## Clean Up

```bash
docker-compose down -v
```

