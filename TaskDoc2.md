# Managing Research Data and Jobs on UW HPC Systems

## Audience
This guide is for academic researchers and principal investigators managing computational projects on Hyak and Tillicum with data archiving needs on Lolo.

## What You Will Learn
How to organize data across storage tiers, submit batch jobs, transfer data efficiently, and archive to Lolo.

## Prerequisites
- Active accounts on Hyak, Tillicum, and/or Lolo
- SSH access configured
- Understanding of your project's computational requirements

## Estimated Time
20-30 minutes to learn core workflows

---

## Understanding Storage Architecture

**Hyak**:
- Home: `/mmfs1/home/YourUWNetID` (10 GB)
- Lab scratch: `/mmfs1/gscratch/YourLabName` (variable quota)

**Tillicum**:
- Home: `/gpfs/home/YourUWNetID` (10 GB)
- Lab projects: `/gpfs/projects/YourLabName` (1 TB)
- Scratch: `/gpfs/scrubbed` (100 TB, purged after 60 days)

**Lolo**:
- Lab archive: `/archive/YourLabName` (backed up, slow retrieval)

---

## Task 1: Checking Storage Usage

Monitor storage to avoid quota issues.

### Steps

1. On Hyak, check lab allocation.
   ```bash
   ssh YourUWNetID@klone.hyak.uw.edu
   hyakalloc
   ```

**Result**: Shows storage quotas and current usage.

2. Check directory sizes.
   ```bash
   cd /mmfs1/gscratch/YourLabName
   du -h --max-depth=1
   ```

**Result**: Shows usage breakdown by subdirectory.

3. On Lolo, view usage report.
   ```bash
   ssh YourUWNetID@lolo.uw.edu
   cd /archive/YourLabName
   cat usage_report.txt
   ```

---

## Task 2: Submitting Batch Jobs

Run long computations without manual supervision.

### Steps

1. Create a job script.
   ```bash
   nano my_job.sh
   ```

2. Write the script content:
   
   ```bash
   #!/bin/bash
   #SBATCH --job-name=analysis
   #SBATCH --account=YourLabName
   #SBATCH --partition=compute
   #SBATCH --nodes=1
   #SBATCH --cpus-per-task=16
   #SBATCH --mem=128G
   #SBATCH --time=12:00:00
   #SBATCH --output=job_%j.out
   #SBATCH --error=job_%j.err
   
   module load python/3.12
   conda activate myproject
   python analysis_script.py
   echo "Job completed"
   ```

3. Save and exit (Ctrl+O, Enter, Ctrl+X).

4. Submit the job.
   ```bash
   sbatch my_job.sh
   ```

**Result**: Returns job ID number.

5. Monitor job status.
   ```bash
   squeue -u YourUWNetID
   ```

**Result**: Shows job state (PD=pending, R=running, CG=completing).

6. Check output after completion.
   ```bash
   less job_JOBID.out
   ```

7. To cancel a running job:
   ```bash
   scancel JOBID
   ```

---

## Task 3: Creating Job Arrays

Run multiple similar tasks efficiently.

### Steps

1. Create array job script.
   ```bash
   nano array_job.sh
   ```

2. Add array directive:
   
   ```bash
   #!/bin/bash
   #SBATCH --job-name=array_analysis
   #SBATCH --account=YourLabName
   #SBATCH --partition=compute
   #SBATCH --cpus-per-task=8
   #SBATCH --mem=64G
   #SBATCH --time=04:00:00
   #SBATCH --array=1-10
   #SBATCH --output=array_%A_%a.out
   #SBATCH --error=array_%A_%a.err
   
   echo "Processing task ${SLURM_ARRAY_TASK_ID}"
   python script.py --input data_${SLURM_ARRAY_TASK_ID}.csv
   ```

3. Submit:
   ```bash
   sbatch array_job.sh
   ```

**Result**: Creates 10 jobs numbered 1-10, each processing a different input file.

**Array Notes**:
- `%A` is the master job array ID
- `%a` is the individual task ID
- `SLURM_ARRAY_TASK_ID` variable contains the current task number
- Useful for parameter sweeps or processing multiple datasets

---

## Task 4: Transferring Data Between Systems

### For Small Files (< 1 GB)

```bash
scp local_file.txt YourUWNetID@klone.hyak.uw.edu:/mmfs1/gscratch/YourLabName/
```

### For Large Files (> 1 GB)

Use rsync for reliability:
```bash
rsync -avz --progress local_data/ YourUWNetID@klone.hyak.uw.edu:/mmfs1/gscratch/YourLabName/data/
```

**Options**:
- `-a`: Preserve permissions and timestamps
- `-v`: Show progress
- `-z`: Compress during transfer
- `--progress`: Display progress bar

If transfer is interrupted, rerun the same rsync command to resume automatically.

### Between Hyak and Tillicum

```bash
rsync -avz /mmfs1/gscratch/YourLabName/data/ YourUWNetID@tillicum.hyak.uw.edu:/gpfs/projects/YourLabName/data/
```

### For Very Large Datasets (Multi-TB)

Contact UW Research Computing about using Globus for optimal transfer speeds and reliability.

---

## Task 5: Archiving Data to Lolo

Move completed projects to long-term storage.

### Steps

1. Compress data on Hyak or Tillicum.
   ```bash
   tar -czf project_final.tar.gz /path/to/project/data/
   ```

   **Command breakdown**:
   - `tar`: Archive utility
   - `-c`: Create archive
   - `-z`: Compress with gzip
   - `-f`: Specify filename

2. Verify archive created successfully.
   ```bash
   ls -lh project_final.tar.gz
   ```

3. Transfer to Lolo.
   ```bash
   scp project_final.tar.gz YourUWNetID@lolo.uw.edu:/archive/YourLabName/
   ```

**Note**: Large transfers may take hours. Consider using screen or tmux to prevent interruption if SSH connection drops.

4. Verify arrival on Lolo.
   ```bash
   ssh YourUWNetID@lolo.uw.edu
   ls -lh /archive/YourLabName/
   ```

5. Create documentation.
   ```bash
   cd /archive/YourLabName
   nano README_project_final.txt
   ```
   
   Include:
   - Project name and date
   - Data description and file structure
   - Processing steps and parameters used
   - Software versions and environments
   - Related publications or reports
   - Contact information

6. Verify archive integrity before deleting source.
   On Lolo, test the archive:
   ```bash
   tar -tzf project_final.tar.gz | head
   ```
   
   This lists contents without extracting. Verify key files are present.

7. Delete from hot storage after verification.
   ```bash
   ssh YourUWNetID@klone.hyak.uw.edu
   rm -rf /mmfs1/gscratch/YourLabName/project_directory/
   ```

**Warning**: Deletion is permanent from hot storage. Always verify Lolo archive first.

---

## Task 6: Retrieving Data from Lolo

Restore archived data when needed.

### Steps

1. Copy from Lolo to Hyak.
   ```bash
   scp YourUWNetID@lolo.uw.edu:/archive/YourLabName/project_final.tar.gz /mmfs1/gscratch/YourLabName/
   ```

**Note**: Retrieval from tape-based storage may take several hours. Plan ahead.

2. Extract on Hyak.
   ```bash
   cd /mmfs1/gscratch/YourLabName
   tar -xzf project_final.tar.gz
   ```

3. Verify extraction.
   ```bash
   ls -lR | head -50
   ```

**Result**: Shows directory structure and files were extracted correctly.

---

## Task 7: Organizing Lab Data Structure

Implement consistent organization for better management.

### Recommended Structure

```
/gpfs/projects/YourLabName/
├── raw_data/           (original, unprocessed data)
├── processed_data/     (cleaned, transformed data)
├── scripts/            (analysis scripts and code)
├── results/            (final outputs, figures, tables)
├── envs/               (Conda environments)
└── archive_staging/    (data ready to move to Lolo)
```

### Steps

1. Create directory structure.
   ```bash
   cd /gpfs/projects/YourLabName
   mkdir -p raw_data processed_data scripts results envs archive_staging
   ```

2. Set appropriate permissions for lab sharing.
   ```bash
   chmod -R 775 raw_data processed_data scripts results
   ```
   
   This allows lab members to read and write shared data.

3. Document structure for lab members.
   ```bash
   nano README.txt
   ```
   
   Explain what belongs in each directory and naming conventions.

---

## Data Lifecycle Best Practices

**Active computation**: Keep data in hot storage (Hyak/Tillicum scratch or projects directories).

**Project completion**: Archive final datasets, results, and code to Lolo within 30 days of completion.

**Monthly reviews**: Schedule regular storage audits. Identify and archive projects meeting criteria:
- Analysis completed
- Results published or submitted
- No active work planned for 3+ months
- Raw data no longer being actively processed

**Archival checklist**:
- Data compressed into .tar.gz archives
- README file created with comprehensive metadata
- Transfer to Lolo completed and verified
- Archive integrity tested
- Original data deleted from hot storage
- Lab members notified of archive location
- Archive location documented in lab records

---

## Optimizing Resource Requests

**Right-size allocations**: Over-requesting reduces scheduling priority and wastes budget. Under-requesting causes job failures.

**Profile your jobs**: Run representative workloads and monitor actual usage:
```bash
seff JOBID
```

This shows CPU efficiency, memory usage, and runtime statistics.

**Request 10-20% overhead**: After profiling, request slightly more than typical usage to handle variability without excessive over-allocation.

**Use job dependencies**: Chain related jobs to automate workflows:
```bash
sbatch --dependency=afterok:JOBID second_job.sh
```

**Dependency types**:
- `afterok`: Start after successful completion
- `afterany`: Start after completion (success or failure)
- `afternotok`: Start only if previous job failed

**Check queue status**: Before large job submissions, check system load:
```bash
sinfo
```

**Use off-peak times**: Submit large jobs during evenings or weekends for faster allocation.

---

## Troubleshooting

**Problem**: Job pending for hours  
**Solution**: System may be busy. Check `hyakalloc` for available resources. Consider reducing resource request or submitting during off-peak times. For Tillicum, verify budget has remaining funds with `hyakusage -u all`.

**Problem**: Job fails with "out of memory" error  
**Solution**: Increase `--mem` allocation in job script. Check actual memory usage of completed jobs with `seff JOBID` to determine appropriate request.

**Problem**: Transfer interrupted or stalled  
**Solution**: Use rsync instead of scp - it automatically resumes. For very large transfers, use screen or tmux to maintain session if SSH disconnects.

**Problem**: Out of storage quota  
**Solution**: Identify large files with `du -sh * | sort -h`. Archive completed projects to Lolo. Delete temporary files and intermediate results that can be regenerated. Contact lab PI about quota increase if needed.

**Problem**: Can't find archived data on Lolo  
**Solution**: Check README files in archive directory. If data was archived by another lab member, check lab documentation or contact them for archive location and structure.

**Problem**: Job array tasks failing inconsistently  
**Solution**: Check error files for individual tasks (`array_JOBID_TASKID.err`). Ensure input files exist for all array indices. Verify tasks are independent and don't conflict (e.g., writing to same output file).

---

## Best Practices Summary

**Document everything**: Maintain clear documentation of data processing, analysis workflows, and archive locations.

**Automate workflows**: Use job arrays and dependencies to reduce manual job submission.

**Monitor costs**: For Tillicum, regularly check budget usage with `hyakusage -u all`. Each GPU-hour costs $0.90.

**Test before production**: Run short test jobs with subset of data before submitting large batch jobs.

**Compress aggressively**: Use gzip compression for archival to reduce storage costs and transfer times.

**Coordinate with lab**: Share successful job scripts, established workflows, and data organization practices with lab members.

**Plan for reproducibility**: Archive not just data but also code, environments (export conda environments), and documentation to enable future reproduction of results.
