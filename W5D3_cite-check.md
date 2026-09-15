# Site Check: HUMAIN Dammam
**Team07 Pod:** ~/aidc/site-check.md  
**Submission Deadline:** 12:00 (15 September 2026)

---

### Site Selected
* **Site:** HUMAIN Dammam (Initial stage: 100 MW connection).

---

### Question 1: Racks and GPUs
* **Assumptions & Working:**
  * Total Power = $100\text{ MW}$.
  * Power after PUE 1.25: $100\text{ MW} / 1.25 = 80\text{ MW}$.
  * Power after 20% Headroom: $80\text{ MW} / 1.20 = 66.67\text{ MW}$.
  * Power after 10% Switches/Storage: $66.67\text{ MW} \times 0.90 = 60.0\text{ MW}$ to $60.6\text{ MW}$ available for compute.
* **Answers:**
  * **DGX H100 Fleet:** $60.6\text{ MW} / 40.8\text{ kW per rack} = \mathbf{1,485\text{ racks}}$ (~5,940 nodes, **~47,500 GPUs**).
  * **GB300 NVL72 Fleet:** $60.6\text{ MW} / 120\text{ kW per rack} = \mathbf{505\text{ racks}}$ (**~36,400 GPUs**).

---

### Question 2: Largest Open Model & Copy Count
* **Assumptions & Working:**
  * Largest open-weight model: **Kimi K3** (2.8T MoE).
  * Model weight memory at FP8 = ~$2.8\text{ TB}$.
  * KV Cache (32 conversations @ 128K context) = ~$0.3\text{ TB}$.
  * Total footprint per copy = ~$3.1\text{ TB}$.
* **Answers:**
  * **DGX H100:** $\mathbf{1,190\text{ copies}}$ (at 5 nodes / 40 GPUs per copy).
  * **GB300 NVL72:** $\mathbf{3,030\text{ copies}}$ (at 6 GPUs / ~0.08 racks per copy).

---

### Question 3: Largest Model Trainable in 6 Months
* **Assumptions & Working:**
  * Time: $6\text{ months} = 15.77 \times 10^6\text{ seconds}$.
  * Compute: $47,500\text{ H100 GPUs} \times (989\text{ TFLOPS} \times 0.40) \approx 1.88 \times 10^{19}\text{ FLOP/s}$.
  * Total compute capacity over 6 months: $\approx 2.96 \times 10^{26}\text{ FLOPs}$.
  * Formula: $\text{FLOPs} = 6 \times P \times (20 P) = 120 P^2$.
  * $P = \sqrt{(2.96 \times 10^{26}) / 120} \approx 1.57 \times 10^{12}\text{ parameters}$.
* **Answer:**
  * A **1.6 Trillion parameter model** trained on **31 Trillion tokens**.

---

### Question 4: Monthly Electricity Bill
* **Assumptions & Working:**
  * Average draw at 65% connection: $100\text{ MW} \times 0.65 = 65\text{ MW}$.
  * Monthly energy draw: $65,000\text{ kW} \times 720\text{ hours} = \mathbf{46.8\text{M} - 47.5\text{M kWh}}$.
* **Answers:**
  * **Standard Rate ($0.08 / kWh / 30 halalas):** **USD 3.8 million** (SAR 14.2 million).
  * **Industrial Rate ($0.048 / kWh):** **USD 2.3 million** (SAR 8.5 million).

---

### Question 5: Cost per Million Tokens
* **Assumptions & Working:**
  * Facility OpEx (non-electricity) = $\$450,000 / \text{MW} / \text{month} \times 100\text{ MW} = \$45.0\text{M}$.
  * Total Monthly Cost = $\$45.0\text{M} + \$3.8\text{M electricity} = \mathbf{\$48.8\text{ million}}$.
  * Output benchmark: 125 tokens/sec/GPU.
* **Answers:**
  * **H100 Fleet:** **USD 10.40** at 30% capacity sold | **USD 3.90** at 80% capacity sold.
  * **GB300 Fleet:** **USD 13.60** at 30% capacity sold | **USD 5.10** at 80% capacity sold.
  * *Note:* Electricity is <8% of total costs; selling higher capacity cuts unit costs by nearly 3x.

---

### One Thing the Announcement Does Not Tell Us
* **Grid Interconnect Timeline & Substation Readiness:** The press releases detail land allocation for up to 2 GW campus capacity across 10 buildings, but completely omit high-voltage substation commissioning dates, utility interconnect agreements, and power delivery schedules required to actually energize even the initial 100 MW site.
