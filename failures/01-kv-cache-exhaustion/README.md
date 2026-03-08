# Failure Scenario: KV Cache Exhaustion

## Why This Scenario Exists

KV cache pressure is one of the clearest inference-specific reliability problems in LLM serving.
This scenario is intended to show how cache pressure becomes visible in metrics, how it impacts user-facing latency, and how an operator should respond before the system reaches a hard failure mode.

## Objective

Demonstrate that increasing prompt length and concurrency can drive the serving system into a cache-pressure regime where:

- `TTFT` degrades materially
- queueing or rejection behavior changes
- GPU memory pressure becomes operationally relevant
- alerts and runbook guidance can be tied to observable thresholds

## Hypothesis

As cache usage approaches saturation, user-visible latency will degrade before the system fully fails.
The best operating response may be to reject or constrain traffic before the system starts thrashing.

## Questions To Answer

1. Which metric best signals dangerous cache pressure?
2. At what threshold does latency degrade enough to violate the SLO?
3. What behavior does the stack show at high pressure: queueing, rejection, or instability?
4. Which mitigation is most effective first: tighter token limits, lower max model length, scale-out, or gateway admission control?

## Planned Trigger

Use a scripted traffic profile that gradually increases context length and concurrency until the system enters a clearly degraded state.

Proposed shape:

1. warm the system with normal traffic
2. introduce longer prompts in controlled increments
3. increase concurrency
4. hold the degraded state long enough to capture metrics and screenshots

## Planned Evidence

- dashboard screenshot at baseline
- dashboard screenshot during degradation
- dashboard screenshot near or at failure
- benchmark notes with timestamps
- before/after mitigation comparison

## Detection

Planned detection signals:

- cache pressure metric from `vLLM`
- `TTFT p95` and `p99`
- queue depth at the gateway
- rejection rate
- GPU memory pressure metrics

Metric names should be validated against actual implementation before finalizing alert rules.

## Candidate Mitigations

- lower maximum accepted prompt size
- lower maximum model length
- introduce earlier admission control at the gateway
- shed or reject long-context traffic under pressure
- tune serving configuration conservatively

## Deliverables

This scenario is not complete until it includes:

- `trigger.sh`
- `postmortem.md`
- annotated screenshots
- concrete alert thresholds
- a runbook section describing immediate and medium-term actions
