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

For the following experiments, I am using a host that has 4 A100 GPUs connected to it. In pairs they are connected through NVLink for a total of 2 NVLink connections. You can create a similar setup in AWS by provisioning the `p4d` class machines. They will provision 8 A100 GPUs and the results might vary from what you see here. But the analysis remains the same.

As always, you might have to request quota in order to provision these GPUs and for that you'll have to submit a ticket. Also, A100 is expensive and please pay attention to the price per hour before provisioning them in order to avoid a sticker shock 😆

### NVLink and What is it?

Nvidia describes NVLink as the following on their website -

```
NVLink is a 1.8TB/s bidirectional, direct GPU-to-GPU interconnect that scales multi-GPU input and output (IO) within a server. The NVIDIA NVLink Switch chips connect multiple NVLinks to provide all-to-all GPU communication at full NVLink speed within a single rack and between racks. 
```

To unpack it, GPUs connected through NVLink get almost 1.8TB/s of bandwidth. It's a separate piece of hardware that connects two GPUs directly and bypasses the whole PCI express slot and host thereby avoiding any latency there. As yo will see the later sections, this allows two GPUs to talk to each other as if they are almost one GPU which is great from data transfer standpoint.

### Topology Information from nvidia-smi

We can get the topology information of our host by running `nvidia-smi` with `matrix` set as flag.

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

Looking at the matrix, we see that `GPU0` and `GPU1` are connected to each other using NVLink. Similarly `GPU2` and `GPU3` are connected to each other using NVLink. If any non-NVLink-ed GPUs want to talk to each other, they will have to do so via PCI-e slot. I will do a future post talking more about this as I understand it better. Right now, it is suffient to understand that data transfer across two GPUs which are not connected through NVLink will be much slower. To see this we can go to the next section.

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

This matrix shows us things much more clearly. When P2P is diabled, no NVLink is used and therefore every communication across GPUs will be through PCIexpress slot and it will be slow. The magic happens for the cases when P2P is enabled.

We observe the following things -

1. If two GPUs are connected using NVLink, their bandwidth is 10x as compared to communicating across GPUs not connected through NVLink. This becomes 20x. i.e., it doubles when we are checking bidirectional data transfer.
2. The latency to transfer data from a GPU to itself about 2.62 us. The same amount of data will almost 19 us to transfer to another GPU that is not NVLink-ed. If we have NVLink, it drops to 2.75 us. Almost close to GPU transferring it to itself. This shows that NVLink is like almost bonding the two GPUs in terms of data bandwidth.
3. Lastly, we see CPU jump up when talking about transfers across GPUs that are not NVLink-ed. This is expected considering that data is being transferred through PCIe slot and the CPU will have to step in to do some synchronization and housekeeping.

### Conclusion

Talk about the broad points. Maybe also throw in a line around orchestrators like SLURM - a workload orchestrator. They are more less similar to Kubernetes but more popular in the HPC world.
