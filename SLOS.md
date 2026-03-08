# SLOs

## Purpose

These SLOs define what healthy behavior looks like for this proof of concept.
The exact thresholds should be updated once baseline measurements exist.

## Initial SLO Set

### SLO 1: Steady-State TTFT

For baseline traffic with prompts at or below the documented target size, `TTFT p95` should remain below the agreed threshold.

Initial placeholder target:

- `TTFT p95 < 1500ms`

### SLO 2: End-to-End Latency

For the baseline load profile, `end-to-end latency p95` should remain below the agreed threshold.

Initial placeholder target:

- `latency p95 < 5000ms`

### SLO 3: Error Rate

Under steady-state load, request failures should remain below the agreed threshold.

Initial placeholder target:

- `error rate < 1%`

### SLO 4: Cache Pressure Detection

The system should alert before KV cache pressure becomes a hard failure.

Initial success condition:

- alert fires before cache saturation or request thrashing occurs
- runbook identifies at least one safe mitigation path

### SLO 5: Recovery After Restart or Pod Loss

After a restart or pod loss event, the system should return to an acceptable state within the agreed threshold.

Initial placeholder target:

- `operational recovery < 10 minutes`

Operational recovery means more than readiness returning green.
It means latency and error behavior have returned to an acceptable range.

### SLO 6: Overload Handling

The gateway should reject or shed traffic before the system enters pathological overload.

Initial success condition:

- controlled rejection is preferred over unbounded queue growth and cascading latency collapse

## Required Evidence

Each SLO should eventually be backed by one or more of:

- benchmark tables
- dashboard screenshots with timestamps
- load test output
- incident reports
- before and after mitigation comparisons

## Review Notes

These are intentionally conservative placeholders.
They exist to force the project into measurable claims early.
