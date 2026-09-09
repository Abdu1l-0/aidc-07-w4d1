# Resource policy

Guarantee the serving endpoint, allow the dashboard to use spare capacity, and throttle the background batch workload first.

## Serving

```yaml
resources:
  requests:
    cpu: "2"
    memory: 2Gi
  limits:
    cpu: "2"
    memory: 2Gi
```

QoS class: Guaranteed.

## Batch

```yaml
resources:
  requests:
    cpu: 100m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

QoS class: Burstable. The 500m CPU limit makes batch the first workload to throttle.

## Dashboard

```yaml
resources: {}
```

QoS class: BestEffort. It can use spare resources but receives no guarantee.

## Evidence

The unlimited-neighbour test measured p95 latency of 3 ms, and the 500m-limited-neighbour test also measured 3 ms. This shows that the serving reservation protected user-facing health traffic, so background batch work can safely be throttled first.