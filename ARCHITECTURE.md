# Architecture

## Intent

This project wraps a `vLLM` serving component in a minimal control and observability layer so that inference-specific operational behavior can be measured and managed.

## Components

### Gateway

The gateway is intentionally thin.
Its job is to:

- validate requests
- enforce input and output token limits
- classify requests for experiments
- emit queue and rejection metrics
- provide a place for overload shedding or admission control

### vLLM

`vLLM` is the inference engine.
It is treated as the stateful core of the system because serving behavior depends heavily on GPU memory and KV cache state.

### Observability

Prometheus and Grafana provide:

- request and latency metrics
- cache-pressure visibility
- queue depth and rejection behavior
- recovery timelines after restart or disruption

### Kubernetes

Kubernetes is used to exercise realistic lifecycle events:

- readiness vs actual recovery
- pod termination and draining
- rollout behavior
- disruption handling

## Design Principle

The project should feel like a platform because it makes policy decisions around an inference engine, not because it includes every possible infrastructure component.
