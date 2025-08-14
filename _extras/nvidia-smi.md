---
title: "Monitoring GPU performance with nvidia-smi"
---

## Understanding Outputs - `nvidia-smi` GPU Monitoring

### Example `nvidia-smi` Output:

```
------ Wed Jul  2 17:12:23 IST 2025 ------
Wed Jul  2 17:12:23 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 560.35.05              Driver Version: 560.35.05      CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------|
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA H100 NVL                On  |   00000000:AB:00.0 Off |                    0 |
| N/A   37C    P0             86W /  400W |    1294MiB /  95830MiB |      0%      Default |
|                                         |                        |             Disabled |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A   2234986      C   python                                       1284MiB |
+-----------------------------------------------------------------------------------------+
```

...

### Explanation of `nvidia-smi` Output:

#### GPU Summary Header

* **NVIDIA-SMI Version:** 560.35.05 — Monitoring tool version.
* **Driver Version:** 560.35.05 — NVIDIA driver version installed.
* **CUDA Version:** 12.6 — CUDA toolkit compatibility version.

#### GPU Info Section

| Field                    | Meaning                                      |
| ------------------------ | -------------------------------------------- |
| **GPU**                  | GPU index number (0)                         |
| **Name**                 | GPU model: NVIDIA H100 NVL                   |
| **Persistence-M**        | Persistence Mode: On (reduces init overhead) |
| **Bus-Id**               | PCI bus ID location                          |
| **Disp.A**               | Display Active: Off (no display connected)   |
| **Volatile Uncorr. ECC** | GPU memory error count (0 = no errors)       |
| **Fan**                  | Fan speed (N/A — passive cooling)            |
| **Temp**                 | Temperature (37C — healthy)                  |
| **Perf**                 | Performance state (P0 = maximum performance) |
| **Pwr\:Usage/Cap**       | Power usage (86W of 400W max)                |
| **Memory-Usage**         | 1294MiB used / 95830MiB total                |
| **GPU-Util**             | GPU utilization (0% — idle)                  |
| **Compute M.**           | Compute mode (Default)                       |
| **MIG M.**               | Multi-Instance GPU mode (Disabled)           |

#### Processes Section

| Field            | Meaning                      |
| ---------------- | ---------------------------- |
| **GPU**          | GPU ID (0)                   |
| **PID**          | Process ID (2234986)         |
| **Type**         | Type of process: C (compute) |
| **Process Name** | Process name (python)        |
| **GPU Memory**   | 1284MiB used by this process |

* These explanations cover the descriptions of each of the different parameters given by the `nvidia-smi` output. 

---


{% include links.md %}
