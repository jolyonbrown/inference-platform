# Failure Scenarios

This directory holds the evidence-heavy part of the project.

Each scenario should contain:

- a reproducible trigger
- baseline vs degraded vs recovered evidence
- a postmortem
- mitigation notes
- runbook updates

## Planned Scenarios

- `01-kv-cache-exhaustion`
- `02-kv-cache-loss-on-restart`
- `03-tail-latency-poisoning`
- `04-cold-start-under-load` optional
- `05-spot-interruption` optional
