# HPA Failure Analysis: CPU vs. GPU-Bound Serving

## 1. Where Today's Scaler Fails
The CPU-utilization HPA fails on the real engine because vLLM is GPU-bound. Under concurrent load, the GPU saturates its compute and memory bandwidth, yet host CPU peaked at only ~1030m (~25% of a 4-core allocation) before dropping to 18m (0.45%) at idle. Because CPU never sustains above the 30% or 50% target, the HPA holds 1 replica indefinitely, starving incoming requests in an unmonitored queue.

## 2. Signal Deployed Instead
* **Signal:** Engine queue depth (`vllm:num_requests_waiting` via Prometheus metrics).
* **Why:** Unlike CPU%, queue depth directly indicates when token generation requests exceed the engine's continuous batching capacity. A non-zero queue signals immediate TTFT (Time-to-First-Token) degradation.

## 3. Starting Target & Tuning
* **Starting Target:** Average queue depth of 5 waiting requests per pod.
* **Tuning Metric:** Observe p95 Time-to-First-Token (TTFT) and KV-cache usage factor (`vllm:gpu_cache_usage_factor`). If TTFT spikes or preemption occurs before the HPA triggers, lower the target threshold to 2–3 requests.
