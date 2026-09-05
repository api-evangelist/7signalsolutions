---
name: 7signal-run-on-demand-test
description: Trigger an on-demand network test on a Sapphire Eye sensor and poll for its result, including retrieving a packet capture file.
api: 7SIGNAL Platform API (Gateway v2)
base_url: https://api-v2.7signal.com
operations:
  - on-demand-tests-speedtest
  - on-demand-tests-speedtest-status
  - on-demand-tests-pcap
  - on-demand-tests-pcap-status
  - on-demand-tests-pcap-file
  - on-demand-tests-active-tests-list
generated: '2026-09-05'
method: generated
source: openapi/7signalsolutions-openapi.json + https://github.com/7Signal/API-Examples/blob/develop/docs/13-on-demand-tests.md
---

# Run an on-demand test on a sensor

On-demand tests are the largest family on the gateway (31 operations) and all of them follow the same
trigger-then-poll shape. Speedtest and packet capture are shown here; ping, ping-gateway, traceroute,
iperf3, MOS, http/tcp/udp upload and download, and web-download work identically.

## Steps

1. **Trigger.** `POST /on-demand-tests/sensors/{sensorId}/speedtest` (`on-demand-tests-speedtest`).
   The response carries the `testId`.
2. **Poll.** `GET /on-demand-tests/sensors/{sensorId}/speedtest/{testId}`
   (`on-demand-tests-speedtest-status`) until the test reports a terminal status. Back off between
   polls; every poll spends a rate-limit token.
3. **For a packet capture**, the shape is the same — `POST .../packet-capture`
   (`on-demand-tests-pcap`), then `GET .../packet-capture/{testId}` (`on-demand-tests-pcap-status`) —
   but there is a third step: `GET .../packet-capture/{testId}/download` (`on-demand-tests-pcap-file`)
   returns the capture as `application/vnd.tcpdump.pcap`. Handle it as **binary**, not JSON.
4. **Before triggering anything**, `GET /on-demand-tests/sensors/active-tests`
   (`on-demand-tests-active-tests-list`) to see what is already running on the fleet. A sensor that is
   busy testing is not passively monitoring.

## Rules

- **These calls are not idempotent.** 7SIGNAL publishes no `Idempotency-Key` header and no dedupe
  window. A retried trigger starts a *second* test on real hardware. If a trigger times out, poll
  `active-tests` before firing again.
- **There is no dry-run mode.** Nothing on this API lets you rehearse a write.
- **There is no cancel operation** for a running test. Once triggered it runs. Choose parameters
  carefully; this is the least reversible surface on the API.
- A test takes real airtime on a real sensor. Do not loop these on a schedule without a reason.
