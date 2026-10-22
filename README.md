# HPC-INFRA
# Production-Inspired HPC Infrastructure Lab

A virtualized High-Performance Computing (HPC) environment built with **Rocky Linux 9.8, Slurm, xCAT, LDAP, Lmod, shared storage, monitoring, and a Streamlit-based job submission portal**.

## Overview

This project brings together core HPC infrastructure components in a virtualized cluster environment. It integrates workload scheduling, node provisioning, centralized user management, software environment management, file services, monitoring, and a web-based job submission interface.

The infrastructure is designed around two master nodes, two compute nodes, centralized identity management, and dedicated network interfaces for cluster communication and provisioning.

The Streamlit-based HPC Job Submission Portal provides a simplified interface for selecting input files, configuring execution environments and resource requirements, submitting jobs to Slurm, and monitoring job status.

## Architecture

![HPC Infrastructure Architecture](docs/architecture/infra_net.png)

The infrastructure integrates workload management, provisioning, identity services, storage, software modules, monitoring, and user-facing job submission into a unified HPC environment.

## Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| Operating System | Rocky Linux 9.8 | Linux platform for the HPC infrastructure |
| Workload Management | Slurm | Job scheduling and resource allocation |
| High Availability | Slurm controller HA design | Redundant controller architecture |
| Node Provisioning | xCAT | Compute-node provisioning and management |
| User Management | LDAP, PAM, NSS, nslcd | Centralized identity and authentication integration |
| Software Management | Lmod | Managing compiler, MPI, and application environments |
| MPI | OpenMPI | Distributed-memory parallel execution |
| File Services | Samba | Centralized file sharing and input-file management |
| Shared Storage | NFS | Shared directory access across systems |
| Parallel Filesystem | Lustre | Parallel storage integration |
| Metrics Monitoring | Prometheus | Infrastructure metrics collection |
| Dashboards | Grafana | Metrics and monitoring visualization |
| Log Collection | Grafana Alloy | Log collection and forwarding |
| Log Management | Loki | Centralized log storage and querying |
| Web Application | Streamlit | HPC job submission portal |
| Automation | Ansible | Infrastructure configuration and automation |

## Infrastructure Components

### 1. Slurm Workload Management

Configured a Slurm-based workload management environment with two master nodes in an HA-oriented architecture and two compute nodes.

Slurm manages job submission, scheduling, resource allocation, and execution across the compute nodes.

### 2. xCAT Provisioning

Integrated xCAT for centralized node provisioning and management, covering the infrastructure concepts required to prepare and manage HPC compute nodes.

### 3. Centralized User Management

Integrated LDAP with Linux identity and authentication components, including PAM, NSS, and nslcd, to support centralized user account management across participating systems.

### 4. Lmod Software Environment Management

Configured Lmod to manage software environments, allowing users to load compilers, MPI libraries, and application software through environment modules.

Example:

```bash
module load gcc/13.0
module load mpi-5.0.10
```

### 5. Storage and File Services

Integrated different storage and file-sharing technologies for HPC workloads:

- **Samba:** Centralized file sharing and input-file management.
- **NFS:** Network-based shared directory access.
- **Lustre:** Parallel-filesystem configuration and integration.

### 6. Monitoring and Observability

Integrated Prometheus, Grafana, Grafana Alloy, and Loki to support infrastructure monitoring, dashboard visualization, and centralized log management.

### 7. Streamlit HPC Job Submission Portal

Developed a Streamlit-based interface to simplify the HPC job submission workflow.

The portal brings together file selection, execution configuration, and Slurm job management in a single interface.

**Portal workflow:**

1. Users access the Streamlit interface.
2. Users select input files from a database-backed catalog.
3. Selected files are copied from the Samba file server into the user's working directory.
4. Users configure environment variables and resource requirements.
5. The application generates a Slurm batch script.
6. The job is submitted to the Slurm scheduler.
7. Users check job status through the portal.

The portal provides a user-friendly interface for interacting with the HPC environment without requiring users to manually construct every batch submission command.

**Portal documentation:** [Dashboard README](Dashboard/README.md)

## Repository Structure

```text
HPC-INFRA/
├── README.md
├── docs/
│   └── architecture/
│       └── infra_net.png
├── Dashboard/
├── ldap/
├── lmod/
├── monitoring/
├── nfs/
├── samba/
├── simulation/
├── slurm/
└── xcat/
```

Each component directory contains its respective documentation and related project materials.

## Future Enhancements

- Automate infrastructure configuration using Ansible.
- Improve Slurm controller failover and recovery.
- Extend portal functionality with job history and resource-usage information.
- Improve multi-node MPI workload execution.
- Expand monitoring dashboards and alerting.
- Complete and refine parallel-filesystem integration.
- Introduce automated HPC performance benchmarking.

## Skills and Technologies

**HPC Administration · Linux · Slurm · xCAT · LDAP · PAM/NSS · Lmod · OpenMPI · Samba · NFS · Lustre · Prometheus · Grafana · Loki · Streamlit · Ansible**
