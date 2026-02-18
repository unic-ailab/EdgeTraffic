# EdgeTraffic

EdgeTraffic is an EdgeAI dataset that aligns real world road traffic dynamics with fine grained system telemetry from a production grade smart traffic monitoring service.
This repository hosts the dataset, lightweight loaders, and example analyses.

## What the dataset captures

EdgeTraffic was collected from a road intersection in Iași, Romania where six HD IP cameras stream video at 25 FPS over a private 5G network to a GPU powered Multi-access Edge Computing node running a video analytics pipeline.

The key idea is the alignment between

- application-level model outputs such as per frame car counts
- performance indicators such as per frame inference latency
- system metrics such as GPU and CPU utilization, memory usage, power draw, and network activity

To enable controlled comparisons across hardware and model variants, the dataset also includes offline replay runs of a one-hour reference video on additional devices and YOLOv8 variants.

*Important note about privacy:* During the live field trials, operational constraints prevented making raw video content publicly available. The dataset therefore focuses on derived analytics and time aligned system logs rather than publishing the full captured footage.

## Dataset layout

The dataset is organized as a set of trial directories. Each directory corresponds to one experimental trial, defined by a hardware target, a model variant, and a capture mode.

Each trial directory contains

- data.csv  
  high frequency time series measurements
- metadata.yaml  
  execution context and trial metadata

A typical layout is

```text
.
├── dataset
│   ├── v100_yolov8x_live_001
│   │   ├── data.csv
│   │   └── metadata.yaml
│   ├── t4_yolov8m_replay_001
│   │   ├── data.csv
│   │   └── metadata.yaml
│   └── orin_yolov8n_replay_001
│       ├── data.csv
│       └── metadata.yaml
|   ...
└── analysis.ipynb
```

The exact directory names may differ across releases, but the per trial structure is consistent.

## Data schema

Each row in data.csv corresponds to one processed frame.

The main fields are

- **fseqno:** frame sequence number within the stream
- **timestamp:** Unix epoch time in seconds at 25 FPS multiple frames share the same timestamp value
- **car_count:** number of cars detected in the frame by the deployed model
- **inf_latency:** per frame model inference time in seconds it excludes stable pipeline overheads such as batching and message serialization

Monitoring metrics fields, when available

- **gpu_util:** GPU utilization in percent
- **gpu_power:** GPU power draw in Watts
- **gpu_mem:** GPU memory usage in GB
- **cpu_util:** host CPU utilization in percent
- **cpu_power:** CPU power draw in Watts
- **cpu_mem:** host memory usage in Bytes
- **network:** outgoing traffic from edge to cloud in Bytes

Metric availability varies across hardware targets due to driver and exporter limitations. The V100 MEC server provides the full set above, while the T4 server and Jetson AGX Orin expose a subset.

## metadata.yaml

Each trial directory includes a metadata.yaml file that documents the execution context. The file includes a unique trial identifier, capture type, device details, model configuration, pipeline details, and the time window.

Example

```yaml
trial_id: v100_yolov8x_live_001
video_type: live
duration: 120
date: "28-01-2025"
device:
  name: Server
  gpu_model: NVIDIA V100
model:
  family: YOLOv8
  variant: X
  input_size: [640, 640]
pipeline:
  framework: Savant
  sdk: NVIDIA DeepStream
  fps: 25
time:
  start: 1738041386
  end: 1738048586
output:
  csv_file: data.csv
```

## Getting started

Clone the repository

```bash
git clone https://github.com/unic-ailab/EdgeTraffic.git
cd EdgeTraffic
```

If you downloaded the dataset as an archive, place the trial directories under the dataset directory as shown above.

For the Python environment, you need `Pandas`  

## Quick example

Load a trial and aggregate the per frame car counts to per second averages

```python
import pandas as pd

df = pd.read_csv("dataset/v100_yolov8x_live_001/data.csv")
per_sec = df.groupby("timestamp")["car_count"].mean()
print(per_sec.head())
```

## Reproducing paper figures

The paper includes example analyses such as

- traffic pattern discovery using temporal aggregation and smoothing
- accuracy degradation and under counting when switching to smaller YOLOv8 variants
- device dependent correlations between latency, resource saturation, and energy consumption

This repository provides analysis notebook (`analysis.ipynb`). For a typical run flow, start `Jupyter` and execute the notebook file.

## Contributing new trials

The dataset schema is designed to be extensible.

If you contribute a new trial, keep these invariants

- one directory per trial
- `data.csv` uses the same column names and units where possible
- `metadata.yaml` includes device, model, pipeline, and time window information

If a metric is unavailable on your platform, omit the column and record the limitation in metadata.yaml.

## License

See LICENSE file for the code and dataset license.

## Acknowledgements

This work was supported by the European Union TriasNet Horizon JU SNS 2022 programme through an open call for third party access to B5G testbeds. The production pipeline builds on Savant AI and the NVIDIA DeepStream SDK.

## Contact

For questions and collaborations, open a GitHub issue or contact the authors: `msymeo03@ucy.ac.cy` or `trihinas.d@unic.ac.cy`.
