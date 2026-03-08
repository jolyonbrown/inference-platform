# inference-platform

**A cache-aware LLM inference platform proof of concept focused on reliability, observability, and failure recovery.**

---

## Goal

Build a portfolio project that demonstrates strong hiring signal for foundational AI labs by showing that you can:

1. Serve an LLM on a single GPU with realistic operational controls
2. Measure the system with useful inference-specific metrics
3. Reproduce and diagnose failures that matter in real deployments
4. Document design tradeoffs, incident response, and mitigations clearly
5. Preserve a credible path to AWS deployment without making cloud infrastructure the core story

This is not trying to be a full commercial platform.
This is a focused, evidence-driven proof of concept for **LLM reliability engineering**.

---

## Project Thesis

Most LLM portfolio projects prove that a model can answer a prompt.
This project should prove that you understand what happens when inference systems come under pressure, lose state, or recover badly.

The centerpiece is not the model.
The centerpiece is **cache-aware operational reliability**.

That means the project should emphasize:

- KV cache pressure and exhaustion
- Loss of cache state during restart, rollout, or node loss
- Long-context traffic degrading latency for everyone else
- Readiness, admission control, and graceful recovery under load
- Clear SLOs, alerts, dashboards, and runbooks

---

## What This Project Is

A production-oriented LLM serving stack with:

- `vLLM` as the inference engine
- a thin request gateway for admission control and metrics
- Prometheus and Grafana for observability
- Kubernetes manifests for local and cloud deployment
- reproducible failure scenarios with postmortems and remediation notes
- an optional AWS EKS deployment path for credibility and extension

---

## What This Project Is Not

Be explicit about boundaries. That increases credibility.

This proof of concept does **not** attempt to solve:

- distributed multi-node inference
- distributed KV cache
- prefill/decode disaggregation
- large-scale multi-tenant fairness and quota enforcement
- cross-region failover
- production security hardening beyond sensible defaults

If asked in interview: those are valid next steps, but they are not needed to demonstrate strong reliability and platform instincts on a single-node serving system.

---

## Revised Scope

### Primary objective

Deliver a local-first system that is measurable, reproducible, and technically honest.

### Secondary objective

Include a documented and reviewable AWS deployment path that shows platform awareness, but do not let AWS work crowd out the main reliability story.

### Non-goal

Do not try to impress by sheer breadth. Impress by selecting the right failure modes and documenting them rigorously.

---

## Core Architecture

```text
Client / load generator
        |
        v
Lightweight gateway
- request validation
- prompt/token budget enforcement
- classify short vs long requests
- expose queue / rejection metrics
- optional overload shedding
        |
        v
vLLM serving pod
- model loaded on single GPU
- metrics endpoint
- health / readiness semantics
        |
        +--> Prometheus
        +--> Grafana
        +--> Alerting rules
```

### Why add a gateway?

Without a control layer, this becomes mostly a `vLLM-on-Kubernetes` repo.
A thin gateway makes it feel like a platform and gives you a place to implement:

- request admission logic
- max-input and max-output token policy
- request classification for experiments
- queue depth metrics for autoscaling or alerting
- load shedding under cache or latency pressure

Keep it thin. It should be simple enough to explain in five minutes.

---

## Repo Structure

```text
inference-platform/
├── README.md
├── Makefile
├── ARCHITECTURE.md
├── SLOS.md
├── TESTED-MODELS.md
│
├── gateway/
│   ├── app/                         # thin request gateway / proxy
│   ├── tests/
│   └── README.md
│
├── config/
│   ├── models.yaml
│   └── platform.yaml
│
├── docker/
│   ├── Dockerfile.vllm
│   └── Dockerfile.gateway
│
├── k8s/
│   ├── base/
│   │   ├── namespace.yaml
│   │   ├── gateway-deployment.yaml
│   │   ├── vllm-deployment.yaml
│   │   ├── services.yaml
│   │   ├── pdb.yaml
│   │   └── kustomization.yaml
│   ├── overlays/
│   │   ├── local/
│   │   └── aws/
│   └── monitoring/
│       ├── prometheus-config.yaml
│       ├── alerting-rules.yaml
│       └── grafana-dashboards/
│
├── loadtest/
│   ├── locustfile.py
│   ├── profiles/
│   │   ├── steady-state.yaml
│   │   ├── long-context.yaml
│   │   └── recovery.yaml
│   └── analysis.py
│
├── failures/
│   ├── README.md
│   ├── 01-kv-cache-exhaustion/
│   ├── 02-kv-cache-loss-on-restart/
│   ├── 03-tail-latency-poisoning/
│   ├── 04-cold-start-under-load/          # optional
│   └── 05-spot-interruption/              # optional AWS scenario
│
├── docs/
│   ├── RUNBOOK.md
│   ├── ADR/
│   │   ├── 001-why-vllm.md
│   │   ├── 002-why-add-a-gateway.md
│   │   ├── 003-cache-pressure-policy.md
│   │   └── 004-aws-scope.md
│   ├── INCIDENT-TEMPLATE.md
│   └── COST-ANALYSIS.md
│
├── terraform/
│   ├── main.tf
│   ├── eks.tf
│   ├── networking.tf
│   ├── iam.tf
│   └── variables.tf
│
├── scripts/
│   ├── local-up.sh
│   ├── local-down.sh
│   ├── deploy-aws.sh
│   └── smoke-test.sh
│
└── .github/
    └── workflows/
        ├── ci.yaml
        └── integration-test.yaml
```

---

## Success Criteria

This section is missing from the current plan and should be added early.

Define success in terms of measurable behavior, not just completed files.

### SLO candidates

You can tune the numbers later once you have real measurements.

- `steady-state TTFT p95` stays below a documented threshold for prompts up to a chosen size
- `end-to-end latency p95` stays below a documented threshold under baseline concurrency
- `error rate` stays below a documented threshold during steady-state load
- `recovery time after pod loss` stays below a documented threshold
- `gateway rejects or sheds traffic` before the system enters pathological overload
- `cache-pressure alerts` fire early enough to allow mitigation before hard failure

### Evidence required

For every claim, include at least one of:

- benchmark table
- dashboard screenshot with annotation
- failure reproduction script
- before/after comparison
- postmortem with timestamps and metrics

---

## The Three Mandatory Failure Scenarios

These are the heart of the project. Do these deeply rather than doing seven shallowly.

### 01. KV Cache Exhaustion

This remains the flagship scenario.

#### Why it matters

This is one of the clearest inference-specific failure modes and a strong signal that you understand LLM serving beyond normal web operations.

#### What to show

- increasing prompt/context pressure drives cache usage upward
- TTFT and queueing degrade before hard failure
- the system either rejects, stalls, or thrashes depending on configuration
- alerting and runbook guidance are tied to observed behavior

#### Questions to answer

- Which metric best reflects usable cache pressure?
- At what threshold does user-visible latency degrade materially?
- What mitigation actually helps first: lower max model length, request rejection, scale out, or more conservative admission control?
- How does `gpu_memory_utilization` or similar configuration change the result?

#### Required artifacts

- trigger script
- dashboard evidence at normal, degraded, and failing states
- postmortem
- runbook entry
- before/after mitigation comparison

### 02. KV Cache Loss on Restart / Rollout / Node Loss

This should be the second centerpiece because it demonstrates that you understand cache state as an operational dependency, not just a memory statistic.

#### Why it matters

A system can look healthy after a restart while actually being much worse for users because hot cache state is gone.
That is exactly the kind of subtle failure mode a reliability-minded reviewer will care about.

#### What to show

- steady traffic warms the system
- a pod restart, rollout, or node failure wipes useful cache state
- TTFT, throughput, and queueing worsen during cache rebuild
- readiness alone is not the same as operational recovery

#### Questions to answer

- How bad is the post-restart penalty?
- What metrics reveal cache rebuild pain most clearly?
- Can graceful draining reduce the blast radius?
- Does a warm spare or rollout strategy improve recovery materially?

#### Required artifacts

- trigger script for restart / termination / simulated node loss
- measured comparison of pre-restart and post-restart latency
- documented mitigation options
- incident-style write-up focused on hidden recovery cost

### 03. Tail-Latency Poisoning from Long-Context Traffic

This replaces the vaguer `slow poison` framing with a more precise description.

#### Why it matters

It shows that you understand fairness, latency percentiles, and the fact that one class of requests can harm everyone else without causing a clean outage.

#### What to show

- baseline short-request traffic looks healthy
- introduction of long-context requests leaves average metrics deceptively acceptable
- p95 or p99 latency diverges sharply
- request classification at the gateway makes the problem visible

#### Questions to answer

- What ratio of long to short requests causes unacceptable tail behavior?
- Which metric shows the issue earliest: queue depth, TTFT p99, throughput, or rejection rate?
- Does admission control or request class isolation help?

#### Required artifacts

- mixed-traffic profile
- percentile-focused dashboard evidence
- one mitigation experiment
- postmortem and runbook update

---

## Optional Scenarios

Only do these after the three mandatory scenarios are strong.

### 04. Cold Start Under Load

Useful if you want to show readiness semantics, model load time, and the difference between process health and service availability.

### 05. Spot Interruption on AWS

Useful for cloud credibility. Document carefully even if the demo is partial.
This is a good extension, but it should not displace the cache-state scenarios above.

---

## Build Order

### Phase 1: Walking skeleton

Goal: one end-to-end request path with basic metrics.

1. Run `vLLM` locally with a single chosen model
2. Add the gateway in front of it
3. Make one request through the gateway
4. Expose metrics from both components
5. Add one basic dashboard
6. Document local startup in the README

Deliverable: a repo that works locally and can be demonstrated quickly.

### Phase 2: Local production patterns

Goal: make local deployment operationally credible.

1. Kubernetes manifests for gateway and vLLM
2. readiness and liveness checks with correct semantics
3. resource requests and limits
4. PodDisruptionBudget
5. graceful shutdown and drain behavior
6. baseline load test profile

Deliverable: local kind deployment with reproducible steady-state measurements.

### Phase 3: Observability and SLOs

Goal: define what healthy means before breaking things.

1. Create `SLOS.md`
2. Add Prometheus alerting rules
3. Build focused dashboards
4. Establish baseline numbers for normal operation
5. Write the initial runbook

Deliverable: a measurable system with an explicit definition of failure.

### Phase 4: Mandatory failure scenarios

Goal: produce the strongest portfolio evidence.

1. KV cache exhaustion
2. KV cache loss on restart / rollout / node loss
3. Tail-latency poisoning from long-context traffic

For each scenario, produce:

- trigger
- metrics and screenshots
- postmortem
- mitigation experiment
- runbook update

Deliverable: the `failures/` directory becomes the centerpiece of the repo.

### Phase 5: AWS deployment path

Goal: show cloud competence without derailing the core project.

1. Terraform for EKS and basic networking
2. AWS Kustomize overlay
3. IAM with least privilege
4. documented GPU node approach
5. documented cost analysis
6. optional spot interruption scenario

Deliverable: a reviewable, plausible AWS path with enough detail to be credible.

### Phase 6: CI and polish

Goal: make the repo easy to review.

1. CI for linting and manifest validation
2. local integration smoke test
3. architecture diagram
4. concise top-level README
5. tested models and benchmark summary

Deliverable: a portfolio repo that another engineer can understand quickly.

---

## Experiment Matrix

The repo should include a small matrix for all benchmark and failure work.
That avoids isolated screenshots with no shared frame of reference.

Track at least:

- model name
- prompt length bucket
- output length bucket
- concurrency
- request mix
- TTFT p50/p95/p99
- end-to-end latency p50/p95/p99
- throughput in tokens/sec
- error rate
- cache pressure metric
- GPU memory usage

This becomes the backbone of:

- `TESTED-MODELS.md`
- postmortem evidence
- mitigation comparisons
- README summary tables

---

## Documentation Standards

### README

The README should be short, concrete, and evidence-led.

Recommended structure:

1. one-line description
2. architecture diagram
3. local quickstart
4. what the project proves
5. failure scenarios table
6. benchmark summary table
7. AWS deployment path
8. links to runbook, ADRs, and incident reports

### RUNBOOK

The runbook should have practical sections such as:

- TTFT degradation investigation
- cache pressure response
- restart / rollout recovery checks
- long-context traffic response
- cold start diagnosis
- how to add or swap a model

### ADRs

Keep these short and defensible.
Suggested ADRs:

- why `vLLM`
- why add a gateway
- why focus on cache-aware controls
- why AWS is secondary in the first version

### Incident reports

The incident format is good, but tone it down and keep it factual.
Do not say it matches any specific company unless you can support that claim.

Use this structure:

```markdown
# Failure Scenario: KV Cache Exhaustion

## Summary

## Impact

## Timeline

## Detection

## Root Cause

## Mitigation

## What Improved After the Fix

## Follow-up Actions

## How to Reproduce
```

---

## AWS Positioning

AWS should remain in scope, but its role should be explicit.

### Why include it

- it shows you can think beyond localhost
- it is relevant to how many large AI systems are deployed
- it gives you room to discuss GPU scheduling, IAM, networking, and cost

### Why keep it secondary

A reviewer will be more impressed by a local system with excellent failure evidence than by a half-finished EKS setup with no real operating story.

### Standard to meet

The AWS path should be:

- technically plausible
- documented clearly
- cost-conscious
- narrow enough to maintain

If you do deploy it, only include claims you can support with evidence.

---

## Risks and How to Avoid Them

### Risk 1: Overbuilding infrastructure before proving the core story

Mitigation: do the local failure scenarios first.

### Risk 2: Treating screenshots as evidence without measurements

Mitigation: use the experiment matrix and baseline benchmarks.

### Risk 3: Spending too much time on generic Kubernetes work

Mitigation: keep pulling the narrative back to inference-specific reliability.

### Risk 4: Making claims about metrics or behavior before validating them

Mitigation: verify actual `vLLM` metrics and observed behavior before baking names and thresholds into docs.

### Risk 5: Building too many scenarios shallowly

Mitigation: complete three strong scenarios before adding optional ones.

---

## Interview Story

This project should let you tell a clear story:

> I built a production-oriented LLM inference proof of concept around the operational problems that actually matter: cache pressure, loss of cache state, tail-latency degradation, observability, and recovery. I defined SLOs, reproduced failures, measured the impact, and documented mitigations. I also preserved a credible AWS deployment path, but kept the main focus on reliability evidence rather than cloud sprawl.

That is a much stronger signal than “I deployed a model on Kubernetes.”

---

## Recommended First Milestone

If you want the strongest early signal, the first serious milestone should be:

1. local vLLM deployment works through the gateway
2. Prometheus and Grafana are wired up
3. one baseline load profile is captured
4. `SLOS.md` exists
5. `failures/01-kv-cache-exhaustion/` is fully documented

At that point, the repo is already worth reviewing.
The remaining work then deepens the story instead of trying to create it.
