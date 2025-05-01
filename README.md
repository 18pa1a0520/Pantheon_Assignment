
# TCP Congestion Control Algorithms: Evaluation with Pantheon

This project evaluates the performance of three TCP congestion control algorithms—**Cubic**, **Vegas**, and **BBR**—under varying network conditions using [Pantheon](https://github.com/StanfordSNR/pantheon) and [Mahimahi](https://github.com/ravinet/mahimahi).

---

## Experiment Overview

**Algorithms Compared**
- Cubic
- Vegas
- BBR

**Network Scenarios**
1. **Low Latency / High Bandwidth**  
   - Bandwidth: 50 Mbps  
   - Round-Trip Time (RTT): 10 ms  

2. **High Latency / Low Bandwidth**  
   - Bandwidth: 1 Mbps  
   - RTT: 200 ms  

**Collected Metrics**
- Throughput (average and time-series)
- Round-Trip Time (average and 95th percentile)
- Packet Loss (time-series and total)

---

##  Test Environment

- **Host OS**: macOS  
- **Guest OS**: Ubuntu 24.04 LTS (via UTM VM)  
- **Pantheon Dependencies**: Python 2.7, Git, Submodules  
- **Mahimahi Dependencies**: Autotools, `libprotobuf-dev`, Apache2, Iptables, etc.

---

## Installation Steps

### 1. Clone Pantheon and Install Dependencies
```bash
git clone https://github.com/StanfordSNR/pantheon.git
cd pantheon
sudo ./tools/install_deps.sh
git submodule update --init --recursive
```

### 2. Install Mahimahi

#### Option A: From Package Manager
```bash
sudo apt-get install mahimahi
```

#### Option B: Build from Source
```bash
git clone https://github.com/ravinet/mahimahi
cd mahimahi
./autogen.sh
./configure
make
sudo make install
```

---

## Project Layout

```
pantheon/
├── output/
│   ├── HighLatencyLowBandwidth/
│   └── LowLatencyHighBandwidth/
├── traces/
│   ├── 50mbps_uplink.trace
│   ├── 50mbps_downlink.trace
│   ├── 1mbps_uplink.trace
│   └── 1mbps_downlink.trace
├── analysis/
│   └── rtt_vs_throughput_plot.py
```

---

##  Running Experiments

### Generate Network Traces

```bash
# 50 Mbps trace (~4 packets/ms for 60 seconds)
python2 -c "import sys; [sys.stdout.write('1500\n') for _ in range(4167)]" > traces/50mbps.trace

# 1 Mbps trace (1 packet every 12ms)
python2 -c "import sys; [sys.stdout.write('1500\n') for _ in range(83)]" > traces/1mbps.trace
```

### Run Low Latency / High Bandwidth Test
```bash
python src/experiments/test.py local \
  --schemes "cubic vegas bbr" \
  --data-dir results/low-latency-high-bandwidth/ \
  --runtime 60 \
  --uplink-trace traces/50mbps.trace \
  --downlink-trace traces/50mbps.trace \
  --prepend-mm-cmds "mm-delay 5" \
  --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail \
    --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

### Run High Latency / Low Bandwidth Test
```bash
python src/experiments/test.py local \
  --schemes "cubic vegas bbr" \
  --data-dir results/high-latency-low-bandwidth/ \
  --runtime 60 \
  --uplink-trace traces/1mbps.trace \
  --downlink-trace traces/1mbps.trace \
  --prepend-mm-cmds "mm-delay 100" \
  --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail \
    --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

---

##  Analyzing Results

### Generate Reports
```bash
python ./src/analysis/analyze.py --data-dir output/LowLatencyHighBandwidth
python ./src/analysis/analyze.py --data-dir output/HighLatencyLowBandwidth
```

---

##  Outputs

- **Logs**: `results/*/*.log`  
- **Metadata**: `pantheon_perf.json`, `pantheon_metadata.json`  
- **Visualizations**: `pantheon_report.pdf`

---

## Notes

- Make sure Python 2.7 is installed, as Pantheon is not compatible with Python 3.
- It's recommended to run tests in a VM or containerized environment to avoid host network interference.

---
