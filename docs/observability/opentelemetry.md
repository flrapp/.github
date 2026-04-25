# OpenTelemetry

Flare exports telemetry — logs, metrics, and traces — via the [OpenTelemetry](https://opentelemetry.io/) protocol. All signals are sent to an OTLP-compatible collector, which routes them to your observability backend of choice.

## Configuration

Set the `FLARE_OTEL_ENDPOINT` environment variable to your OTel Collector's OTLP HTTP endpoint:

```env
FLARE_OTEL_ENDPOINT=http://otel-collector:4318
```

| Behavior | Description |
|---|---|
| Variable set | Telemetry is enabled and exported to the specified endpoint |
| Variable absent | Telemetry is silently disabled — no errors, no retries |

Only **OTLP HTTP** (`4318`) is supported. OTLP gRPC is not.

---

## What Flare Exports

### Metrics

**`flare.flag.evaluations`** — incremented on every SDK evaluation request.

| Tag | Present in | Description |
|---|---|---|
| `project` | `evaluate` + `evaluate-all` | Project alias |
| `scope` | `evaluate` + `evaluate-all` | Environment alias (e.g. `production`) |
| `flag` | `evaluate` only | Flag key. Not present for `evaluate-all` — one request evaluates all flags, so adding this tag would distort the request counter. |

In addition, Flare exports the full set of standard **ASP.NET Core and .NET runtime metrics** — HTTP request duration, active connections, GC pressure, thread pool usage, and more.

#### Example Prometheus / Grafana Queries

```promql
# Overall evaluation RPS
rate(flare_flag_evaluations_total[5m])

# Load by project
sum by (project) (rate(flare_flag_evaluations_total[5m]))

# Load by environment
sum by (scope) (rate(flare_flag_evaluations_total[5m]))

# Top 10 flags by request volume
topk(10, sum by (flag) (rate(flare_flag_evaluations_total[5m])))
```

### Traces

Flare instruments incoming HTTP requests and database operations. Each request produces a trace with spans covering request handling and query execution.

### Logs

Flare emits **structured logs** via OTel. This includes application logs and the audit log stream — a record of all significant changes made in the system. See [Audit Logs](./audit-logs.md) for details.

---

## OTel Collector Setup

Run an OTel Collector alongside Flare to receive telemetry and forward it to your storage backends.

Add it to your `docker-compose.yml`:

```yaml
otel-collector:
  image: otel/opentelemetry-collector-contrib:latest
  container_name: otel-collector
  volumes:
    - ./otel-collector-config.yaml:/etc/otelcol-contrib/config.yaml
  ports:
    - "4318:4318"
  networks:
    - flare-network
```

### Collector Configuration

The following config receives OTLP signals from Flare and forwards logs to Loki and metrics to Prometheus:

```yaml
receivers:
  otlp:
    protocols:
      http:
        endpoint: 0.0.0.0:4318

exporters:
  otlp_http/loki:
    endpoint: http://loki:3100/otlp
  prometheusremotewrite:
    endpoint: http://prometheus:9090/api/v1/write
    remote_write_queue:
      enabled: true
  debug:
    verbosity: detailed

service:
  pipelines:
    logs:
      receivers: [otlp]
      exporters: [otlp_http/loki]
    metrics:
      receivers: [otlp]
      exporters: [prometheusremotewrite, debug]
```

This routes:
- **Logs** → Loki (via OTLP HTTP)
- **Metrics** → Prometheus (via Remote Write)

Traces can be added to the pipeline by configuring an exporter such as Tempo or Jaeger and adding a `traces` pipeline entry.

---

## Grafana Dashboards

Pre-built Grafana dashboards for Flare metrics and logs are currently in development.
