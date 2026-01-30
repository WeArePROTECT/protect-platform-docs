# 🖥️ Arkin Lab Server Etiquette & Resource Monitoring Guide

**Contact:** spencerlong@berkeley.edu  
**Last Updated:** October 2025

---

## 🌿 Purpose

Our shared servers are designed for collaborative data processing and analysis — not for high-throughput or large-scale compute jobs. Please follow these guidelines to ensure fair usage and stable performance for everyone.

---

## ⚖️ Usage Guidelines

| Job Type | CPU Limit | Memory Limit | Notes |
|----------|-----------|--------------|-------|
| Short jobs (≤ few hours) | ≤ 75% of available cores | ≤ 75% of available RAM | e.g., small preprocessing, data exploration |
| Long jobs (> few hours) | ≤ 50% of available cores | ≤ 50% of available RAM | e.g., genome assembly, modeling, heavy training |

If you expect to exceed these limits, please post in the `#protect-compute` Slack channel (or equivalent) with:
- Job name / command
- Estimated duration
- Expected CPU and memory usage

If your work routinely exceeds these limits, please request access to a compute cluster such as NERSC or Lawrencium.

---

## 🧠 Quick Checks Before & During a Job

### 🔹 1. Check CPU Usage

```bash
top -u $USER
```

or

```bash
ps -u $USER -o pid,ppid,%cpu,%mem,cmd --sort=-%cpu | head -n 10
```

**For a 32-core server:**
- Short job → use ≤ 24 cores
- Long job → use ≤ 16 cores

**Interactive option:**
```bash
htop
```
(Press `1` to expand all cores.)

### 🔹 2. Check Memory Usage

```bash
free -h
```

Displays total, used, and available RAM.

**Per-process detail:**
```bash
ps -u $USER -o pid,%mem,cmd --sort=-%mem | head
```

If your job uses > 75% of total RAM, stop or downscale to avoid Out-of-Memory errors.

### 🔹 3. Check System Load

```bash
uptime
```

The load average should remain below the number of physical cores (e.g., ≤ 32 on a 32-core machine).

### 🔹 4. Check Disk I/O (for large datasets)

```bash
iotop -u $USER
```

or system-wide:

```bash
iostat -xz 5
```

High `%iowait` values → disk contention. If this occurs, contact Keith to create a local scratch directory.

### 🔹 5. Monitor Long Jobs Over Time

```bash
watch -n 30 "ps -u $USER -o pid,etime,%cpu,%mem,cmd --sort=-%cpu | head"
```

Updates every 30 seconds with live resource usage.

---

## 💾 Best Practices

- ✅ Use local directories for heavy I/O work (ask Keith for setup).
- ✅ Use rsync to move large data instead of working directly off the network filesystem.
- ✅ Test on small subsets before running full datasets.
- ✅ Be considerate — others share the server!

---

## 🧩 Example Outputs and How to Read Them

### 🔸 `top -u $USER` (sample output)

```
%Cpu(s): 75.0 us,  2.0 sy,  0.0 ni, 23.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem : 128000.0 total, 64000.0 free, 48000.0 used, 16000.0 buff/cache
  PID USER   PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
12934 s.long  20   0  23.6g   8.2g  3540 R  7500  6.4   2:45:18 python3
```

**Interpretation:**
- `%CPU` = total across all cores. Here 75 × 100% = ~24 cores used on a 32-core server → OK for short jobs.
- `%MEM` = 6.4% of 128 GB ≈ 8 GB RAM — well within limits.

### 🔸 `free -h` (sample output)

```
             total        used        free      shared  buff/cache   available
Mem:           125G         45G         60G         2G         20G         75G
Swap:            0B          0B          0B
```

**Interpretation:**
- System is ~36% used → safe zone.
- If "used" > 80% or "available" < 15%, pause large jobs.

### 🔸 `uptime` (sample output)

```
14:20:06 up 12 days,  3:18,  3 users,  load average: 10.45, 8.72, 7.95
```

**Interpretation:**
- On a 32-core machine, load ≈ 10 → fine.
- If load average > 32 for long periods → system overloaded.

### 🔸 `iostat -xz 5` (sample output)

```
Device:         r/s     w/s  rkB/s  wkB/s  await  svctm  %util
sda            32.0    4.0 20480.0 1024.0    1.5    0.5   12.0
```

**Interpretation:**
- `%util` < 70% = OK.
- If `%util` or `await` values rise > 80% for long periods, disk is bottlenecked — move data to local scratch.

---

## 🆘 When to Ask for Help

If you notice the system running slowly or aren't sure your job is within limits, contact:

- **Keith Keller** – System Administrator
- **Spencer Long** – Data Management & Server Integration

They can verify server load, help allocate storage, or recommend cluster resources.
