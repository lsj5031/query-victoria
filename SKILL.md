---
name: query-victoria
description: Queries VictoriaLogs using LogsQL via HTTP API endpoints for logs, stats, facets, and metadata. Use when working with VictoriaLogs queries, building log analysis tools, or accessing VictoriaLogs data programmatically.
---

# Querying VictoriaLogs HTTP API

Query VictoriaLogs logs and metrics using the HTTP API with LogsQL syntax.

## Environment Variables

Set these before querying:

```bash
export VICTORIALOGS_URL="http://your-victorialogs-host:9428"
export VICTORIALOGS_USERNAME="your-username"
export VICTORIALOGS_PASSWORD="your-password"
```

## Core Endpoints

### Query Logs
`/select/logsql/query` - Returns matching log entries as JSON lines stream.

```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/query" -d 'query=error' -d 'limit=10'
```

**Example Response:**
```json
{"_time":"2024-01-15T10:30:00Z","_msg":"Connection error to database","level":"error","host":"web-01"}
{"_time":"2024-01-15T10:29:55Z","_msg":"Timeout error on request","level":"error","host":"web-02"}
```

Parameters: `query` (required), `limit`, `offset`, `start`/`end`, `timeout`

### Hits Stats
`/select/logsql/hits` - Returns count of matching logs grouped by time buckets.

```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/hits" -d 'query=error' -d 'start=3h' -d 'step=1h'
```

**Example Response:**
```json
{"hits":[{"fields":{},"timestamps":["2024-01-15T08:00:00Z","2024-01-15T09:00:00Z","2024-01-15T10:00:00Z"],"values":[42,87,23],"total":152}]}
```

Parameters: `query`, `start`/`end`, `step`, `field` (group by), `fields_limit`

### Facets
`/select/logsql/facets` - Returns most frequent field values in matching logs.

```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/facets" -d 'query=_time:1h error' -d 'limit=3'
```

**Example Response:**
```json
{"facets":[{"field_name":"level","values":[{"field_value":"error","hits":150},{"field_value":"warn","hits":42}]},{"field_name":"host","values":[{"field_value":"web-01","hits":89},{"field_value":"web-02","hits":61}]}]}
```

Parameters: `query`, `start`/`end`, `limit`, `max_values_per_field`

### Log Stats (Point-in-Time)
`/select/logsql/stats_query` - Returns log stats at a specific timestamp.

```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/stats_query" -d 'query=_time:1d | stats by (level) count(*)'
```

Parameters: `query` (must contain `stats` pipe), `time`

### Log Range Stats
`/select/logsql/stats_query_range` - Returns log stats over time range with step buckets.

```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/stats_query_range" \
  -d 'query=* | stats by (level) count(*)' -d 'start=2024-01-01Z' -d 'end=2024-01-02Z' -d 'step=6h'
```

Parameters: `query` (must contain `stats` pipe), `start`/`end`, `step`

### Live Tailing
`/select/logsql/tail` - Returns live tailing results like `tail -f`.

```bash
curl -N -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/tail" -d 'query=error' -d 'start_offset=1h'
```

Parameters: `query` (no `stats`/`sort`/`limit`), `start_offset`, `refresh_interval`

## Metadata Endpoints

| Endpoint | Purpose | Key Params |
|----------|---------|------------|
| `/select/logsql/stream_ids` | Get stream IDs with hit counts | `query`, `start` |
| `/select/logsql/streams` | Get stream label sets | `query`, `start` |
| `/select/logsql/stream_field_names` | Get stream field names | `query`, `start` |
| `/select/logsql/stream_field_values` | Get values for stream field | `query`, `field` |
| `/select/logsql/field_names` | Get all field names | `query`, `start` |
| `/select/logsql/field_values` | Get values for a field | `query`, `field` |

Example:
```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/field_values" -d 'query=error' -d 'field=host' -d 'start=5m'
```

## Multi-Tenant Queries

Specify tenant via headers:

```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/query" \
  -H 'AccountID: 12' -H 'ProjectID: 34' -d 'query=error'
```

## Common Patterns

**Count logs with 'error' in last hour:**
```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/query" -d 'query=_time:1h error | stats count() total'
```

**Find top 5 error types:**
```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/facets" -d 'query=error' -d 'limit=5'
```

**Parse JSON and filter with jq:**
```bash
curl -u "$VICTORIALOGS_USERNAME:$VICTORIALOGS_PASSWORD" \
  "$VICTORIALOGS_URL/select/logsql/query" -d 'query=error' | jq -r '._msg'
```

## Notes

- Responses stream as JSON lines (unsorted by default)
- Use `limit` to get most recent logs
- Close connection to cancel query safely
- All query args must be percent-encoded for curl
