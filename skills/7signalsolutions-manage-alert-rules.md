---
name: 7signal-manage-alert-rules
description: Create, tune, disable and audit 7SIGNAL alert rules and the incidents they raise, including email, webhook and ServiceNow delivery.
api: 7SIGNAL Platform API (Gateway v2)
base_url: https://api-v2.7signal.com
operations:
  - list-alert-rules
  - create-alert-rule
  - get-alert-rule
  - update-alert-rule
  - patch-alert-rule-enabled
  - delete-alert-rule
  - alert-rules-summary
  - list-alerting-incidents
  - list-alerting-incidents-by-rule
  - alerting-incidents-summary
  - get-alerting-incident
  - resolve-alerting-incident
generated: '2026-09-05'
method: generated
source: openapi/7signalsolutions-openapi.json + https://github.com/7Signal/API-Examples/blob/develop/docs/16-alerting.md
---

# Manage alerting and incidents

Two related resources: **alert rules** (`/alerting/alert-rules`) are the configuration, **incidents**
(`/alerting/incidents`) are the history.

## Create a rule

`POST /alerting/alert-rules` (`create-alert-rule`). The rule names a metric, a `dimensionSet`, an
`aggregationFunction`, a `thresholdValue` and `thresholdOperator`, how long the breach must persist
(`pendingPeriodSeconds`), and where notifications go.

**`dimensionSet` is the main lever on volume.** A rule keyed on `device_id` raises one incident *per
device*; a rule keyed on `network` raises one *per network*. Get this wrong and you will page someone
a thousand times.

Defaults when omitted: `pendingPeriodSeconds` `300` (minimum `1`), `missingDataPolicy` `ignore`,
`enabled` `true`.

`missingDataPolicy` decides what a silent metric means: `ignore` holds state (safe default), `good`
treats the gap as compliant, `bad` treats it as a breach — and `bad` will alert on every device that
legitimately goes offline.

## Notifications

`notificationConfig.deliveries[]` is a list discriminated by `kind`:

- `email` — needs `address`.
- `webhook` — needs `url` and `auth`. `format` is `generic` (default) or `slack`. For an authenticated
  webhook, pass credentials by **AWS Secrets Manager ARN**
  (`{"type":"BASIC","username":"svc","passwordSecretArn":"arn:aws:secretsmanager:..."}` or
  `{"type":"TOKEN","tokenSecretArn":"..."}`). The secret value is never stored on, or returned by, the
  rule.
- `servicenow` — needs `mdsConfigId` and `organizationName`, and the ServiceNow integration must
  already exist.

## Edit safely

- **`PUT` is a full replace** (`update-alert-rule`). Omitted optional fields fall back to defaults,
  which reads to a user as their settings being wiped. `GET` the rule, change what you mean to change,
  `PUT` the complete object back.
- **To turn a rule off, use `PATCH /alerting/alert-rules/{id}/enabled`** (`patch-alert-rule-enabled`),
  not `DELETE`. The toggle keeps the configuration and the incident history and is fully reversible;
  `delete-alert-rule` is permanent.
- Enum values are **lowercase**. `"avg"`, not `"AVG"` or `"Avg"` — the wrong casing returns `400`.
  Same for `missingDataPolicy` (`ignore`/`good`/`bad`) and operators (`<`).

## Work the incidents

- `GET /alerting/incidents` (`list-alerting-incidents`) filtered by status, metric, rule or time.
- `GET /alerting/incidents/by-rule` (`list-alerting-incidents-by-rule`) counts incidents per rule —
  this is how you find your noisiest rule and fix its `dimensionSet` or `pendingPeriodSeconds`.
- `POST /alerting/incidents/{id}/resolve` (`resolve-alerting-incident`) resolves one manually.
  **This is terminal** — a second call returns `409 Conflict`, and there is no un-resolve.
- Each incident holds a snapshot of the rule as it was when the incident was raised, so history stays
  readable after you edit or delete the rule.

## Troubleshooting

A breaching metric with no incident is almost always `pendingPeriodSeconds`: the condition must hold
*continuously* for that long. With the default of 300, a two-minute dip never alerts.
