# DeepStream Multi-Stream Scalability Benchmark Report

## 1. Objective

Evaluate the scalability of different DeepStream pipeline configurations on Jetson by measuring the impact of increasing concurrent RTSP stream count on:

- **FPS** (frames per second per stream)
- **CPU usage** (deepstream_app process)
- **GPU usage** (GR3D via tegrastats)
- **RAM usage** (process RSS and total system RAM)

The goal is to determine the maximum number of streams each configuration can sustain before the device becomes unresponsive.

---

## 2. Test Setup

| Item | Detail |
|------|--------|
| **Hardware** | NVIDIA Jetson (unified memory architecture) |
| **Software** | NVIDIA DeepStream |
| **Input** | Same RTSP source duplicated N times |
| **Method** | Start with 1 stream, add streams one-by-one via `add_deepstream.py` until device becomes unresponsive |
| **Monitoring** | `monitor_gpt.py` logging CPU, GPU, RAM, and per-stream FPS every 1 second |
| **Stabilization** | First 10 seconds after each stream addition are excluded from steady-state analysis |

---

## 3. Configurations Tested

| Config | Model | Batch Size | Notes | Max Streams |
|--------|-------|:----------:|-------|:-----------:|
| `vehicle_b5` | Vehicle (detection + plate recognition) | 5 | — | 5 |
| `vehicle_b7` | Vehicle | 7 | — | 7 |
| `vehicle_b9` | Vehicle | 9 | — | 9 |
| `human_b9` | Human (detection + intrusion) | 9 | — | 9 |
| `human_b10_gdm_on` | Human | 10 | GDM enabled | 6 (unresponsive) |
| `face_b9` | Face (detection + recognition) | 9 | — | 9 |

---

## 4. Results by Configuration

### 4.1 Vehicle Detection — Batch Size 5

![Impact Summary](logs/plots/vehicle_b5/impact_summary.png)

- **Max streams:** 5 (limited by batch size)
- **FPS:** 14.9 → 9.7 (−35%)
- **CPU:** 18.6% → 38.6%
- **GPU:** 14.4% → 27.3%
- **Process RAM:** 3459 → 3557 MB (+98 MB total, ~25 MB/stream)
- **Lowest baseline RAM** of all configs due to smaller batch allocation

<details>
<summary>Time series & per-stream FPS</summary>

![Time Series](logs/plots/vehicle_b5/time_series.png)
![Per-Stream FPS](logs/plots/vehicle_b5/per_stream_fps.png)
</details>

---

### 4.2 Vehicle Detection — Batch Size 7

![Impact Summary](logs/plots/vehicle_b7/impact_summary.png)

- **Max streams:** 7
- **FPS:** 14.8 → 9.1 (−39%)
- **CPU:** 18.7% → 45.8%
- **GPU:** 14.6% → 38.4%
- **Process RAM:** 4153 → 4304 MB (+151 MB total, ~22 MB/stream)

<details>
<summary>Time series & per-stream FPS</summary>

![Time Series](logs/plots/vehicle_b7/time_series.png)
![Per-Stream FPS](logs/plots/vehicle_b7/per_stream_fps.png)
</details>

---

### 4.3 Vehicle Detection — Batch Size 9

![Impact Summary](logs/plots/vehicle_b9/impact_summary.png)

- **Max streams:** 9
- **FPS:** 14.1 → 7.9 (−44%)
- **CPU:** 20.4% → 50.4%
- **GPU:** 13.1% → 39.9%
- **Process RAM:** 4860 → 5045 MB (+185 MB total, ~23 MB/stream)
- **Highest CPU usage** among vehicle configs at max streams

<details>
<summary>Time series, per-stream FPS & distribution</summary>

![Time Series](logs/plots/vehicle_b9/time_series.png)
![Per-Stream FPS](logs/plots/vehicle_b9/per_stream_fps.png)
![Box Plots](logs/plots/vehicle_b9/box_plots.png)
</details>

---

### 4.4 Human Detection — Batch Size 9

![Impact Summary](logs/plots/human_b9/impact_summary.png)

- **Max streams:** 9
- **FPS:** 15.4 → 8.0 (−48%)
- **CPU:** 16.2% → 39.4%
- **GPU:** 12.5% → 33.4%
- **Process RAM:** 4821 → 4994 MB (+173 MB total, ~22 MB/stream)
- **Highest single-stream FPS** (15.4) of all configs

<details>
<summary>Time series, per-stream FPS & distribution</summary>

![Time Series](logs/plots/human_b9/time_series.png)
![Per-Stream FPS](logs/plots/human_b9/per_stream_fps.png)
![Box Plots](logs/plots/human_b9/box_plots.png)
</details>

---

### 4.5 Human Detection — Batch Size 10, GDM Enabled

![Impact Summary](logs/plots/human_b10_gdm_on/impact_summary.png)

- **Max streams:** 6 (device became unresponsive)
- **FPS:** 15.0 → 5.9 (−61%)
- **CPU:** 12.8% → 54.1%
- **GPU:** 8.4% → 26.3% (peak at 4 streams)
- **Process RAM:** 4341 → 4465 MB (+124 MB total, ~25 MB/stream)
- **GDM causes severe CPU bottleneck.** CPU spiked to 95.9% momentarily at 6 streams, causing FPS collapse and system unresponsiveness.

<details>
<summary>Time series, per-stream FPS & distribution</summary>

![Time Series](logs/plots/human_b10_gdm_on/time_series.png)
![Per-Stream FPS](logs/plots/human_b10_gdm_on/per_stream_fps.png)
![Box Plots](logs/plots/human_b10_gdm_on/box_plots.png)
</details>

---

### 4.6 Face Detection — Batch Size 9

![Impact Summary](logs/plots/face_b9/impact_summary.png)

- **Max streams:** 9
- **FPS:** 13.1 → 8.3 (−37%)
- **CPU:** 14.0% → 35.4%
- **GPU:** 9.4% → 45.0%
- **Process RAM:** 4847 → 5048 MB (+201 MB total, ~25 MB/stream)
- **Most GPU-intensive** model, reaching 45% GPU at 8–9 streams

<details>
<summary>Time series, per-stream FPS & distribution</summary>

![Time Series](logs/plots/face_b9/time_series.png)
![Per-Stream FPS](logs/plots/face_b9/per_stream_fps.png)
![Box Plots](logs/plots/face_b9/box_plots.png)
</details>

---

## 5. Cross-Configuration Comparison

### 5.1 FPS Scaling

| Streams | vehicle_b5 | vehicle_b7 | vehicle_b9 | human_b9 | human_b10_gdm | face_b9 |
|:-------:|:----------:|:----------:|:----------:|:--------:|:-------------:|:-------:|
| 1 | 14.9 | 14.8 | 14.1 | 15.4 | 15.0 | 13.1 |
| 2 | 10.0 | 9.8 | 10.5 | 10.2 | 10.1 | 10.6 |
| 3 | 9.3 | 10.0 | 8.4 | 10.3 | 10.3 | 10.8 |
| 4 | 9.9 | 10.0 | 10.0 | 10.8 | 10.1 | 10.7 |
| 5 | 9.7 | 10.2 | 9.5 | 9.6 | 9.0 | 10.6 |
| 6 | — | 8.9 | 9.5 | 8.8 | **5.9** | 9.4 |
| 7 | — | 9.1 | 8.6 | 8.1 | — | 8.7 |
| 8 | — | — | 7.8 | 8.1 | — | 8.3 |
| 9 | — | — | 7.9 | 8.0 | — | 8.3 |

**Key observation:** All configurations show a ~30% FPS drop from 1→2 streams (decoder initialization overhead), then gradual decline. The GDM-enabled config collapses at 6 streams.

### 5.2 CPU Scaling

| Streams | vehicle_b5 | vehicle_b7 | vehicle_b9 | human_b9 | human_b10_gdm | face_b9 |
|:-------:|:----------:|:----------:|:----------:|:--------:|:-------------:|:-------:|
| 1 | 18.6 | 18.7 | 20.4 | 16.2 | 12.8 | 14.0 |
| 5 | 38.6 | 37.7 | 38.0 | 29.4 | 31.2 | 26.2 |
| 9 | — | — | 50.4 | 39.4 | — | 35.4 |

**Key observation:** Vehicle model is the most CPU-hungry. Face model has the lowest CPU footprint. GDM adds disproportionate CPU load at scale.

### 5.3 GPU Scaling

| Streams | vehicle_b5 | vehicle_b7 | vehicle_b9 | human_b9 | human_b10_gdm | face_b9 |
|:-------:|:----------:|:----------:|:----------:|:--------:|:-------------:|:-------:|
| 1 | 14.4 | 14.6 | 13.1 | 12.5 | 8.4 | 9.4 |
| 5 | 27.3 | 25.6 | 19.5 | 22.3 | 25.5 | 28.2 |
| 9 | — | — | 39.9 | 33.4 | — | 45.0 |

**Key observation:** Face model is the most GPU-intensive (45% at 9 streams). GPU never reaches saturation -- the bottleneck is CPU-side (decoding, pre/post-processing).

### 5.4 RAM Scaling

| Config | Baseline (1 stream) | At Max Streams | Delta / Stream |
|--------|:-------------------:|:--------------:|:--------------:|
| vehicle_b5 | 3459 MB | 3557 MB | ~25 MB |
| vehicle_b7 | 4153 MB | 4304 MB | ~25 MB |
| vehicle_b9 | 4860 MB | 5045 MB | ~23 MB |
| human_b9 | 4821 MB | 4994 MB | ~22 MB |
| human_b10_gdm | 4341 MB | 4465 MB | ~25 MB |
| face_b9 | 4847 MB | 5048 MB | ~25 MB |

**Key observation:** RAM is dominated by one-time model/engine loading. Per-stream overhead is only ~22–25 MB (decoder buffers + metadata), regardless of model type. Higher batch sizes increase baseline RAM but not per-stream cost.

---

## 6. Batch Size Effect (Vehicle b5 vs b7 vs b9)

| Metric | b5 (5 streams) | b7 (7 streams) | b9 (9 streams) |
|--------|:-:|:-:|:-:|
| Max streams | 5 | 7 | 9 |
| FPS at max | 9.7 | 9.1 | 7.9 |
| CPU at max | 38.6% | 45.8% | 50.4% |
| GPU at max | 27.3% | 38.4% | 39.9% |
| Baseline RAM | 3459 MB | 4153 MB | 4860 MB |

- Increasing batch size from 5→9 enables 4 more concurrent streams.
- Trade-off: +1400 MB baseline RAM and +12% CPU at max load.
- FPS at max capacity decreases with higher batch sizes (9.7 → 7.9), meaning each individual stream gets slightly less throughput when fully loaded.

---

## 7. GDM Impact (human_b9 vs human_b10_gdm_on)

| Metric | human_b9 (no GDM) | human_b10_gdm_on |
|--------|:--:|:--:|
| Max streams | 9 | **6 (crashed)** |
| FPS at 5 streams | 9.6 | 9.0 |
| CPU at 5 streams | 29.4% | 31.2% |
| CPU at 6 streams | 31.3% | **54.1%** |
| Baseline RAM | 4821 MB | 4341 MB |

- GDM reduces max stream capacity by 33% (9 → 6 streams).
- CPU usage with GDM nearly doubles at 6 streams (31.3% → 54.1%).
- GDM provides lower baseline RAM (−480 MB) but this advantage is negated by the scalability loss.
- **Recommendation:** Disable GDM for multi-stream deployments unless its functionality is strictly required.

---

## 8. Conclusions

1. **CPU is the primary bottleneck**, not GPU. 

2. **All models sustain ~8–10 FPS per stream** at high concurrency (7–9 streams). If the target is ≥10 FPS, the practical limit is approximately 4–5 streams.

3. **Batch size directly controls max streams** but comes at a cost of higher baseline RAM (~700 MB per +2 batch size for the vehicle model).

4. **GDM should be disabled** for multi-stream workloads. It causes a CPU bottleneck that halves the maximum stream capacity.

5. **Face model is the most GPU-heavy** (45% at 9 streams) but has the lowest CPU usage (35.4%). Human and vehicle models are more CPU-bound.

6. **RAM is not a limiting factor** for stream scaling. Per-stream overhead is negligible (~25 MB) compared to the one-time model loading cost.
