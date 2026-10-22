# Slurm Workload Manager — HPC Cluster

## Overview

This component documents the Slurm workload management layer of a virtualized High-Performance Computing (HPC) cluster running Rocky Linux 9.8.

Slurm manages compute resources, queues user jobs, schedules workloads, and coordinates job execution across compute nodes.

The lab is designed with two master nodes for controller high availability and two compute nodes for workload execution.

## Architecture

```text
                      HPC Users
                          |
                          | Submit jobs: sbatch / srun
                          |
                  +-------------------+
                  |  Slurm Client CLI |
                  +-------------------+
                          |
                          v
             +---------------------------+
             |   Master Node 1            |
             |   Primary slurmctld        |
             +---------------------------+
                          |
                 Scheduling and control
                          |
               +----------+----------+
               |                     |
               v                     v
       +---------------+     +---------------+
       | Compute Node 1|     | Compute Node 2|
       |    slurmd     |     |    slurmd     |
       +---------------+     +---------------+
               |                     |
               +----------+----------+
                          |
                  Job execution
                          |
             MPI communication network


             +---------------------------+
             |   Master Node 2            |
             |   Backup slurmctld         |
             +---------------------------+
                          |
                 HA / failover design
```

> **Architecture note:** This is a logical architecture. The backup controller's actual readiness and automatic failover behavior must be validated in the deployed environment. The diagram does not imply that both controllers schedule jobs simultaneously.

## Components

| Component | Responsibility |
|---|---|
| `slurmctld` | Central scheduling, job management, and resource allocation |
| `slurmd` | Node-level daemon that manages job execution on compute nodes |
| `sbatch` | Submits batch jobs to the scheduler |
| `srun` | Launches job steps and can request interactive allocations |
| `squeue` | Displays queued and running jobs |
| `sinfo` | Displays partitions and node availability |
| `scontrol` | Inspects and manages Slurm configuration and cluster state |
| `sacct` | Reports accounting information when Slurm accounting is configured |

## Job Execution Workflow

### 1. Job submission

The user submits a batch script:

```bash
sbatch job.sh
```

The Slurm client submits the job request to the active controller.

### 2. Resource selection

The controller evaluates the job's resource requirements, partition, priority, and available resources.

The job may remain pending until suitable resources become available.

### 3. Resource allocation

When resources are available, Slurm allocates the required nodes, CPUs, and other configured resources.

### 4. Job execution

Slurm coordinates execution through `slurmd` on the allocated compute nodes.

For distributed MPI workloads, the MPI runtime launches the parallel processes using the configured Slurm integration or supported launch mechanism.

### 5. Monitoring

Users can inspect the job and cluster state:

```bash
squeue
sinfo
scontrol show nodes
scontrol show jobs
```

### 6. Completion and output

For a batch job, the application output and errors are written to the paths configured in the batch script, commonly files such as `slurm-%j.out`.

The job's final state can be inspected using:

```bash
sacct
```

This command requires the appropriate Slurm accounting configuration and available job records.

## Example Batch Job

Save this example as `job.sh`:

```bash
#!/bin/bash
#SBATCH --job-name=hostname-test
#SBATCH --partition=compute
#SBATCH --nodes=2
#SBATCH --ntasks=2
#SBATCH --time=00:02:00
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

echo "Job ID: $SLURM_JOB_ID"
echo "Allocated nodes: $SLURM_JOB_NODELIST"

srun hostname
```

Submit the job:

```bash
sbatch job.sh
```

Check the queue:

```bash
squeue -u "$USER"
```

Inspect the output after completion:

```bash
cat slurm-JOBID.out
```

Replace `JOBID` with the job ID returned by `sbatch`.

**Prerequisites:** The `compute` partition must exist, both compute nodes must be configured and available, and the requested resources must be schedulable. This example demonstrates basic multi-node execution; it is not an MPI benchmark.

## Validation Commands

Run the following commands from a host with the Slurm client installed and configured:

```bash
scontrol ping
sinfo
scontrol show nodes
squeue
```

Check service status on the appropriate hosts:

```bash
systemctl status slurmctld
systemctl status slurmd
```

The service names and expected states depend on the node role and installed Slurm configuration.

### Validation Checklist

- [ ] Slurm controller responds to client requests.
- [ ] Both compute nodes are registered with the controller.
- [ ] Compute nodes report an expected state.
- [ ] A batch job can be submitted.
- [ ] A job transitions from pending to running when resources are available.
- [ ] A job completes successfully on the allocated compute nodes.
- [ ] Job output and exit status are verified.
- [ ] Backup-controller behavior is tested.
- [ ] Controller failover is tested without corrupting scheduler state.
- [ ] MPI execution is tested separately from basic hostname execution.

Mark a checklist item complete only after testing it.

## High Availability

The lab design includes two master nodes intended to support Slurm controller high availability.

A production-quality deployment must address:

- Primary and backup controller configuration.
- Consistent Slurm configuration across both controllers.
- Appropriate controller-state handling and access.
- Reliable host naming and network connectivity.
- Failover and recovery procedures.
- Testing of jobs and controller behavior during failure.

A second controller alone does not guarantee automatic failover. The deployment should be tested to establish whether failover is automatic or requires operator intervention.

## Networking

The cluster separates network purposes where the virtualized infrastructure permits:

- **Slurm control traffic:** Communication between Slurm clients, controllers, and compute nodes.
- **MPI traffic:** Communication between distributed application processes.
- **Provisioning network:** xCAT-related node provisioning and boot traffic.
- **Storage network:** File-system access and data transfer.

The actual interfaces, IP addresses, routes, and firewall rules should be documented in the cluster's architecture documentation. Do not publish sensitive network details unnecessarily.

## Evidence

Store screenshots in this directory and reference them here:

```text
slurm/
├── README.md
└── screenshots/
    ├── controller-status.png
    ├── node-status.png
    ├── job-submission.png
    ├── job-queue.png
    └── job-execution.png
```

Example:

```markdown
### Compute Node Status

![Compute node status](screenshots/node-status.png)
```

Use these filenames only when the corresponding screenshots exist and demonstrate the stated behavior.

## Current Scope and Limitations

This documentation describes the intended architecture and expected validation workflow.

The following should be reported according to actual test results:

- Whether both compute nodes can execute jobs successfully.
- Whether controller failover works.
- Whether distributed MPI workloads run successfully.
- Whether shared storage is mounted and accessible during job execution.
- Whether job accounting and monitoring are configured.

Do not treat planned or configured functionality as validated until the corresponding tests have passed.

## Technologies

- Rocky Linux 9.8
- Slurm Workload Manager
- Linux systemd
- MPI runtime, where configured
- Virtualized HPC infrastructure

## Future Improvements

- Automated controller failover validation.
- Slurm job accounting and historical usage reporting.
- Performance benchmarking with HPL, OSU Micro-Benchmarks, and STREAM.
- Ansible-based configuration management.
- Integration with Prometheus and Grafana.
- Testing of shared storage under concurrent workload execution.

## Project Goal

The goal is to understand and document HPC workload management, resource scheduling, distributed job execution, and controller high availability through a reproducible virtualized lab.