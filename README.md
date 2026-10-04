# VM vs Container Performance Analysis

A hands-on comparison of **Virtual Machines (VMs)** and **Docker containers** using CPU, memory, disk I/O, network and a small FastAPI application as benchmarks.

**Author:** Soumya Surpur · Cloud Computing Laboratory, Experiment 2

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [VM vs Container Explained](#2-vm-vs-container-explained)
3. [Experimental Setup](#3-experimental-setup)
4. [Execution Steps](#4-execution-steps)
5. [Results](#5-results)
6. [Graphs](#6-graphs)
7. [Why the Results Look This Way](#7-why-the-results-look-this-way)
8. [Notes and Limitations](#8-notes-and-limitations)
9. [Conclusion](#9-conclusion)
10. [Repository Structure](#10-repository-structure)
11. [Technologies Used](#11-technologies-used)

---

## 1. Project Overview

This project measures the performance overhead of virtualization by running the **same benchmarks** in a VM and in a container, both limited to **2 CPUs and 2 GB RAM**.

**Objectives**

1. Compare CPU, memory, disk I/O and network performance of a VM and a container.
2. Measure the overhead each approach adds.
3. Check how a real application (FastAPI) behaves in both.
4. Decide which one suits which kind of workload.

**Tools used:** `sysbench` (CPU, memory), `fio` (disk), `iperf3` (network), `ApacheBench` (FastAPI load test).

---

## 2. VM vs Container Explained

Both let you run many isolated environments on one physical machine. They differ in **what is isolated**.

> **Simple analogy:** A VM is like a separate **house** with its own walls, plumbing and electricity. A container is like an **apartment** in a shared building: private rooms, but the building's plumbing and electricity are shared.

### 2.1 Virtual Machine (hardware-level virtualization)

A VM emulates a full computer. A **hypervisor** (VMware, KVM, VirtualBox, Hyper-V) splits the physical hardware into virtual CPUs, RAM, disks and network cards. Each VM installs its **own complete guest operating system with its own kernel**.

```text
Application
     ↓
Guest OS (own kernel)
     ↓
Virtual Hardware
     ↓
Hypervisor
     ↓
Host OS
     ↓
Physical Hardware
```

- **Type-1 hypervisor** runs directly on hardware (KVM, ESXi, Hyper-V). Used in data centres.
- **Type-2 hypervisor** runs on top of a normal OS (VirtualBox, VMware Workstation). Used on laptops and desktops.
- Every VM boots a full OS, so it needs more RAM and disk and takes longer to start.
- Strong isolation: a crash or compromise inside one VM rarely affects the host or other VMs.

### 2.2 Container (OS-level virtualization)

A container is a normal Linux **process** that the kernel isolates from other processes. It does **not** boot its own OS. It **shares the host kernel** and only packages the application and its libraries.

```text
Application
     ↓
Container (app + libraries)
     ↓
Container Engine (Docker)
     ↓
Host OS Kernel (shared)
     ↓
Physical Hardware
```

Two Linux kernel features make this possible:

| Feature | What it does |
| --- | --- |
| **Namespaces** | Give each container its own view of processes (`pid`), network (`net`), filesystem (`mnt`), hostname (`uts`), users and IPC. |
| **cgroups** (control groups) | Limit and account for CPU, memory and I/O each container can use (this is what `--cpus=2 --memory=2g` sets). |

### 2.3 Key Terms

| Term | Meaning |
| --- | --- |
| **Hypervisor** | Software that creates and runs VMs. |
| **Guest OS** | The operating system installed inside a VM. |
| **Image** | A read-only template (app + libraries) a container is started from. |
| **Container** | A running instance of an image. |
| **Docker** | The most popular tool for building and running containers. |
| **vCPU** | A virtual CPU core given to a VM. |
| **IOPS** | Input/output operations per second, a disk speed measure. |

### 2.4 Side-by-Side Comparison

| Feature | Virtual Machine | Container |
| --- | --- | --- |
| Virtualization level | Hardware | Operating system |
| Guest OS | Required (full OS) | Not required |
| Kernel | Own guest kernel | Shared host kernel |
| Isolation | Strong (hardware-level) | Lightweight (process-level) |
| Startup time | Tens of seconds to minutes | Milliseconds to seconds |
| Size | GBs | MBs |
| Memory overhead | Higher (OS per VM) | Lower |
| CPU overhead | Slightly higher | Slightly lower |
| Density (instances per host) | Tens | Hundreds |
| Different OS than host | Yes (Windows on Linux, etc.) | No (must use host kernel type) |
| Portability | High | Very high ("build once, run anywhere") |
| Typical use | Full systems, strong security | Microservices, CI/CD, scaling |

### 2.5 Advantages

**Containers**
- Lightweight and very fast to start
- Low memory and disk overhead
- Many containers fit on one host
- Easy to deploy and reproduce (same image everywhere)
- Great fit for microservices and CI/CD

**Virtual Machines**
- Strong isolation and security boundary
- Can run different operating systems on one host
- Full OS control, including the kernel
- Mature tooling and wide enterprise support
- Good for legacy applications

### 2.6 Limitations

- **Containers:** weaker isolation (a kernel bug can affect all containers), and cannot run a different OS kernel.
- **VMs:** heavier, slower to start, and each VM wastes resources on its own OS.

---

## 3. Experimental Setup

Equivalent resources were used for both environments.

| | Virtual Machine | Docker Container |
| --- | --- | --- |
| Technology | VMware / KVM | Docker CE |
| CPU | 2 vCPU | `--cpus=2` |
| Memory | 2 GB | `--memory=2g` |
| OS / Image | Ubuntu 22.04 LTS | `ubuntu:22.04` |
| Storage | SSD/NVMe-backed disk | Docker default storage |

**Host:** Ubuntu 22.04 LTS, Linux kernel 5.15.
**Tool versions:** sysbench 1.0.20, fio 3.28, iperf3 3.9, Python 3.10 with FastAPI and uvicorn.

---

## 4. Execution Steps

Run every test **twice, once in the VM and once in the container**, with the same parameters. Save each output to a file so you can compare later.

### Step 1: Set up the VM

1. Create a VM (VMware or KVM) with **2 vCPU, 2 GB RAM** and install **Ubuntu 22.04**.
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

Or build the provided image instead:

```bash
docker build -t bench ./docker
```

### Step 3: Baseline CPU test (VM and container)

```bash
sysbench cpu --threads=2 --time=30 run
```

### Step 4: CPU scalability

```bash
for t in 1 2 4 8; do
  sysbench cpu --threads=$t --cpu-max-prime=20000 --time=30 run
done
```

Or use the script: `./scripts/run_cpu.sh results/raw/cpu`

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
# Terminal 1: start the server
iperf3 -s

# Terminal 2: start the client
iperf3 -c 127.0.0.1 -t 30      # VM (loopback)
iperf3 -c 172.17.0.1 -t 30     # Container (docker0 bridge)
```

Or: `./scripts/run_network.sh server` and
`./scripts/run_network.sh client 172.17.0.1 results/raw/network/container-network-bridge.txt`

### Step 8: FastAPI application test

1. Start the app (port 8000) in the VM and in a container:

```bash
# VM
cd api && pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8000

# Container
docker build -t fastapi-bench ./api
docker run -p 8000:8000 --cpus=2 --memory=2g fastapi-bench
```

2. Check that it works:

```bash
curl http://localhost:8000/health
```

3. Load test with ApacheBench:

```bash
ab -n 10000 -c 100 http://localhost:8000/health
ab -n 1000  -c 10  http://localhost:8000/compute
ab -n 1000  -c 10  http://localhost:8000/memory
```

### Step 9: Repeat, then analyse

- Repeat each test **2 to 3 times** and close other applications.
- Generate graphs and CSVs, then compare in the terminal:

```bash
python scripts/generate_plots.py
python scripts/analyze_results.py
```

---

## 5. Results

### 5.1 CPU (sysbench, events/sec, higher is better)

| Threads | VM | Container | Better |
| :---: | ---: | ---: | :---: |
| 1 | 515.84 | 517.19 | Container (+0.26%) |
| 2 | 883.55 | 894.38 | Container (+1.23%) |
| 4 | 928.17 | 900.45 | VM (+3.08%) |
| 8 | 905.17 | 914.42 | Container (+1.02%) |

Throughput stops growing after 2 threads because only 2 cores are available. Latency rises with thread count instead (about 1.9 ms at 1 thread to about 8.8 ms at 8 threads).

### 5.2 Memory (sequential write, MiB/s, higher is better)

| Threads | VM | Container |
| :---: | ---: | ---: |
| 1 | **9,541.97** | 5,152.43 |
| 2 | **9,880.38** | 6,970.16 |

### 5.3 Disk I/O (fio)

| Test | VM | Container | Better |
| --- | ---: | ---: | :---: |
| Sequential read (1 MB) | 461 MiB/s | **500 MiB/s** | Container (+8.46%) |
| Sequential write (1 MB) | **358 MiB/s** | 291 MiB/s | VM (+23.02%) |
| Random read (4 KB) | 1,313 IOPS | **1,767 IOPS** | Container (+34.58%) |
| Random write (4 KB) | 1,331 IOPS | **1,346 IOPS** | Container (+1.13%) |

### 5.4 Network (iperf3, 30 s)

| Metric | VM (127.0.0.1) | Container (172.17.0.1) |
| --- | ---: | ---: |
| Sender | 14.1 Gbits/s | 13.7 Gbits/s |
| Receiver | 14.1 Gbits/s | 10.3 Gbits/s |
| Data transferred | 49.3 GB | 47.9 GB |
| TCP retransmissions | 3 | 13 |

### 5.5 FastAPI (ApacheBench, 0 failed requests)

| Endpoint | VM req/s | Container req/s | VM mean latency | Container mean latency |
| --- | ---: | ---: | ---: | ---: |
| `/health` (c=100, n=10,000) | **419.79** | 371.07 | **238.21 ms** | 269.49 ms |
| `/compute` (c=10, n=1,000) | **12.24** | 10.76 | **817.31 ms** | 929.47 ms |
| `/memory` (c=10, n=1,000) | **16.43** | 14.40 | **608.50 ms** | 694.62 ms |

---

## 6. Graphs

### CPU Scalability
<img width="4200" height="1500" alt="image" src="https://github.com/user-attachments/assets/3a8918b4-3a97-4333-b584-daad152b5036" />


### Memory Performance
<img width="3900" height="1500" alt="image" src="https://github.com/user-attachments/assets/a700493a-10ab-4699-901c-9abaa362419e" />


### Disk I/O Performance
<img width="4200" height="1650" alt="image" src="https://github.com/user-attachments/assets/61776710-9af9-4aa3-8575-66752ffc2eea" />


### Network Performance
<img width="3900" height="1500" alt="image" src="https://github.com/user-attachments/assets/17266ab3-f8dd-4033-81a5-270595df184d" />


### FastAPI Performance
<img width="4200" height="1560" alt="image" src="https://github.com/user-attachments/assets/2ea65611-6ec5-4ff1-95ec-e06f658a8c8c" />

### Overall Dashboard
<img width="4800" height="4500" alt="image" src="https://github.com/user-attachments/assets/a5c04d18-6157-4386-9490-5e76abc19029" />

---

## 7. Why the Results Look This Way

- **CPU is almost the same.** A container runs instructions directly on the CPU, and a modern VM uses hardware support (Intel VT-x / AMD-V), so both run near native speed. Differences of a few percent are noise.
- **Disk: containers did better on reads and random I/O.** A container goes through the host filesystem directly, while a VM goes through a virtual disk controller. The VM won on sequential writes in this run.
- **Memory: the VM was faster here.** Containers go through cgroup memory accounting, which can add cost. Treat this as specific to this setup, since memory tests are sensitive to caching and configuration.
- **Network and app: the VM was faster here.** Container traffic passes through a `veth` pair, the `docker0` bridge and `iptables` NAT rules. This costs a small amount per packet, about 13 to 14% in the FastAPI test.

```text
Container network path:
Container socket → veth pair → docker0 bridge → iptables / NAT → host network stack
```

---

## 8. Notes and Limitations

- The network test is **not like-for-like**: the VM used loopback while the container used the Docker bridge.
- Results come from **one machine and a small number of runs**. Treat differences of a few percent as noise.
- Results depend on hardware, hypervisor, container runtime, filesystem, storage type, CPU and memory allocation, and background processes.

Use your own measurements rather than assuming one environment is always faster.

---

## 9. Conclusion

- **CPU:** containers and VMs perform about the same (within about 3%).
- **Disk:** containers did better on reads and random I/O, and the VM did better on sequential writes.
- **Memory, network, app:** the VM was faster in this setup, and the container networking path (`docker0`, `veth`, NAT) explains part of the application gap.
- **Use containers** for microservices, CI/CD, fast scaling and high density.
- **Use VMs** when you need strong isolation, a different OS kernel, or legacy applications.

Containers are lighter and faster to start. VMs are heavier but safer and more flexible. The right choice depends on the workload.

---

## 10. Repository Structure

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
│   ├── run_cpu.sh
│   ├── run_memory.sh
│   ├── run_disk.sh
│   ├── run_network.sh
│   ├── analyze_results.py
│   └── generate_plots.py
├── screenshots/
├── .gitignore
└── README.md
```

---

## 11. Technologies Used

Virtual Machines · Docker · Linux · Sysbench · fio · iperf3 · ApacheBench · FastAPI · Python · Shell scripting
