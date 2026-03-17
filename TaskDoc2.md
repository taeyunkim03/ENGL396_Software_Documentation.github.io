# Managing Research Data and Jobs on UW HPC Systems

## Audience
This guide is for academic researchers and principal investigators managing computational projects on Hyak and Tillicum with data archiving needs on Lolo.

## What You Will Learn
In this guide, you will learn how to organize data across storage tiers, submit batch jobs, transfer data efficiently, and archive completed projects to Lolo.

## Prerequisites
- Active accounts on Hyak, Tillicum, and/or Lolo
- SSH access configured
- Understanding of your project's computational requirements

## Estimated Time
20–30 minutes to learn core workflows

---

## Understanding Storage Architecture

**Hyak**:
- Home: `/mmfs1/home/`*YourUWNetID* (10 GB)
- Lab scratch: `/mmfs1/gscratch/`*YourLabName* (variable quota)

**Tillicum**:
- Home: `/gpfs/home/`*YourUWNetID* (10 GB)
- Lab projects: `/gpfs/projects/`*YourLabName* (1 TB)
- Scratch: `/gpfs/scrubbed` (100 TB, purged after 60 days)

**Lolo**:
- Lab archive: `/archive/`*YourLabName* (backed up, slow retrieval)

---

## Checking Storage Usage

Monitor storage regularly to avoid hitting quota limits.

### Steps

1. On Hyak, check lab allocation.
```bash
   ssh YourUWNetID@klone.hyak.uw.edu
   hyakalloc
```
   Replace *YourUWNetID* with your actual NetID.

**Result**: Shows storage quotas and current usage.

2. Check directory sizes.
```bash
   cd /mmfs1/gscratch/YourLabName
   du -h --max-depth=1
```
   Replace *YourLabName* with your actual lab name.

**Result**: Shows usage breakdown by subdirectory.

3. On Lolo, view usage report.
```bash
   ssh YourUWNetID@lolo.uw.edu
   cd /archive/YourLabName
   cat usage_report.txt
```
   Replace *YourUWNetID* and *YourLabName* with your actual NetID and lab name.

---

## Submitting Batch Jobs

Use batch jobs to run long computations without manual supervision. This is the standard workflow for most production analyses.

### Steps

1. Create a job script.
```bash
   nano my_job.sh
```
   Replace *my_job.sh* with your preferred script name.

   > **Note**: This guide uses `nano` as the text editor. If your system uses a different editor (such as `vim` or `emacs`), substitute that command instead.

2. Write the script content. Copy the template below as a starting point. Lines marked with a comment are ones you will likely need to customize:
```bash
   #!/bin/bash
   #SBATCH --job-name=analysis          # Replace with a descriptive name for your job
   #SBATCH --account=YourLabName        # Replace with your lab or account name
   #SBATCH --partition=compute
   #SBATCH --nodes=1
   #SBATCH --cpus-per-task=16           # Adjust based on your workload
   #SBATCH --mem=128G                   # Adjust based on your memory needs
   #SBATCH --time=12:00:00              # Adjust: format is HH:MM:SS
   #SBATCH --output=job_%j.out
   #SBATCH --error=job_%j.err

   module load python/3.12
   conda activate myproject             # Replace with your conda environment name
   python analysis_script.py           # Replace with your script name
   echo "Job completed"
```

3. Save and exit (Ctrl+O, Enter, Ctrl+X in nano).

4. Submit the job.
```bash
   sbatch my_job.sh
```
   Replace *my_job.sh* with your script name.

**Result**: Returns a job ID number.

5. Monitor job status.
```bash
   squeue -u YourUWNetID
```
   Replace *YourUWNetID* with your actual NetID.

**Result**: Shows job state (PD=pending, R=running, CG=completing).

6. Check output after completion.
```bash
   less job_JOBID.out
```
   Replace *JOBID* with the job ID number returned in step 4.

7. To cancel a running job:
```bash
   scancel JOBID
```
   Replace *JOBID* with the actual job ID.

---

## Creating Job Arrays

Follow these steps only if you need to run multiple similar tasks in parallel, such as processing a set of input files or running a parameter sweep. If you are running a single analysis, use [Submitting Batch Jobs](#submitting-batch-jobs) instead.

### Steps

1. Create an array job script.
```bash
   nano array_job.sh
```
   Replace *array_job.sh* with your preferred script name.

2. Add the array directive. Copy the template below and customize the marked lines:
```bash
   #!/bin/bash
   #SBATCH --job-name=array_analysis    # Replace with a descriptive name
   #SBATCH --account=YourLabName        # Replace with your lab or account name
   #SBATCH --partition=compute
   #SBATCH --cpus-per-task=8            # Adjust based on your workload
   #SBATCH --mem=64G                    # Adjust based on your memory needs
   #SBATCH --time=04:00:00              # Adjust: format is HH:MM:SS
   #SBATCH --array=1-10                 # Replace with your desired task range
   #SBATCH --output=array_%A_%a.out
   #SBATCH --error=array_%A_%a.err

   echo "Processing task ${SLURM_ARRAY_TASK_ID}"
   python script.py --input data_${SLURM_ARRAY_TASK_ID}.csv   # Replace with your script and input pattern
```

3. Submit:
```bash
   sbatch array_job.sh
```
   Replace *array_job.sh* with your script name.

**Result**: Creates 10 jobs numbered 1–10, each processing a different input file.

**Array variable reference**:
- `%A`: Master job array ID
- `%a`: Individual task ID
- `SLURM_ARRAY_TASK_ID`: Contains the current task number at runtime
- Useful for parameter sweeps or processing multiple datasets

---

## Transferring Data Between Systems

Choose the appropriate command based on where your data is coming from and how large it is.

### Transferring from Your Local Machine to a Cluster

Use these commands when copying data from your own computer to Hyak or Tillicum.

**For small files (< 1 GB)**:
```bash
scp local_file.txt YourUWNetID@klone.hyak.uw.edu:/mmfs1/gscratch/YourLabName/
```
Replace *YourUWNetID*, *local_file.txt*, and *YourLabName* with your actual values.

**For large files (> 1 GB)**, use `rsync` for reliability:
```bash
rsync -avz --progress local_data/ YourUWNetID@klone.hyak.uw.edu:/mmfs1/gscratch/YourLabName/data/
```
Replace *YourUWNetID*, *local_data/*, and *YourLabName* with your actual values.

`rsync` options:
- `-a`: Preserve permissions and timestamps
- `-v`: Show progress
- `-z`: Compress during transfer
- `--progress`: Display a progress bar

If the transfer is interrupted, rerun the same `rsync` command to resume automatically.

**For very large datasets (multi-TB)**: Contact UW Research Computing about using Globus for optimal transfer speeds and reliability.

### Transferring Between Hyak and Tillicum

Use this command when moving data between UW clusters rather than from your local machine. Run this command from a Hyak login node:
```bash
rsync -avz /mmfs1/gscratch/YourLabName/data/ YourUWNetID@tillicum.hyak.uw.edu:/gpfs/projects/YourLabName/data/
```
Replace *YourLabName* and *YourUWNetID* with your actual values.

---

## Archiving Data to Lolo

Follow these steps when a project is complete and you are ready to move it to long-term storage. Do not use Lolo for data you are actively working with, as retrieval can take several hours.

### Steps

1. Compress data on Hyak or Tillicum.
```bash
   tar -czf project_final.tar.gz /path/to/project/data/
```
   Replace *project_final.tar.gz* with your preferred archive name and replace */path/to/project/data/* with the actual path to your data.

   **Command breakdown**:
   - `tar`: Archive utility
   - `-c`: Create archive
   - `-z`: Compress with gzip
   - `-f`: Specify filename

2. Verify the archive was created successfully.
```bash
   ls -lh project_final.tar.gz
```
   Replace *project_final.tar.gz* with your archive name.

3. Transfer to Lolo.
```bash
   scp project_final.tar.gz YourUWNetID@lolo.uw.edu:/archive/YourLabName/
```
   Replace *project_final.tar.gz*, *YourUWNetID*, and *YourLabName* with your actual values.

   > **Note**: Large transfers may take hours. Consider running this inside `screen` or `tmux` to prevent interruption if your SSH connection drops.

4. Verify arrival on Lolo.
```bash
   ssh YourUWNetID@lolo.uw.edu
   ls -lh /archive/YourLabName/
```
   Replace *YourUWNetID* and *YourLabName* with your actual values.

5. Create documentation.
```bash
   cd /archive/YourLabName
   nano README_project_final.txt
```
   Replace *YourLabName* with your actual lab name and replace *README_project_final.txt* with a descriptive filename for your project.

   Include:
   - Project name and date
   - Data description and file structure
   - Processing steps and parameters used
   - Software versions and environments
   - Related publications or reports
   - Contact information

6. Verify archive integrity before deleting source files. On Lolo, test the archive:
```bash
   tar -tzf project_final.tar.gz | head
```
   Replace *project_final.tar.gz* with your archive name. This lists contents without extracting. Confirm that key files are present.

> **Warning**: Deletion from hot storage is permanent. Always verify the Lolo archive in step 6 before proceeding to step 7.

7. Delete from hot storage after verification.
```bash
   ssh YourUWNetID@klone.hyak.uw.edu
   rm -rf /mmfs1/gscratch/YourLabName/project_directory/
```
   Replace *YourUWNetID*, *YourLabName*, and *project_directory* with your actual values.

---

## Retrieving Data from Lolo

Follow these steps only if you need to restore a previously archived project. Plan ahead — retrieval from tape-based storage may take several hours.

### Steps

1. Copy from Lolo to Hyak.
```bash
   scp YourUWNetID@lolo.uw.edu:/archive/YourLabName/project_final.tar.gz /mmfs1/gscratch/YourLabName/
```
   Replace *YourUWNetID*, *YourLabName*, and *project_final.tar.gz* with your actual values.

2. Extract on Hyak.
```bash
   cd /mmfs1/gscratch/YourLabName
   tar -xzf project_final.tar.gz
```
   Replace *YourLabName* and *project_final.tar.gz* with your actual values.

3. Verify extraction.
```bash
   ls -lR | head -50
```

**Result**: Shows directory structure confirming that files were extracted correctly.

---

## Organizing Lab Data Structure

Follow these steps when setting up a new project or standardizing your lab's storage conventions. This is a one-time setup, not a recurring workflow.

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

1. Create the directory structure.
```bash
   cd /gpfs/projects/YourLabName
   mkdir -p raw_data processed_data scripts results envs archive_staging
```
   Replace *YourLabName* with your actual lab name.

2. Set appropriate permissions for lab sharing.
```bash
   chmod -R 775 raw_data processed_data scripts results
```
   This allows all lab members to read and write to shared directories.

3. Document the structure for lab members.
```bash
   nano README.txt
```
   Explain what belongs in each directory and the naming conventions your lab uses.

---

## Data Lifecycle Best Practices

**Active computation**: Keep data in hot storage (Hyak/Tillicum scratch or projects directories).

**Project completion**: Archive final datasets, results, and code to Lolo within 30 days of completion.

**Monthly reviews**: Schedule regular storage audits. Identify and archive projects that meet these criteria:
- Analysis completed
- Results published or submitted
- No active work planned for 3+ months
- Raw data no longer being actively processed

**Archival checklist**:
- Data compressed into `.tar.gz` archives
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
Replace *JOBID* with your actual job ID. This shows CPU efficiency, memory usage, and runtime statistics.

**Request 10–20% overhead**: After profiling, request slightly more than typical usage to handle variability without excessive over-allocation.

**Use job dependencies**: Chain related jobs to automate workflows:
```bash
sbatch --dependency=afterok:JOBID second_job.sh
```
Replace *JOBID* with the ID of the preceding job and replace *second_job.sh* with your script name.

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

## Best Practices Summary

**Document everything**: Maintain clear documentation of data processing, analysis workflows, and archive locations.

**Automate workflows**: Use job arrays and dependencies to reduce manual job submission.

**Monitor costs**: For Tillicum, regularly check budget usage with `hyakusage -u all`. Each GPU-hour costs $0.90.

**Test before production**: Run short test jobs with a subset of data before submitting large batch jobs.

**Compress aggressively**: Use gzip compression for archival to reduce storage costs and transfer times.

**Coordinate with your lab**: Share successful job scripts, established workflows, and data organization practices with lab members.

**Plan for reproducibility**: Archive not just data but also code, environments (export Conda environments), and documentation to enable future reproduction of results.
