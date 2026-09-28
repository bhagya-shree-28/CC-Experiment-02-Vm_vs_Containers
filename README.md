# VM vs Container Performance Analysis

## 1. Project Overview

This project evaluates the performance differences between **Virtual Machines (VMs)** and **Containers** using three major system performance metrics:

* CPU Performance
* Memory Performance
* Disk I/O Performance

The objective is to understand the performance overhead introduced by virtualization and compare it with the lightweight virtualization approach used by containers.

---

## 2. Objective

The main objectives of this experiment are:

1. Compare CPU performance between a VM and a container.
2. Compare memory performance between a VM and a container.
3. Compare disk I/O performance between a VM and a container.
4. Measure the overhead introduced by each virtualization approach.
5. Determine how closely containers perform compared with the underlying host system.

---

## 3. VM vs Container

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

* Execution time
* Events per second
* CPU utilization
* CPU latency

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
* CPU: [Specify CPU allocation]
* Memory: [Specify RAM]
* Storage: [Specify disk configuration]
* Operating System: [Specify OS]
* Hypervisor: [Specify hypervisor]

### Container

* Container Technology: Docker
* CPU Limit: [Specify CPU limit]
* Memory Limit: [Specify memory limit]
* Storage: [Specify storage configuration]
* Base Image: [Specify image]

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

### Step 1 — Configure the VM

Create and configure the VM with a fixed number of CPU cores, memory, and storage.

### Step 2 — Configure the Container

Create a container with equivalent CPU and memory limits.

### Step 3 — Run CPU Benchmark

Execute the CPU benchmark multiple times and record:

* Execution time
* Events per second
* CPU utilization

### Step 4 — Run Memory Benchmark

Execute the memory benchmark and record:

* Memory throughput
* Execution time
* Memory bandwidth

### Step 5 — Run Disk I/O Benchmark

Execute sequential and random read/write tests and record:

* Throughput
* IOPS
* Latency

### Step 6 — Repeat the Experiments

Each benchmark should preferably be repeated multiple times to reduce the effect of temporary system variations.

---

## 8. Results

The results can be recorded using the following format.

### CPU Performance

| Environment | Execution Time | Events/sec |
| ----------- | -------------: | ---------: |
| VM          |              — |          — |
| Container   |              — |          — |

### Memory Performance

| Environment | Memory Throughput | Latency |
| ----------- | ----------------: | ------: |
| VM          |                 — |       — |
| Container   |                 — |       — |

### Disk I/O Performance

| Environment | Read Throughput | Write Throughput | IOPS |
| ----------- | --------------: | ---------------: | ---: |
| VM          |               — |                — |    — |
| Container   |               — |                — |    — |

---

## 9. Performance Analysis

The results are analyzed based on:

### CPU

Containers are expected to have CPU performance close to the host because they share the host kernel and do not require a complete guest operating system.

VMs introduce additional virtualization overhead through virtual hardware and the hypervisor.

### Memory

Containers generally require less memory because they do not need a separate guest operating system.

A VM requires memory for both the applications and the guest operating system.

### Disk I/O

Disk performance depends strongly on the storage driver, filesystem, caching, and VM virtual disk configuration.

Containers can achieve performance close to the host when using an appropriate storage configuration. VM storage performance can be affected by additional virtualization and virtual-disk layers.

---

## 10. Expected Outcome

In a controlled experiment, containers are generally expected to show lower overhead and performance closer to the host system.

The expected general relationship is:

```text
Performance

Native Host
     │
     ├── Container
     │
     ├── Type-1 VM
     │
     └── Type-2 VM
```

However, the actual results depend on:

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

This experiment compares VM and container performance using CPU, memory, and disk I/O benchmarks.

Containers generally introduce less resource overhead because they share the host operating system kernel. VMs provide stronger isolation and complete operating-system environments but require additional resources for the guest OS and virtual hardware.

The benchmark results provide a quantitative basis for understanding the performance trade-offs between the two virtualization approaches.

---

## 15. Repository Structure

```text
VM-vs-Container-Performance/
│
├── README.md
│
├── benchmarks/
│   ├── cpu/
│   ├── memory/
│   └── disk/
│
├── results/
│   ├── vm/
│   ├── container/
│   └── comparison/
│
├── scripts/
│   ├── cpu_benchmark.sh
│   ├── memory_benchmark.sh
│   └── disk_benchmark.sh
│
├── graphs/
│   ├── cpu_comparison.png
│   ├── memory_comparison.png
│   └── disk_io_comparison.png
│
└── screenshots/
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

