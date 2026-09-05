---
name: 7signal-query-wifi-experience
description: Pull Wi-Fi experience time series for Mobile Eye agents or Sapphire Eye sensors, grouped by location, band, channel or access point.
api: 7SIGNAL Platform API (Gateway v2)
base_url: https://api-v2.7signal.com
operations:
  - time-series-agents-metric-types
  - time-series-agents-numeric-group-by
  - time-series-sensors-metric-types
  - time-series-sensors-overall-numeric
  - agent-locations-get
generated: '2026-09-05'
method: generated
source: openapi/7signalsolutions-openapi.json + https://github.com/7Signal/API-Examples/blob/develop/docs/09-time-series.md
---

# Query Wi-Fi experience over time

Time Series is the metrics surface that replaced the deprecated `/kpis/*` family. Use it, not `/kpis`.

## Choose the population first

7SIGNAL measures from two vantage points and they are separate object graphs:

- **Agents** — Mobile Eye software on real endpoints. `/time-series/agents/*`.
- **Sensors** — Sapphire Eye hardware. `/time-series/sensors/*`.

## Steps

1. Discover what you can ask for: `GET /time-series/agents/metric-types`
   (`time-series-agents-metric-types`) or `GET /time-series/sensors/metric-types`
   (`time-series-sensors-metric-types`). These return the metric keys and KPI codes, with descriptions.
   Never guess a metric name.
2. Resolve the scope you want to group by — for locations, `GET /locations/agents`
   (`agent-locations-get`).
3. Pull the series: `GET /time-series/agents/numeric/{groupByDimension}`
   (`time-series-agents-numeric-group-by`), or `GET /time-series/sensors/numeric`
   (`time-series-sensors-overall-numeric`) for an organization-wide roll-up.

## Parameter rules

- `from` and `to` are **epoch milliseconds**, not seconds and not ISO strings.
- `timeBucket` is one of `1_MIN`, `10_MIN`, `1_HOUR`, `1_DAY`, `1_MONTH`.
- `aggregateFunctions` is **required** — pick from `SUM`, `AVG`, `MIN`, `MAX`, `COUNT`, `PCTL`. Check
  the spec for which are supported on the endpoint you are calling.
- Send **multiple metric codes in one request**. That is the documented way to cut request volume, and
  it matters here because pagination works per metric.

## Reading the response

```
{ "pagination": {...}, "results": [ { <applied filter dimensions>, "metricAggregates": [
  { "metric": "...", "kpiCode": "...", "avg": ..., "threshold": ...,
    "timeSeries": [ { "ts": 1234567890000, "avg": 0.95 } ] } ] } ] }
```

- Aggregates at the `metricAggregates` level summarize the **whole** range; the per-bucket values live
  inside `timeSeries`.
- **A single series is hard-capped at 120 data points.** If you need finer resolution, shorten the range
  or widen `timeBucket` — do not try to page through it, because pagination applies to metrics, not
  points. Ten metrics at `perPage=5` gives you two pages.

## Errors

`400` on a bad enum or a missing `aggregateFunctions`. `429` when the token bucket empties — back off
and watch `ratelimit-remaining`. See `errors/7signalsolutions-problem-types.yml`.
