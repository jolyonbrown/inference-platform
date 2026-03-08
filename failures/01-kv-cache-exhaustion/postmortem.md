# Failure Scenario: KV Cache Exhaustion

## Summary

Placeholder for incident-style write-up once the scenario has been reproduced.

## Impact

Describe user-facing latency, error behavior, and request classes affected.

## Timeline

- T+0:00 baseline traffic starts
- T+X:XX long-context pressure begins
- T+X:XX cache pressure alert fires
- T+X:XX SLO violation observed
- T+X:XX mitigation applied
- T+X:XX system returns to acceptable behavior

## Detection

Document the earliest useful metrics and alerts.

## Root Cause

Explain how request mix, context length, and serving configuration drove cache pressure.

## Mitigation

List immediate, short-term, and longer-term actions.

## What Improved After the Fix

Compare before and after measurements.

## Follow-up Actions

List specific engineering changes worth making.

## How to Reproduce

```bash
./failures/01-kv-cache-exhaustion/trigger.sh
```
