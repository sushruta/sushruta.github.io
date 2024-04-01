---
title: "NVLink and GPU Interconnect"
description: Looking into what is NVLink and getting multiple GPUs to communicate with each other.
slug: nvlink-multiple-gpus
date: 2024-03-30T16:17:13-07:00
categories:
  - nvlink
  - gpu
  - bandwidth
tags:
  - gpu
  - bandwidth
  - nvlink
---

Topics to cover --

### Getting Multiple GPUs with an Interconnect

0. To set the stage, let us talk about a system where multiple GPUs are connected to each other through fast interconnect. A few words on how to get that on AWS. Once we get it, do `nvidia-smi` to show that we infact have multiple GPUs running.

### NVLink and What is it?

A few lines on what is NVLink. A diagram would be great as well

### Topology Information from nvidia-smi

Now introduce topology part of `nvidia-smi` with `matrix` set as flag.

```
nvidia-smi topo --matrix
```

An example output is the following --

```
	GPU0	GPU1	GPU2	GPU3	CPU Affinity	NUMA Affinity	GPU NUMA ID
GPU0	 X 	NV12	SYS	SYS	0-23	0		N/A
GPU1	NV12	 X 	SYS	SYS	24-47	1		N/A
GPU2	SYS	SYS	 X 	NV12	48-71	2		N/A
GPU3	SYS	SYS	NV12	 X 	72-95	3		N/A

Legend:

  X    = Self
  SYS  = Connection traversing PCIe as well as the SMP interconnect between NUMA nodes (e.g., QPI/UPI)
  NODE = Connection traversing PCIe as well as the interconnect between PCIe Host Bridges within a NUMA node
  PHB  = Connection traversing PCIe as well as a PCIe Host Bridge (typically the CPU)
  PXB  = Connection traversing multiple PCIe bridges (without traversing the PCIe Host Bridge)
  PIX  = Connection traversing at most a single PCIe bridge
  NV#  = Connection traversing a bonded set of # NVLinks
```

Write a few lines describing what these things mean and what information we can glean.

### Running the P2P Bandwidth Test

Talk about the sample executable available in CUDA samples and run it and provide some information on what is the output telling us.

The output we got was -

```
[P2P (Peer-to-Peer) GPU Bandwidth Latency Test]
Device: 0, NVIDIA A100 80GB PCIe, pciBusID: 0, pciDeviceID: 0, pciDomainID:1
Device: 1, NVIDIA A100 80GB PCIe, pciBusID: 0, pciDeviceID: 0, pciDomainID:2
Device: 2, NVIDIA A100 80GB PCIe, pciBusID: 0, pciDeviceID: 0, pciDomainID:3
Device: 3, NVIDIA A100 80GB PCIe, pciBusID: 0, pciDeviceID: 0, pciDomainID:4
Device=0 CAN Access Peer Device=1
Device=0 CANNOT Access Peer Device=2
Device=0 CANNOT Access Peer Device=3
Device=1 CAN Access Peer Device=0
Device=1 CANNOT Access Peer Device=2
Device=1 CANNOT Access Peer Device=3
Device=2 CANNOT Access Peer Device=0
Device=2 CANNOT Access Peer Device=1
Device=2 CAN Access Peer Device=3
Device=3 CANNOT Access Peer Device=0
Device=3 CANNOT Access Peer Device=1
Device=3 CAN Access Peer Device=2

***NOTE: In case a device doesn't have P2P access to other one, it falls back to normal memcopy procedure.
So you can see lesser Bandwidth (GB/s) and unstable Latency (us) in those cases.

P2P Connectivity Matrix
     D\D     0     1     2     3
     0	     1     1     0     0
     1	     1     1     0     0
     2	     0     0     1     1
     3	     0     0     1     1
Unidirectional P2P=Disabled Bandwidth Matrix (GB/s)
   D\D     0      1      2      3 
     0 1508.20  21.51  21.63  21.80 
     1  21.40 1509.66  21.68  21.59 
     2  21.48  21.55 1509.66  21.60 
     3  21.49  21.66  21.60 1511.12 
Unidirectional P2P=Enabled Bandwidth (P2P Writes) Matrix (GB/s)
   D\D     0      1      2      3 
     0 1502.40 275.07  21.62  21.60 
     1 275.51 1521.42  21.69  21.50 
     2  21.46  21.72 1508.20 275.34 
     3  21.51  21.69 274.11 1521.42 
Bidirectional P2P=Disabled Bandwidth Matrix (GB/s)
   D\D     0      1      2      3 
     0 1525.88  26.41  29.57  29.35 
     1  26.00 1531.86  29.68  29.49 
     2  27.41  27.69 1528.12  28.93 
     3  26.85  27.22  28.99 1529.61 
Bidirectional P2P=Enabled Bandwidth Matrix (GB/s)
   D\D     0      1      2      3 
     0 1531.11 516.19  29.58  29.35 
     1 515.51 1527.37  29.79  29.42 
     2  27.36  27.80 1528.86 517.45 
     3  27.07  27.19 515.35 1523.65 
P2P=Disabled Latency Matrix (us)
   GPU     0      1      2      3 
     0   2.61  21.89  20.48  17.81 
     1  11.68   2.72  14.22  20.16 
     2  18.63  15.70   2.48  13.97 
     3  18.73  18.53  18.04   2.49 

   CPU     0      1      2      3 
     0   2.65   7.81   6.98   6.28 
     1   7.10   2.55   6.22   6.19 
     2   7.20   6.47   2.29   5.74 
     3   6.64   6.49   5.61   2.30 
P2P=Enabled Latency (P2P Writes) Matrix (us)
   GPU     0      1      2      3 
     0   2.62   2.25  19.85  16.96 
     1   2.22   2.71  14.12  20.38 
     2  18.87  16.03   2.47   2.32 
     3  18.78  18.08   2.32   2.48 

   CPU     0      1      2      3 
     0   2.34   1.92   6.34   6.43 
     1   1.96   2.32   6.34   6.56 
     2   6.79   7.20   1.99   1.63 
     3   6.67   6.55   1.68   2.07 

NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.
```

### Conclusion

Talk about the broad points. Maybe also throw in a line around orchestrators like SLURM - a workload orchestrator. They are more less similar to Kubernetes but more popular in the HPC world.
