# Introduction to UW Research Computing

## Audience
The target audience for this document is UW students who are new to high-performance computing through research projects or lab positions. The document assumes readers have basic coding knowledge but no prior experience with UW's Research Computing resources.

## Overview
This introduction will explain UW's three research computing systems and how to think about data storage for projects.

---

## 1. Understanding UW's Research Computing Systems

High-performance computing (HPC) enables you to solve computational problems that would take days or fail entirely on a personal computer. Unlike the personal laptop, HPC systems are shared resources where multiple researchers run jobs simultaneously. This sharing model requires specialized tools for resource management, but it provides access to computational power beyond personal computers.

The University of Washington provides the following three interconnected systems for research computing.

### 1.1 Hyak

Hyak is the primary supercomputing cluster, consisting of hundreds of powerful computers, or nodes, that function as a unified system. You do not work directly on the compute nodes. Instead, you connect to a login node, which is a gateway computer where you prepare your work. Then, submit jobs that execute on compute nodes when resources become available. This workflow is different from using a personal computer as you request specific resources, including CPU cores, memory, and time, and wait for the scheduler to allocate them.

### 1.2 Tillicum

Tillicum is a specialized cluster optimized for GPU (Graphics Processing Units). GPUs excel at parallel processing tasks that can divide work into many simultaneous tasks. If your research involves machine learning, image processing, or simulations that benefit from parallel computation, using Tillicum is recommended instead of Hyak.

### 1.3 Lolo

Lolo is the archival storage system for long-term data preservation. Unlike Hyak and Tillicum, which are active computing environments, Lolo serves as storage for datasets you need to save but are not currently in use. Lolo ensures data remains accessible for years while allowing free space on the computing systems.

---

## 2. How the Systems Work Together

Research projects typically follow this workflow:

- Active computation occurs on Hyak or Tillicum
- Completed datasets are transferred to Lolo for long-term storage
- Archived data can be retrieved from Lolo when needed

---

## 3. Capabilities and Limitations

### 3.1 What These Systems Can Do

- Execute analyses that would require a long time on a personal computer.
- Access specialized hardware, including GPU and high-memory nodes.
- Process datasets that exceed personal computer storage capacity.
- Run multiple analyses by requesting multiple nodes.

### 3.2 What These Systems Cannot Do

- File sharing services such as Google Drive for collaboration.
- Graphical desktop environments by default.
- Unlimited data storage.

---

## 4. Understanding Storage Tiers

Storage in UW's HPC environment is organized into different tiers that have different performance, capacity, and cost.

### 4.1 Hot Storage

Hot storage is the fastest tier, directly attached to compute nodes on Hyak and Tillicum. This is where your data should be stored during active computation. Hot storage uses high-performance drives that can read and write data fast enough to support computational work. As this storage is expensive and limited, each user receives a quota, which is a maximum storage capacity. Hot storage is not backed up, as it is designed for temporary data storage.

### 4.2 Cold Storage

Cold storage is the archival system designed for long-term storage of data, such as Lolo. Retrieving data from Lolo may take several hours, but it can preserve large amounts of data effectively at a low cost. Cold storage is backed up and designed for datasets that must be retained for years. Completed datasets, raw experimental data, and published results should be stored on Lolo when active analysis is done.

### 4.3 Data Lifecycle

Most research data follows the following lifecycle:

1. New data is uploaded to hot storage for initial processing.
2. Active analysis with Hyak or Tillicum in hot storage.
3. Completed analysis and final datasets transfer to Lolo.
4. Retrieved from Lolo back to hot storage when needed.

---

## 5. Key Concepts for Shared Computing

### 5.1 Job Scheduling with Slurm

As the systems are shared, you cannot access resources immediately. Instead, you use Slurm, a job scheduler, to request specific resources, including the number of CPUs, the amount of memory, and time. Slurm places your request in a queue and allocates resources based on availability and your account's priority.

### 5.2 Batch and Interactive Jobs

Batch jobs are scripts you submit to run without direct interaction. They are appropriate for long computations. On the other hand, interactive sessions provide real-time access to a compute node, similar to working on a personal computer. They are useful for code development, debugging, and analysis.

---

## 6. Requirements to Get Started

- An active UW NetID
- Duo Authentication enabled on the UW NetID
- VPN access configured for off-campus connections
- Approved account on the specific system you will use
