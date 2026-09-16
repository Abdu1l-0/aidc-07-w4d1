# Service report

Team: Team 13
Use case: OpenAI-compatible LLM serving platform for paired Agentic AI cohort agents running multi-turn reasoning, tool calling and structured completions
Service and model: vLLM engine at `team-serving:8000` (external `https://t13.aidc.nadir.sh`), namespace `team`; the single model advertised by `/v1/models` on this endpoint
Measured requests or tasks: Chat completion requests to `/v1/chat/completions` that reached the engine and were retired successfully
Indicator and unit: 95th percentile Time to First Token (TTFT), in seconds
SLO target and window: TTFT p95 below 0.120 s, over a 24-hour rolling window; the reported value comes from a 5-minute rate window inside that SLO window
Measurement start and end: 2026-09-10 11:23 to 11:30 UTC+3 (2026-09-10 08:23 to 08:30 UTC), inside the wider 11:00-11:30 UTC+3 session
Workload: Outside-in verification suite plus client chat completions, `max_tokens` clamped to 2048 and prompts up to 1024 tokens; concurrency was not fixed or recorded
Observed result and sample count: TTFT p95 held a plateau near 0.039 s (39 ms), with a low point of 0.028 s (28 ms) during ramp-up; the parallel success-rate panel showed about 24.3 requests/min between 11:25 and 11:28 UTC+3, so roughly 70-75 successful completions back the plateau. The exact request count was not captured from the counter, so the sample count is an estimate.
Evidence: Grafana panels "Time to First Token (TTFT)" and "Completed Requests per Minute" on the team dashboard; indicator definitions in `my-slo-targets.md`; alert notification records in `notification-evidence.jsonl`; exported rule in `my-alert.json`
Conclusion: insufficient evidence
Limitations: The measurement covers about 7 minutes of one session, against a 24-hour rolling SLO window, so it cannot establish compliance over that window. The sample count is an estimate read from a rate panel rather than a counter difference. Traffic was artificial and bounded, so it does not represent real agent usage patterns, burst behaviour or long-context prompts. Prometheus scrapes the engine in-cluster, so the value excludes Caddy proxy processing and Cloudflare tunnel edge transit; real clients experience more latency than reported here. Scrape gaps and idle periods leave the rate query without data, and a histogram quantile over few observations is unstable. A successful HTTP response says nothing about whether the model output was correct or useful; output quality was not evaluated.
Follow-up action: Re-measure TTFT p95 over a full 24-hour window with continuous traffic, and record the exact request count from `vllm:request_success_total` at the window edges rather than reading a rate panel. Add an external probe outside the cluster so proxy and tunnel transit are included, and compare it with the in-cluster value to size the gap. Raise or confirm the 0.120 s threshold once a real distribution is available.

## Measurement query

```promql
histogram_quantile(0.95, sum(rate(vllm:time_to_first_token_seconds_bucket[5m])) by (le))
```

Evaluated as an instant query while traffic was running, between 11:23 and 11:30
UTC+3 on 2026-09-10. Each evaluation summarises the preceding 5 minutes of
observations.

The measurement is taken at the vLLM pod's `/metrics` interface, scraped by
Prometheus inside the `team` namespace. It therefore reports engine-internal
time: request scheduling, queueing in the batch scheduler, and prompt prefill up
to the first output token.

It excludes everything outside the pod: Caddy proxy handling, the Cloudflare
tunnel, and general network transit between the client and the cluster. It also
excludes requests that never reached the engine, such as rejected or
unauthenticated calls and clients that disconnected before generation started.
The quantile is interpolated from histogram buckets, so its precision is limited
by bucket boundaries and by how many observations fall in the window.

## Service alert

Condition and unit: Threshold C fires when the last value of query A, the 95th percentile TTFT in seconds, is above 0.120
Evaluation interval: 1m
Pending period: 5m
Relationship to the SLO: Query A is the same expression that produces the reported SLI, and the threshold is the SLO target itself, so the alert fires exactly when the current 5-minute view of the indicator sits outside the documented objective. The 24-hour SLO window is longer than any single evaluation, so the alert is an early warning that budget is being spent, not a statement that the SLO has been broken. The pending period keeps a single slow scrape from firing, while 5 minutes of continuous breach is long enough to matter for agents waiting on first tokens.
First response to a notification: Confirm the alert is real by opening the TTFT panel and the Completed Requests per Minute panel side by side for the same period. Then check vLLM queue depth and GPU utilisation on the model pod to see whether the cause is load, worker starvation or a stalled batch, and check whether a deployment or config change landed just before the rise.

## Notification test

Firing received at: 2026-09-14 08:26:20 UTC (alert `startsAt` 08:26:20 UTC, values A=1, B=1, C=1)
Resolved received at: 2026-09-14 08:27:10 UTC (alert `endsAt` 08:27:00 UTC, values A=0, B=0, C=0)
What the test establishes: That a Grafana-managed rule in the AIDC lab folder can detect a condition becoming true, fire, deliver a webhook notification to the Lab inbox contact point, and then deliver a matching resolved notification when the condition becomes false again. The delivery path from Grafana to the in-namespace receiver works in both directions. It establishes nothing about the model service: the signal came from `vector(time() % 120 < bool 60)`, which reads the query evaluation clock and never touches a vLLM metric. It does not test the service query, its threshold, or the recovery of any real service condition.
