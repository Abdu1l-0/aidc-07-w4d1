HPA FAILURE ANALYSIS: CPU-based autoscaling on the real engine

Where it fails:
The echo backend's HPA watches CPU because echo is CPU-bound. The real vLLM
engine is GPU-bound. Measured: 32 concurrent callers for 90s moved the
engine's CPU from 17m to 919m out of a 4000m request (~23% at peak), while
nvidia-smi showed the GPU at 91% utilization and 42GB/49GB memory in use. The
engine was fully saturated while CPU stayed low enough that this week's HPA
(target 30%) would never scale out - it would hold one replica through a real
outage.

Signal I'd use instead:
vllm_num_requests_waiting (engine queue depth). It directly reflects whether
the engine is falling behind, which CPU clearly does not for this workload.
KV-cache utilization is a reasonable second signal, since cache pressure can
hurt throughput before the queue looks alarming.

Starting target and what I'd tune by:
Start scaling out around 5-10 waiting requests per replica, sustained briefly
to avoid noise. Tune by checking how queue depth correlates with p95 latency
(week 3's harness) to find the point p95 crosses the SLO, and watch GPU
memory too - scaling only helps if there's still GPU headroom to use.
