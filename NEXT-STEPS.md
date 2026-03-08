# Next Steps

## Objective

Move the repository from a strong scaffold and documentation set to a runnable local inference platform with measurable failure scenarios and a credible AWS deployment path.

## Phase 1: Local Walking Skeleton

1. Choose the initial local development model.
2. Implement the first version of the gateway.
3. Run `vLLM` locally behind the gateway.
4. Add a basic smoke test that sends one request through the full path.
5. Make `make local-up` and `make smoke-test` actually work.

## Phase 2: Container and Kubernetes Baseline

1. Replace placeholder Dockerfiles with working images.
2. Turn the base Kubernetes manifests into a working local deployment.
3. Add correct liveness and readiness checks.
4. Add graceful shutdown behavior.
5. Verify the stack can run locally with kind.

## Phase 3: Observability Foundation

1. Wire Prometheus to scrape gateway and `vLLM` metrics.
2. Define and validate the useful inference metrics exposed by the stack.
3. Build the first Grafana dashboard for latency, request rate, and cache pressure.
4. Replace placeholder alerting rules with real alert definitions.
5. Update `SLOS.md` with measured baseline numbers.

## Phase 4: KV Cache Exhaustion Scenario

1. Implement `failures/01-kv-cache-exhaustion/trigger.sh`.
2. Capture baseline, degraded, and failure-state evidence.
3. Validate which metric best signals dangerous cache pressure.
4. Test at least one mitigation through the gateway or serving configuration.
5. Complete the postmortem and runbook entry.

## Phase 5: Cache Loss Recovery Scenario

1. Flesh out `failures/02-kv-cache-loss-on-restart/`.
2. Measure the effect of restart or rollout on latency and recovery.
3. Compare readiness returning green with actual operational recovery.
4. Document mitigation options such as draining or rollout strategy.
5. Add incident notes and evidence.

## Phase 6: Tail-Latency Scenario

1. Flesh out `failures/03-tail-latency-poisoning/`.
2. Create a mixed traffic profile with short and long-context requests.
3. Measure p50 versus p95 and p99 behavior.
4. Test a mitigation such as request classification or admission control.
5. Document the resulting tradeoffs.

## Phase 7: AWS Path

1. Replace Terraform placeholders with a minimal EKS path.
2. Define the AWS overlay for Kubernetes manifests.
3. Document GPU node assumptions and IAM requirements.
4. Produce a first-pass cost analysis.
5. Decide whether to implement the optional spot interruption scenario now or later.

## Phase 8: Repository Polish

1. Tighten the README once the local stack actually runs.
2. Replace placeholder ADRs with short, defensible decisions.
3. Add benchmark data to `TESTED-MODELS.md`.
4. Implement CI checks that validate the repo structure and manifests.
5. Review the clone-to-first-result experience end to end.

## Immediate Next Task

The most sensible next implementation step is:

1. choose the local model
2. implement the gateway
3. make the local request path work end to end

Everything else depends on that.
