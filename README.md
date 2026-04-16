# OS_JACKFRUIT-PES1UG24CS700-PES1UG24CS709-
Linux container runtime with namespace isolation, supervisor lifecycle management, IPC-based logging, and kernel module for memory tracking and scheduling experiments


# Container Runtime and Kernel Monitor Project

## 1. Team Information

* Name: Yash Raj
* SRN: PES1UG24CS700
* NAME:-NAVANEETH T 
* SRN: PES1UG24CS709

---

## 2. Build, Load, and Run Instructions

### 🔧 Build the Project

```bash
make
```

---

### 🔌 Load Kernel Module

```bash
sudo insmod monitor.ko
```

Verify:

```bash
ls -l /dev/container_monitor
```

---

### 🚀 Start Supervisor

```bash
sudo ./engine supervisor ./rootfs-base
```

---

### 📁 Create Writable Root Filesystems

```bash
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

---

### ▶️ Start Containers

```bash
sudo ./engine start alpha ./rootfs-alpha /bin/sh --soft-mib 48 --hard-mib 80
sudo ./engine start beta ./rootfs-beta /bin/sh --soft-mib 64 --hard-mib 96
```

---

### 📊 List Containers

```bash
sudo ./engine ps
```

---

### 📜 View Logs

```bash
sudo ./engine logs alpha
```

---

### 🧪 Run Workloads

Copy workload into container rootfs:

```bash
cp mem_test ./rootfs-alpha/
```

Then execute inside container.

---

### ⛔ Stop Containers

```bash
sudo ./engine stop alpha
sudo ./engine stop beta
```

---

### 📥 Unload Kernel Module

```bash
sudo rmmod monitor
```

---

## 3. Demo with Screenshots

### 1. Multi-container supervision

![Multi Container](screenshots/multi_container.png)
*Two containers running under a single supervisor.*

---

### 2. Metadata tracking

![PS Output](screenshots/ps_output.png)
*Output of ps command showing container metadata.*

---

### 3. Bounded-buffer logging

![Logs](screenshots/logs.png)
*Logs captured through producer-consumer pipeline.*

---

### 4. CLI and IPC

![CLI IPC](screenshots/ipc_cli.png)
*CLI interacting with supervisor via IPC.*

---

### 5. Soft-limit warning

![Soft Limit](screenshots/soft_limit.png)
*Kernel logs showing memory soft-limit warning.*

---

### 6. Hard-limit enforcement

![Hard Limit](screenshots/hard_limit.png)
*Container killed after exceeding memory limit.*

---

### 7. Scheduling experiment

![Scheduling](screenshots/scheduling.png)
*Workload behavior under scheduler.*

---

### 8. Clean teardown

![Teardown](screenshots/teardown.png)
*No zombie processes after shutdown.*

---

## 4. Engineering Analysis

### 🔐 Isolation Mechanisms

The runtime achieves isolation using Linux namespaces:

* **PID namespace**: isolates process IDs so containers have their own process trees
* **UTS namespace**: allows separate hostname per container
* **Mount namespace**: isolates filesystem views

`chroot` or `pivot_root` ensures each container has its own root filesystem.

However, all containers still share:

* The same **Linux kernel**
* Kernel resources like scheduler, memory manager

---

### 🔁 Supervisor and Process Lifecycle

A long-running supervisor:

* Creates containers as child processes
* Tracks metadata (PID, state, limits)
* Reaps terminated children to avoid zombies
* Handles signals (start, stop, kill)

This ensures controlled lifecycle management and centralized monitoring.

---

### 🔄 IPC, Threads, and Synchronization

Two IPC mechanisms used:

* CLI ↔️ Supervisor communication (e.g., pipes or sockets)
* Logging system (producer-consumer buffer)

Race conditions:

* Multiple threads accessing shared log buffer
* Metadata updates

Synchronization used:

* **Mutex** → protects shared structures
* **Condition variables** → coordinate producer-consumer
* Prevents data corruption and ensures ordering

---

### 🧠 Memory Management and Enforcement

* **RSS (Resident Set Size)** measures physical memory used
* Does NOT include swapped-out memory

Soft vs Hard limits:

* **Soft limit** → warning threshold
* **Hard limit** → enforced kill

Why kernel enforcement:

* Only kernel has accurate memory tracking
* User-space enforcement can be bypassed

---

### ⚙️ Scheduling Behavior

Observed behavior:

* CPU-bound workloads compete fairly
* Scheduler distributes CPU time across containers
* Interactive tasks remain responsive

This reflects Linux scheduling goals:

* **Fairness**
* **Responsiveness**
* **Throughput**

---

## 5. Design Decisions and Tradeoffs

### Namespace Isolation

* Choice: Linux namespaces
* Tradeoff: Complexity in setup
* Justification: Strong isolation with low overhead

---

### Supervisor Architecture

* Choice: Centralized supervisor
* Tradeoff: Single point of failure
* Justification: Easier lifecycle control

---

### IPC and Logging

* Choice: Bounded buffer with mutex + condition variables
* Tradeoff: Slight overhead
* Justification: Safe and efficient communication

---

### Kernel Monitor

* Choice: Loadable kernel module
* Tradeoff: Requires kernel-level debugging
* Justification: Accurate enforcement of memory limits

---

### Scheduling Experiments

* Choice: Multiple workloads
* Tradeoff: Hard to isolate variables
* Justification: Realistic demonstration of scheduler behavior

---

## 6. Scheduler Experiment Results

| Workload       | Behavior Observed                    |
| -------------- | ------------------------------------ |
| CPU-bound      | Equal CPU sharing                    |
| Memory-heavy   | Slower performance due to contention |
| Mixed workload | Balanced responsiveness              |

Conclusion:

* Linux scheduler ensures fairness
* Performance varies based on workload type
* Resource contention impacts throughput

---

## 7. How to Run CI Build

```bash
make -C boilerplate ci
```

---

## 8. Conclusion

This project demonstrates how operating system principles like isolation, scheduling, memory management, and IPC work together to build a container runtime. It highlights the importance of kernel-space enforcement and structured process management in modern systems.
