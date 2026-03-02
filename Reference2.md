# Reference: Managing Jobs and Data on Hyak, Tillicum, and Lolo

This document is a quick-reference companion to [Managing Research Data and Jobs](https://github.com/taeyunkim03/ENGL396_Software_Documentation.github.io/blob/main/TaskDoc2.md).

---

## Storage Tiers

| Location | System | Path | Quota | Retention | Best Used For |
|---|---|---|---|---|---|
| Home | Hyak | `/mmfs1/home/YourUWNetID` | 10 GB | Permanent | Config files, small scripts |
| Lab scratch | Hyak | `/mmfs1/gscratch/YourLabName` | Varies | Not backed up | Active computation |
| Home | Tillicum | `/gpfs/home/YourUWNetID` | 10 GB | Permanent | Config files, small scripts |
| Lab projects | Tillicum | `/gpfs/projects/YourLabName` | 1 TB | Permanent | Shared project data |
| Scratch | Tillicum | `/gpfs/scrubbed/YourUWNetID` | 100 TB | Purged after 60 days | Temporary outputs |
| Archive | Lolo | `/archive/YourLabName` | Varies | Backed up, permanent | Completed projects |

---

## Compute Costs

### Tillicum

Tillicum charges **$0.90 per GPU per hour**, billed monthly via UW-IT ITBill. Every job must request at least 1 GPU — CPU-only jobs are not permitted. Billing is rounded down to the nearest GPU-hour based on actual usage.

**Formula:** `Number of GPUs × Hours used × $0.90`

| GPUs | Duration | Estimated Cost |
|---|---|---|
| 1 | 30 min (debug) | $0.45 |
| 1 | 2 hours | $1.80 |
| 2 | 8 hours (interactive) | $14.40 |
| 4 | 12 hours | $43.20 |
| 8 | 24 hours (normal) | $172.80 |

Slurm shows an estimated cost when you submit a job:

```
salloc: Req GPUs: 1
salloc: Req Time: 2.00 hrs
salloc: YOUR COST: $1.80 (Est.)
```

**QoS Levels:**

| QoS | Max Wall Time | Max GPUs | Use Case |
|---|---|---|---|
| `debug` | 30 minutes | 1 | Testing and debugging |
| `interactive` | 8 hours | 2 | Active development |
| `normal` | 24 hours | 16 | Production batch jobs |

**Monitor budget:**
```bash
hyakusage -u YourUWNetID    # Your usage
hyakusage -u all            # Lab-wide usage
seff JOBID                  # Efficiency report for a completed job
```

### Hyak

Hyak uses a **condo model** — labs purchase compute slices and pay annual slot fees. There is no per-job or per-hour charge.

| Slice Type | Specs | Annual Slot Fee (sponsored) |
|---|---|---|
| HPC (CPU) | 32 cores, 256 GB RAM | ~$800/slot/year |
| GPU (L40S) | 32 cores, 384 GB, 2× L40S GPUs | ~$1,200/slot/year |
| GPU (H200) | Large GPU slice | ~$1,600/slot/year |

Hardware prices change frequently. For current quotes, email help@uw.edu.

---

## Troubleshooting

### Batch Jobs

**Problem:** Job pending for hours.  
**Solution:** Check cluster load with `sinfo` and remaining allocation with `hyakalloc` (Hyak) or `hyakusage -u all` (Tillicum). Reduce resource requests, or submit during off-peak hours. On Tillicum, verify the lab budget has not been exhausted.

**Problem:** Job fails with "out of memory."  
**Solution:** Run `seff JOBID` on a completed job to see actual memory usage. Increase `--mem` by 20–30% above peak usage and resubmit.

**Problem:** Job fails immediately with no useful error.  
**Solution:** Check `job_JOBID.err`. Common causes: missing `module load` or `conda activate`, incorrect input file paths, or a misspelled environment name (`conda env list` to verify).

**Problem:** Job array tasks failing inconsistently.  
**Solution:** Check individual task error files (`array_JOBID_TASKID.err`). Verify input files exist for all array indices. Ensure tasks write to separate output files using `$SLURM_ARRAY_TASK_ID` in filenames.


---

### Storage and Quotas

**Problem:** Out of storage quota.  
**Solution:** Identify large files with `du -sh * | sort -h`. Delete intermediate files that can be regenerated, compress completed datasets (`tar -czf`), and archive finished projects to Lolo.

**Problem:** Files disappeared from scratch.  
**Solution:** Hyak gscratch and Tillicum `/gpfs/scrubbed` do not back up files and purge data that has not been accessed recently (60 days on Tillicum). Move results to a permanent directory before the purge window closes.

**Problem:** Can't find archived data on Lolo.  
**Solution:** Check README files in `/archive/YourLabName/`. To inspect an archive without extracting: `tar -tzf archive.tar.gz | head -50`. Note that Lolo tape retrieval can take several hours.

---

### Data Transfer

**Problem:** `scp` transfer stalled or interrupted.  
**Solution:** Switch to `rsync`, which resumes automatically on rerun:
```bash
rsync -avz --progress source/ YourUWNetID@klone.hyak.uw.edu:/destination/
```

**Problem:** Transfer too slow for a very large dataset (multi-TB).  
**Solution:** Contact UW Research Computing about Globus for optimized large-scale transfers.

---

### Billing (Tillicum)

**Problem:** Unexpected charge on ITBill.  
**Solution:** Tillicum bills for all scheduled jobs, including jobs that fail or are cancelled mid-run. Audit usage with `hyakusage -u all`. Cancel unneeded jobs immediately with `scancel JOBID`.

**Problem:** Lab member left a job running unintentionally.  
**Solution:** Cancel it with `scancel JOBID`. Monitor all lab jobs with `squeue -A YourLabName`. Set `--time` limits in all job scripts to cap runaway jobs.

---

Email **help@uw.edu** with "Hyak" in the subject for additional questions or unsolved problems.
