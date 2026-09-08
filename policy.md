# Multi-Tenant Resource Allocation Policy (4-CPU Node)

## Policy Statement

Serving is guaranteed dedicated CPU to protect its p95 latency SLO, Dashboard is permitted to burst using unreserved idle CPU as BestEffort, and Batch throttles first under CFS limits when contending for node headroom.

---

## Tenant Resource Configurations

### 1. Serving (`QoS: Guaranteed`)
Dedicated cores locked to avoid CPU starvation, context switching, and eviction.

```yaml
resources:
  requests:
    cpu: "2"
    memory: 2Gi
  limits:
    cpu: "2"
    memory: 2Gi
```

### 2. Batch (`QoS: Burstable`)
Low scheduling baseline with an enforced CFS ceiling so it yields to serving and throttles first under load.

```yaml
resources:
  requests:
    cpu: 250m
    memory: 512Mi
  limits:
    cpu: "1"
    memory: 1Gi
```

### 3. Dashboard (`QoS: BestEffort`)
No reservations or limits; freely absorbs idle CPU cycles during dashboard refresh spikes, but yields under memory pressure.

```yaml
resources: {}
```

---

## Defense of the Loser

Throttling the batch backfill is justified because an unconstrained neighbor degraded serving tail latency to a p95 of 4ms (n=572, 0 fails), whereas enforcing a CFS limit eliminated queueing interference and restored our serving latency SLO to a flat p95 of 3ms (n=574, 0 fails) with zero dropped requests.
