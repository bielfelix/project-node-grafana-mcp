# Telemetry Investigation Report: /students/db-leaky-connections (last ~15 minutes)

## Summary
- Prometheus: no per-endpoint HTTP 5xx metrics found for the service handling `/students/db-leaky-connections`. General Go/DB metrics present but not correlated to the failing service.
- Loki: no logs referencing `/students/db-leaky-connections` found; no error-level logs or stack traces available.
- Tempo: multiple traces from service `alumnus_app_5b3d` show `GET /students/db-leaky-connections` returning HTTP 500; span durations ~1000 ms. Traces confirm repeated failures but do not include exception stack traces or file/line attributes.

## Prometheus queries executed
- Datasource: `prometheus` (uid: `prometheus`)
- Example queries tried:
  - `sum(increase(http_requests_total{path="/students/db-leaky-connections",status=~"5.."}[1m]))`
  - `sum(increase(http_server_requests_seconds_count{uri="/students/db-leaky-connections",status=~"5.."}[1m]))`
  - `go_sql_stats_connections_in_use` (returned metrics for other services)
  - `go_sql_stats_connections_open` (shows 2 open connections for grafana/unified_storage in this Prometheus)
  - `histogram_quantile(0.95, sum(rate(db_client_operation_duration_seconds_bucket[5m])) by (le))` (returned NaN / no counts)

## Loki queries executed
- Datasource: `loki` (uid: `loki`)
- Example queries tried:
  - `{job=~".+"} |= "/students/db-leaky-connections" |= "error"`
  - `{job=~".+"} |= "/students/db-leaky-connections"`
  - `{job=~".+"} |= "db-leaky-connections"`
- Result: no matching logs found for the endpoint.

## Tempo queries executed and results
- Datasource: `tempo` (uid: `tempo`)
- Example TraceQL queries and results:
  - `{ span:name =~ ".*db-leaky-connections.*" && span.http.status_code >= 500 }` → returned multiple traces
  - Extracted sample traceIDs and brief details (all from `alumnus_app_5b3d`):
    - `1c09c45043dfe92bf170775a343c630` — span `GET /students/db-leaky-connections` — duration ~1014 ms — status 500
    - `5f4ae74a0db985ffa5c38becd52672f` — duration ~1012 ms — status 500
    - `a906dda1b56f6046b88b4f46d31383b` — duration ~1004 ms — status 500
    - `42bcb22dbd1a10da2bedafa145750da` — duration ~1000 ms — status 500
    - (additional traceIDs found in results; all show similar pattern)
  - Attempts to find `event.name = "exception"` or `exception.*` attributes returned no traces with exception events.

## Correlation and analysis
- Tempo traces confirm repeated 500 responses for the route in `alumnus_app_5b3d` with consistent ~1s durations.
- Lack of logs in Loki prevents extracting the exception message or stack trace.
- Prometheus lacks per-endpoint metrics, so exact failure counts/rates and response-time distributions for failures cannot be derived from metrics alone.

## Root cause and file/line identification
- Not possible with current telemetry: no stack traces or exception attributes in traces, and no logs. The exact file and line number cannot be identified.

## Recommendations and action items
1. Enable or correct log forwarding for `alumnus_app_5b3d` so error logs (with full stack traces) are shipped to Loki with a service/job label.
2. Instrument the application to export per-endpoint HTTP metrics: request count labeled by `path` and `status`, and request duration histograms.
3. Enrich trace spans with exception events and attributes (`exception.type`, `exception.message`, `exception.stacktrace`) so Tempo contains stack traces and file/line information.
4. After logs/exception attributes are available, re-run the investigation; I can then extract the exact file and line number causing the 500.

## Artifacts
- Prometheus and Loki queries attempted are described above.
- Tempo traceIDs (sample):
  - `1c09c45043dfe92bf170775a343c630`
  - `5f4ae74a0db985ffa5c38becd52672f`
  - `a906dda1b56f6046b88b4f46d31383b`
  - `42bcb22dbd1a10da2bedafa145750da`
  - `3ca9fd91ea3c03ea83b316af604ac81`

---

*Report generated on 2026-06-16.*
