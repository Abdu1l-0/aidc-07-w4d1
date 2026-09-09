# HPA failure analysis

1. A CPU-based HPA fails on our vLLM engine because the GPU reached 94% utilization while CPU usage was only 1076m, about 27% of its 4-CPU request. A 50% CPU target would therefore keep one replica while the GPU was nearly saturated.

2. I would scale on `vllm_num_requests_waiting` because queue depth directly measures requests that the engine cannot begin serving.

3. I would start with a target of no more than 2 waiting requests per replica. I would tune it by watching p95 time to first token, end-to-end p95 latency, GPU utilization, and KV-cache utilization.