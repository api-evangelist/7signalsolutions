---
name: 7signal-triage-agent-experience
description: Investigate a degraded endpoint or location — list agents, read impact and experience scores, check platform-detected incidents, inspect RF scans, and request an Eyeris AI analysis.
api: 7SIGNAL Platform API (Gateway v2)
base_url: https://api-v2.7signal.com
operations:
  - eyes-agents
  - eyes-agent
  - agent-locations-get
  - impact-agents
  - impact-agents-summary
  - agent-incidents
  - agent-incident
  - scans-agents
  - eyeris-client-analysis-post-y7z8
  - eyeris-client-analysis-with-id-get-a9b0
generated: '2026-09-05'
method: generated
source: openapi/7signalsolutions-openapi.json + https://github.com/7Signal/API-Examples/blob/develop/docs/14-impact.md
---

# Triage a Wi-Fi experience complaint

The read-only investigation path, from "the Wi-Fi is bad" to a grounded answer.

## Steps

1. **Find the endpoint.** `GET /eyes/agents` (`eyes-agents`) lists Mobile Eye agents; filter by
   `lastLocationId`, or by `scoreSortStart`/`scoreSortEnd` to pull the worst-scoring devices first.
   `GET /eyes/agents/search` searches. `GET /eyes/agents/{agentId}` (`eyes-agent`) fetches one.
2. **Place it.** `GET /locations/agents` (`agent-locations-get`) resolves the location hierarchy the
   agent reports into.
3. **Quantify the damage.** `GET /impact/agents/summary` (`impact-agents-summary`) for the roll-up and
   `GET /impact/agents` (`impact-agents`) for the detail — experience score, connectivity, coverage,
   congestion, interference and roaming. `GET /impact/locations` scopes it to a site.
4. **Check whether the platform already saw it.** `GET /incidents/agents` (`agent-incidents`) lists
   platform-detected incidents where a *share of the agent population* degraded together —
   distinguishing "this laptop is broken" from "this floor is broken". `GET /incidents/agents/{incidentId}`
   (`agent-incident`) opens one.
5. **Look at the RF.** `GET /scans/agents` (`scans-agents`) returns what the agent actually heard:
   detected APs, signal strength, channel, noise floor.
6. **Ask Eyeris.** `POST /eyeris/agents/client-analysis` (`eyeris-client-analysis-post-y7z8`) requests
   an AI analysis of the agent's experience and returns a `requestId`; poll
   `GET /eyeris/agents/client-analysis/{requestId}` (`eyeris-client-analysis-with-id-get-a9b0`) for the
   result, or subscribe to `GET /eyeris/analysis/stream` for Server-Sent Events. The response carries
   `Eyeris-Request-Id`, `Eyeris-Request-Queue-Id` and `Eyeris-Response-Id` — keep them for support.

## Rules

- `/incidents/agents` and `/alerting/incidents` are **unrelated resources**. The first is
  platform-detected; the second is raised by an alert rule you configured. 7SIGNAL calls this out
  explicitly because the names collide.
- Every step here is a `GET` except the Eyeris request, so this whole skill is safe to run
  speculatively — the one thing on this API that is.
- Ids are bare UUIDs with no type prefix, and they are organization-scoped. A wrong-scope id returns
  `404`, not `403`, so a 404 does not prove the resource does not exist.
- Do not reach for `/kpis/*` or `/topologies/*` — both families are deprecated. Use `/time-series/*`,
  `/locations/*`, `/service-areas/*` and `/access-points/*`.
