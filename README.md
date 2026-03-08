# inference-platform

**A cache-aware LLM inference platform proof of concept focused on reliability, observability, and failure recovery.**

## Using this to see how inference platforms work and what failures they might encounter

This repo is a production-oriented proof of concept for operating LLM inference under pressure.

It is designed to demonstrate:

- inference-specific observability rather than generic service monitoring
- controlled failure injection and incident-style analysis
- cache-aware operational thinking, especially around KV cache pressure and loss
- platform instincts such as admission control, graceful recovery, and measurable SLOs
- a deployment story that spans local development and AWS-hosted operation

The point is not just to serve a model.
The point is to show what happens when the serving system degrades, loses state, and recovers.

## Architecture

```text
Client / load generator
        |
        v
Lightweight gateway
- request validation
- token budget enforcement
- request classification
- queue and rejection metrics
- optional load shedding
        |
        v
vLLM serving pod
- single-GPU model serving
- inference metrics
- readiness and shutdown behavior
        |
        +--> Prometheus
        +--> Grafana
        +--> Alerting rules
```

## Quickstart

Local implementation is not complete yet, but the intended workflow is:

```bash
make local-up
make smoke-test
```

Initial development target:

- single GPU
- `vLLM` inference server
- thin gateway in front of `vLLM`
- Prometheus and Grafana for metrics and dashboards
- kind for local Kubernetes testing

## Deployment Targets

This project is intended to support two serious operating environments:

- local development and failure reproduction on a single machine
- AWS deployment for a more production-like infrastructure path

Local work is where failure scenarios, metrics, and recovery behavior are developed quickly.
AWS is where the same design is exercised against cloud concerns such as GPU scheduling, IAM, networking, and interruption handling.

## Local Requirements

The preferred local environment is a machine with a supported GPU and enough memory headroom to run a small model plus observability tooling.

### Recommended local setup

- Linux with an NVIDIA GPU is the clearest primary path
- enough VRAM to run a small development model and reproduce cache-pressure behavior
- Docker and kind for container and Kubernetes testing
- enough system RAM and disk to hold model weights, containers, and monitoring components

### Possible but lower-confidence paths

- Apple Silicon is possible, but current `vLLM` support is experimental on macOS CPU
- GPU-accelerated Apple Silicon support depends on the separate community `vllm-metal` plugin rather than the mainline path
- CPU-only local execution is useful for scaffolding and basic integration work, but it is not the right environment for the main cache and latency experiments

### Practical expectation

If the goal is to reproduce the main reliability scenarios with realistic timings, a GPU-backed local environment is strongly preferred.
If the goal is only to develop the control plane, dashboards, or basic integration flows, CPU-only and Apple Silicon environments can still be useful.

## Core Reliability Scenarios

The `failures/` directory is the technical centerpiece of the project.

| Scenario | What It Demonstrates | Status |
|---|---|---|
| [01 KV Cache Exhaustion](failures/01-kv-cache-exhaustion/) | Cache pressure, latency degradation, and mitigation strategy | Drafted |
| [02 KV Cache Loss on Restart](failures/02-kv-cache-loss-on-restart/) | Hidden recovery cost after restart, rollout, or node loss | Scaffolded |
| [03 Tail-Latency Poisoning](failures/03-tail-latency-poisoning/) | How long-context traffic degrades p95/p99 for everyone | Scaffolded |
| [04 Cold Start Under Load](failures/04-cold-start-under-load/) | Difference between process health and service readiness | Optional |
| [05 Spot Interruption](failures/05-spot-interruption/) | AWS-specific recovery and draining behavior | Optional |

## Success Criteria

The project uses measurable goals rather than purely descriptive docs. See [SLOS.md](SLOS.md).

The current focus is to prove that the system can:

- maintain documented latency targets under baseline load
- detect cache pressure before hard failure
- recover predictably from restart-related cache loss
- expose evidence through dashboards, metrics, and postmortems

## AWS Path

AWS is a first-class target for this project, not just a documentation exercise.
The aim is to keep the local path fast for iteration while also designing for a realistic cloud deployment model.

Planned AWS elements:

- EKS deployment path
- GPU node configuration
- IAM and networking basics
- cost analysis
- optional spot interruption scenario

## Key Docs

- [Project plan](llm-serving-platform-plan.md)
- [SLOs](SLOS.md)
- [Architecture notes](ARCHITECTURE.md)
- [Runbook](docs/RUNBOOK.md)
- [Failure scenarios](failures/README.md)
- [Cost analysis](docs/COST-ANALYSIS.md)

## Current State

This repository is in the buildout stage.
The initial emphasis is:

1. local walking skeleton
2. observability and baseline measurements
3. KV cache exhaustion scenario
4. cache-loss recovery scenario
5. AWS deployment path
