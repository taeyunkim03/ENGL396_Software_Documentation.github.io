# Managing UW Research Computing Resources

## Audience
The target audience for this document is academic researchers and principal investigators managing projects with specialized computational requirements such as genomic sequencing or machine learning. The document assumes readers have technical skills and basic knowledge of UW's research computing resources.

## Overview
This introduction provides an overview of UW's computing infrastructure and guidance on selecting and optimizing resources for specialized research projects.

---

## 1. The UW HPC System

UW's research computing environment consists of three specialized systems optimized for different computational profiles.

### 1.1 Hyak

Hyak provides general purpose CPU computing suitable for a broad range of research applications. Each research group can purchase dedicated nodes while contributing to a shared pool. Hyak supports workflows requiring sustained CPU resources, large memory needs, or specific hardware configurations. It is appropriate for genomic assembly, statistical modeling, molecular simulations, and other CPU-intensive tasks.

### 1.2 Tillicum

Tillicum specializes in GPU computing for work that benefits from running various calculations simultaneously. Modern GPUs provide significant speedups for tasks such as deep learning model training, protein structure prediction, and image analysis. Tillicum's hardware works well with machine learning frameworks such as TensorFlow and PyTorch, as well as applications requiring high-throughput parallel computation.

### 1.3 Lolo

Lolo provides long-term storage designed for data preservation. The system balances cost with data security, making it appropriate for preserving raw data, published datasets, and analysis environments beyond the active project.

---

## 2. Storage Architecture

Understanding storage options is important for optimizing performance and costs in computational research.

### 2.1 Hot Storage (Hyak and Tillicum)

Hot storage provides fast access for active computation. This storage uses SSDs directly connected to compute nodes, delivering the speed required for intensive work. Hot storage is not backed up and comes with quotas. The system assumes data in hot storage is either temporary or exists in another permanent location.

### 2.2 Cold Storage (Lolo)

Cold storage prioritizes capacity and data preservation over access speed. Retrieval may take hours to days, depending on system load, making this storage unsuitable for active computation. It is more ideal for raw experimental data, published datasets, completed analysis outputs, and final results. Cold storage is backed up.

### 2.3 Data Lifecycle

1. New data enters hot storage for quality control and initial processing.
2. Data remains in hot storage during development and computation.
3. Completed datasets and final results transferred to Lolo.
4. Archived data returns to hot storage when needed for additional analysis.

---

## 3. Resource Selection

### 3.1 Hyak Usage

- When research primarily uses CPU processing.
- Tasks require a large amount of memory.
- Applications are not designed for GPU use.
- Workflows involve many independent jobs.

### 3.2 Tillicum Usage

- Algorithms can split work across cores, such as deep learning or simulations.
- Applications use GPU-optimized libraries such as CUDA, TensorFlow, or PyTorch.
- Work involves matrix operations or convolutions.
- Cost per computation favors GPU over extended CPU time.

---

## 4. Resource Allocation

### 4.1 Slurm Job Scheduling

All systems use Slurm for resource allocation. Slurm implements fair scheduling based on requested resources and account priority.

### 4.2 Interactive and Batch Computing

Batch jobs are appropriate for time-consuming computations or jobs that can run without supervision. Interactive sessions are appropriate for:

- Code development and debugging
- Exploratory data analysis
- Workflows requiring immediate adjustments
- Software testing with performance evaluation

Interactive sessions consume resources while idle, so close them when not actively in use.
