# HPA failure analysis

## Where today's scaler fails

Today's HPA scales on CPU utilization (target 30%, max 5). That works for the
echo backend, which burns CPU per token. It does not work for the real engine.

Measured on our own cluster, 9 Sep 2026, 32 concurrent callers against
`vllm` in the `team` namespace:

| Signal | Reading | % of budget |
|---|---|---|
| GPU utilization | 81% (293W / 300W, 76C) | busy |
| GPU memory | 41877 / 49140 MiB | 85% |
| CPU | 1069m of 4000m request | **27%** |

The GPU was saturated and the queue was filling, but CPU sat at 27% — below
the 30% target. The HPA would have held one replica through the entire surge.
The signal never crosses the line, so the scaler never fires. It is not a
tuning problem; CPU is simply not where the pressure shows up on a GPU engine.

## The signal I would deploy instead

`vllm_num_requests_waiting` — engine queue depth.

It counts requests that are actually waiting for capacity, which is the thing
users feel. It rises the moment demand exceeds what the batch can absorb, and
falls the moment capacity returns. In-flight requests saturate at max batch
size and stop rising while the queue keeps growing behind them, so they hide
the worst case. KV-cache utilization is a good second signal — our cache was
already at 85%, and once it fills vLLM preempts requests and latency collapses
— so it is worth adding later as a leading indicator.

## Starting target and tuning

Start at **5 waiting requests per replica**, scale-down window 60s.

Watch p95 latency next to replica count under a repeat of today's load:
- p95 climbs before replicas move → target is too high, lower it
- replicas move up and down repeatedly on flat traffic → widen the window
- replicas climb while p95 is already fine → target is too low, raise it

One caveat from today's run: our pods take ~50-80s to reach ready. Whatever
target we pick, the queue keeps growing for over a minute after scale-out
fires. The target has to be low enough to trigger before the pain, not during.