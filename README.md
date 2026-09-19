# Grafana MCP Observability Lab

This repository is a study and experimentation environment for observability with OpenTelemetry, Prometheus, Grafana, Loki, Tempo and MCP-based access to telemetry.

## Attribution

This repository is based on course material from the Software Engineering with Applied AI program published by UNIPDS and Erick Wendel.

Upstream material:
https://github.com/unipds-engenharia-de-ia-aplicada/engenharia-de-software-com-ia-aplicada

The upstream attribution is preserved intentionally. This repository should be read as a learning and experimentation workspace, not as an original implementation of the entire stack.

## What this lab demonstrates

- OpenTelemetry instrumentation and collection
- Metrics with Prometheus
- Logs with Loki
- Traces with Tempo
- Grafana dashboards and datasource integration
- Blackbox monitoring
- A Node.js/Fastify sample application
- PostgreSQL instrumentation
- MCP access to observability data
- Local Docker-based infrastructure

## Architecture

```text
Application
    |
    | OTLP
    v
OpenTelemetry Collector
    |
    +--> Prometheus
    +--> Loki
    +--> Tempo
             |
             v
          Grafana
```

The application sends telemetry to the OpenTelemetry Collector. The collector routes each telemetry signal to its corresponding backend. Grafana is used to inspect and correlate the resulting data.

## Main components

### OpenTelemetry Collector

Receives OTLP telemetry and routes traces, logs and metrics to the configured backends.

### Prometheus

Stores metrics and evaluates alerting rules.

### Loki

Stores and queries logs.

### Tempo

Stores distributed traces and supports trace correlation.

### Grafana

Provides dashboards and cross-navigation between metrics, logs and traces.

### Blackbox Exporter

Probes HTTP and network endpoints for availability.

### Demo application

A Node.js/Fastify application instrumented with OpenTelemetry and backed by PostgreSQL.

## Running the lab

Requirements:

- Docker
- Docker Compose
- Node.js 22 or newer

Start the observability infrastructure:

```bash
npm run docker:infra:up
```

Follow the infrastructure logs when needed:

```bash
npm run docker:infra:logs
```

Run the application tests locally:

```bash
cd _alumnus
npm test
```

Run the containerized integration test environment:

```bash
npm run test:docker
```

Clean it up afterwards:

```bash
npm run test:docker:down
```

## MCP usage

The repository includes examples for querying observability data through MCP.

See:

```text
docs/grafana-mcp-prompts.md
```

Example questions include:

- Which alerts are firing?
- Which endpoints have the highest latency?
- Are there errors correlated with a specific trace?
- Which database operations are slow?

## Why I keep this repository public

The value of this repository is the hands-on study of observability concepts and the interaction between telemetry systems and MCP tooling.

I do not present the upstream example itself as original work. The repository is useful as evidence of the environment I studied, configured and explored.

## Notes

Some configuration is intended for local development and demonstration. Review credentials, ports and storage settings before adapting this environment to another context.


## License and distribution

The upstream course repository is published under CC BY-NC-ND 4.0. Its LICENSE.md states that modified or adapted versions may not be distributed under the NoDerivatives condition. See [NOTICE.md](NOTICE.md) for the provenance and licensing note.
