# CI/CD process for real-time data pipelines

This guide explains how to implement Continuous Integration and Continuous Deployment (CI/CD) for real-time data pipelines using GitHub Actions. These practices help teams ship changes confidently while enforcing schema compatibility, running automated tests, and safely deploying to staging and production environments.

## Why CI/CD for Data?

- Automatically catch schema-breaking changes
- Run tests on pipeline logic with synthetic data
- Deploy safely to dev/staging/prod environments
- Enable fast iteration without breaking production

## 1. Define GitHub Actions Workflow

Create a file at `.github/workflows/ci.yml`:

```yaml
name: CI for Clickstream Pipeline

on: [push]

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Build pipeline logic
        run: ./gradlew shadowJar

      - name: Validate Avro Schema
        run: ./ci/validate-schema.sh

      - name: Run integration tests
        run: ./test/integration/run-tests.sh

      - name: Deploy to staging
        run: ./deploy/deploy.sh staging

      - name: Canary deploy to prod
        run: ./deploy/deploy.sh production --canary
```
## 2. Schema Validation Script
In ci/validate-schema.sh, use curl to validate schema compatibility:

```bash
#!/bin/bash

curl -X POST http://localhost:8081/apis/registry/v2/groups/default/artifacts/click-event/versions \
  -H "Content-Type: application/json" \
  --data-binary @schemas/click_event.avsc
 ``` 
If the schema breaks compatibility, the CI pipeline fails.

## 3. Add Integration Tests
Use test data to validate transformations:

Input file: `test/data/sample-clickstream.json`

Expected output: `test/data/expected-output.json`

```bash 
./test/integration/run-tests.sh
```
## 4. Safe Deploys and Environment Parity
Use templated config files for each environment (config/dev/, config/staging/, config/prod/). This ensures the same code runs everywhere, but with isolated parameters.

## 5. Automate Canary Deploys
Canary deploys run the new job in parallel with the old one on a small portion of traffic. You can monitor output consistency and metrics before a full cutover.

