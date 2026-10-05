# VM vs Container Performance Analysis

## 1. Project Overview

This project compares the performance of Virtual Machines (VMs) and Containers across three core system metrics:

CPU performance
Memory performance
Disk I/O performance

The goal is to see how much overhead full virtualization adds, and how that compares with the lighter-weight approach containers take.

---

## 2. Objective

This experiment aims to:

Compare the CPU performance of a VM and a container.
Compare their memory performance.
Compare their disk I/O performance.
Measure the overhead each virtualization approach adds.
Find out how close a container's performance comes to that of the underlying host.

---

## 3. VM vs Container

A VM emulates a full set of hardware and runs its own guest operating system on top of it. Because every VM carries its own OS, it uses more CPU, memory, and storage than a container.

A container isolates an application but shares the host's kernel instead of booting its own OS. With no guest OS to run, containers usually carry less overhead.



### Virtual Machine

A Virtual Machine virtualizes the hardware and runs a complete guest operating system.

```text
Application
     ↓
Guest Operating System
     ↓
Virtual Hardware
     ↓
Hypervisor
     ↓
Host Operating System
     ↓
Physical Hardware
```

A VM requires its own operating system, which introduces additional CPU, memory, and storage overhead.

### Container

A container isolates applications while sharing the host operating system kernel.

```text
Application
     ↓
Container
     ↓
Host Operating System Kernel
     ↓
Physical Hardware
```

Because containers do not require a separate guest operating system, they generally have lower resource overhead.

---

## 4. Performance Parameters

### 4.1 CPU Performance

CPU performance measures how efficiently the VM or container performs computational workloads.

Important metrics include:

* Execution time: 30.0005 sec
* Events per second: 6900.86
* CPU utilization: 
* CPU latency: 0.58 ms

Higher events per second and lower execution time generally indicate better CPU performance.

---

### 4.2 Memory Performance

Memory performance evaluates how efficiently the system performs memory operations.

Metrics include:

* Memory bandwidth
* Memory latency
* Read/write throughput
* Memory utilization

Higher memory bandwidth and lower latency indicate better memory performance.

---

### 4.3 Disk I/O Performance

Disk I/O measures the performance of reading and writing data.

The experiment considers:

* Sequential read
* Sequential write
* Random read
* Random write
* IOPS
* Throughput
* Latency

Higher throughput and IOPS, along with lower latency, indicate better storage performance.

---

## 5. Experimental Setup

The experiment uses equivalent resource configurations wherever possible.

### VM

* Virtualization: Virtual Machine
* CPU: 4 cores
* Memory: 8 GB
* Storage: 60 GB
* Operating System: Ubuntu
* Hypervisor: Type - 2

### Container

* Container Technology: Docker
* Memory Limit: 8 GB
* Storage: 60 GB

### Host System

* CPU: [Specify CPU]
* RAM: [Specify RAM]
* Storage: [Specify SSD/HDD]
* Operating System: [Specify OS]

---

## 6. Benchmark Tools

The following tools can be used for the experiment:

### CPU

```bash
sysbench cpu --cpu-max-prime=20000 run
```

### Memory

```bash
sysbench memory --memory-total-size=10G run
```

### Disk I/O

```bash
sysbench fileio --file-total-size=2G prepare
sysbench fileio --file-total-size=2G --file-test-mode=seqwr run
sysbench fileio --file-total-size=2G --file-test-mode=seqrd run
sysbench fileio --file-total-size=2G cleanup
```

Docker can be used to execute the same benchmarks inside a container.

---

## 7. Methodology

The same benchmark workload is executed in both environments.

### Step 1: Set up the VM
1. Create a VM (VMware / KVM) with **2 vCPU, 2 GB RAM** and install Ubuntu 22.04.
2. Install the tools:
```bash
sudo apt update
sudo apt install -y sysbench fio iperf3 apache2-utils python3-pip
```

### Step 2: Set up the container (same limits)
```bash
docker run -it --name bench --cpus=2 --memory=2g ubuntu:22.04 bash
# inside the container:
apt update && apt install -y sysbench fio iperf3
```
Or build the provided image: `docker build -t bench ./docker`

### Step 3: Baseline CPU test (run in both VM and container)
```bash
sysbench cpu --threads=2 --time=30 run
```

### Step 4: CPU scalability
```bash
for t in 1 2 4 8; do
  sysbench cpu --threads=$t --cpu-max-prime=20000 --time=30 run
done
```
Or: `./scripts/run_cpu.sh results/raw/cpu`

### Step 5: Memory
```bash
for t in 1 2; do
  sysbench memory --threads=$t --memory-block-size=1M \
    --memory-total-size=512M --memory-oper=write run
done
```
Or: `./scripts/run_memory.sh results/raw/memory`

### Step 6: Disk I/O (fio)
```bash
# Sequential, 1 MB blocks
fio --name=seqread  --rw=read  --bs=1M --size=512M --direct=1 --runtime=30 --time_based
fio --name=seqwrite --rw=write --bs=1M --size=512M --direct=1 --runtime=30 --time_based

# Random, 4 KB blocks
fio --name=randread  --rw=randread  --bs=4k --size=512M --iodepth=4 --direct=1 --runtime=30 --time_based
fio --name=randwrite --rw=randwrite --bs=4k --size=512M --iodepth=4 --direct=1 --runtime=30 --time_based
```
Or: `./scripts/run_disk.sh results/raw/disk`

### Step 7: Network (iperf3)
```bash
# Terminal 1: start server
iperf3 -s

# Terminal 2: client
iperf3 -c 127.0.0.1 -t 30      # VM (loopback)
iperf3 -c 172.17.0.1 -t 30     # Container (docker0 bridge)
```
Or: `./scripts/run_network.sh server` and `./scripts/run_network.sh client 172.17.0.1 results/raw/network/container-network-bridge.txt`

### Step 8: FastAPI application test
1. Start the app (port 8000) in the VM and in a container:
```bash
cd api && pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000      # VM

docker build -t fastapi-bench ./api
docker run -p 8000:8000 --cpus=2 --memory=2g fastapi-bench   # Container
```
2. Check it works: `curl http://localhost:8000/health`
3. Load test with ApacheBench:
```bash
ab -n 10000 -c 100 http://localhost:8000/health
ab -n 1000  -c 10  http://localhost:8000/compute
ab -n 1000  -c 10  http://localhost:8000/memory
```

### Step 9: Repeat, then analyse
- Repeat each test at least 2 to 3 times and close other applications.
- Generate graphs and CSVs, then compare in the terminal:
```bash
python scripts/generate_plots.py
python scripts/analyze_results.py
```

---

## 8. Results

### CPU (sysbench, events/sec)

| Threads | VM | Container | Better |
| :---: | ---: | ---: | :---: |
| 1 | 515.84 | 517.19 | Container (+0.26%) |
| 2 | 883.55 | 894.38 | Container (+1.23%) |
| 4 | 928.17 | 900.45 | VM (+3.08%) |
| 8 | 905.17 | 914.42 | Container (+1.02%) |

Throughput stops growing after 2 threads because only 2 cores are available. Latency rises with thread count instead.

### Memory (sequential write, MiB/s)

| Threads | VM | Container |
| :---: | ---: | ---: |
| 1 | **9,541.97** | 5,152.43 |
| 2 | **9,880.38** | 6,970.16 |

### Disk I/O (fio)

| Test | VM | Container | Better |
| --- | ---: | ---: | :---: |
| Sequential read (1 MB) | 461 MiB/s | **500 MiB/s** | Container |
| Sequential write (1 MB) | **358 MiB/s** | 291 MiB/s | VM |
| Random read (4 KB) | 1,313 IOPS | **1,767 IOPS** | Container (+34.58%) |
| Random write (4 KB) | 1,331 IOPS | **1,346 IOPS** | Container (+1.13%) |

### Network (iperf3, 30 s)

| Metric | VM (127.0.0.1) | Container (172.17.0.1) |
| --- | ---: | ---: |
| Sender | 14.1 Gbits/s | 13.7 Gbits/s |
| Receiver | 14.1 Gbits/s | 10.3 Gbits/s |
| Data transferred | 49.3 GB | 47.9 GB |
| TCP retransmissions | 3 | 13 |

### FastAPI (ApacheBench, 0 failed requests)

| Endpoint | VM req/s | Container req/s | VM mean latency | Container mean latency |
| --- | ---: | ---: | ---: | ---: |
| `/health` (c=100, n=10,000) | **419.79** | 371.07 | **238.21 ms** | 269.49 ms |
| `/compute` (c=10, n=1,000) | **12.24** | 10.76 | **817.31 ms** | 929.47 ms |
| `/memory` (c=10, n=1,000) | **16.43** | 14.40 | **608.50 ms** | 694.62 ms |

---

## 9. Graphs
### Overall Dashboard
<img width="468" height="436" alt="image" src="https://github.com/user-attachments/assets/a06fc275-b6ae-41c7-b230-de0850790ca4" />

### CPU Scalability
<img width="3000" height="1100" alt="cpu_scalability" src="https://github.com/user-attachments/assets/2196509f-f7fd-4e02-b754-125ee9584c43" />


### Memory Performance
<img width="3000" height="1100" alt="memory_performance" src="https://github.com/user-attachments/assets/f0df7efd-3efa-4b58-b955-70f9de1b073c" />


### Disk I/O Performance
<img width="3000" height="1100" alt="disk_io_performace" src="https://github.com/user-attachments/assets/7a32cbd4-ce83-4e8f-963a-5644e118fbdb" />


### Network Performance
<img width="3000" height="1100" alt="network_performance" src="https://github.com/user-attachments/assets/bcea8921-13f3-405a-b53b-173bb1eed2a8" />


### FastAPI Performance
<img width="3000" height="1100" alt="fastapi_performance" src="https://github.com/user-attachments/assets/d0315414-fca7-4946-a0be-1b25c96ca0fe" />

---

## 10. Notes and Limitations

- Network test is not like-for-like: the VM used loopback while the container used the Docker bridge.
- Results come from one environment and a small number of runs; treat differences of a few percent as noise.
- However, the actual results depend on:
     * Hardware
     * Hypervisor
     * Container runtime
     * Storage technology
     * Filesystem
     * CPU allocation
     * Memory allocation
     * Disk configuration
     * Background processes

Therefore, the experimental measurements should be used rather than assuming that one environment will always be faster for every workload.

---

## 11. VM vs Container Comparison

| Feature              | Virtual Machine       | Container              |
| -------------------- | --------------------- | ---------------------- |
| Virtualization level | Hardware              | Operating-system level |
| Guest OS             | Required              | Not required           |
| Kernel               | Separate guest kernel | Shared host kernel     |
| CPU overhead         | Generally higher      | Generally lower        |
| Memory overhead      | Higher                | Lower                  |
| Disk overhead        | Can be higher         | Usually lower          |
| Startup time         | Higher                | Lower                  |
| Isolation            | Strong                | Lightweight            |
| Portability          | High                  | High                   |
| Resource efficiency  | Lower                 | Higher                 |

---

## 12. Advantages of Containers

* Lightweight
* Fast startup
* Lower memory overhead
* Efficient resource utilization
* Easy deployment
* Good performance for many application workloads
* Multiple containers can run on the same host efficiently

---

## 13. Advantages of Virtual Machines

* Strong isolation
* Complete guest operating system
* Can run different operating systems
* Useful for infrastructure virtualization
* Suitable when kernel-level isolation is required
* Mature virtualization ecosystem

---

## 14. Conclusion

- This experiment compares VM and container performance using CPU, memory, and disk I/O benchmarks.
- Containers generally introduce less resource overhead because they share the host operating system kernel. VMs provide stronger isolation and complete operating-system environments but require additional resources for the guest OS and virtual hardware.
- The benchmark results provide a quantitative basis for understanding the performance trade-offs between the two virtualization approaches.
- **CPU:** containers run on the host kernel, so CPU speed is about the same as the VM.
- **Disk:** containers did better on reads and random I/O; the VM did better on sequential writes.
- **Memory, network, app:** the VM was faster here. For containers, the `docker0` bridge, `veth` pair and NAT add a small cost (about 13 to 14% in the FastAPI test).
- **Use containers** for microservices, CI/CD and fast scaling. **Use VMs** when you need strong isolation or a different OS kernel.
---

## 15. Repository Structure

```text
vm-vs-container-performance/
├── api/
│   ├── Dockerfile
│   ├── main.py
│   └── requirements.txt
├── docker/
│   └── Dockerfile
├── figures/
│   ├── cpu_scalability.png
│   ├── disk_io_performance.png
│   ├── fastapi_performance.png
│   ├── graphs.py
│   ├── memory_performance.png
│   ├── network_performance.png
│   └── overall_performance_dashboard.png
├── processed/
│   ├── api_results.csv
│   ├── cpu_results.csv
│   ├── disk_results.csv
│   ├── memory_results.csv
│   ├── network_results.csv
│   └── summary_comparison.csv
├── results/
│   └── raw/
├── scripts/
├── .gitignore
└── README.md
```

## 16. Technologies Used

* Virtual Machines
* Docker
* Linux
* Sysbench
* CPU Benchmarking
* Memory Benchmarking
* Disk I/O Benchmarking
* Shell Scripting
* Data Analysis
