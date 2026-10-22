# xCAT — HPC Node Provisioning and Management

## Overview

xCAT (Extreme Cloud Administration Toolkit) is an infrastructure provisioning and management tool used to deploy and manage operating systems and configurations across HPC nodes.

In this virtualized HPC lab, xCAT is intended to manage the provisioning lifecycle of compute nodes running Rocky Linux 9.8.

## Architecture

```text
                 HPC Management
                       |
                +--------------+
                |  xCAT Node   |
                |              |
                | Node Database|
                | OS Images    |
                | DHCP / PXE   |
                | Provisioning |
                +--------------+
                       |
               Provisioning Network
                       |
             +---------+---------+
             |                   |
      +--------------+    +--------------+
      | Compute Node1|    | Compute Node2|
      | Rocky Linux  |    | Rocky Linux  |
      +--------------+    +--------------+
             |                   |
             +---------+---------+
                       |
                 Slurm Workload
                   Management
```

This is the intended logical design. The actual boot process depends on the configured provisioning method, network services, and compute-node boot configuration.

## Core Responsibilities

- Maintain node definitions and hardware or virtual-machine attributes.
- Manage OS images and provisioning resources.
- Support network boot and operating-system deployment where configured.
- Apply node configuration and provisioning workflows.
- Provide administrative commands for node inventory and management.

## Provisioning Workflow

1. Define a compute node in the xCAT database.
2. Configure the node's network and provisioning attributes.
3. Prepare the required operating-system image or installation resources.
4. Configure the provisioning services and boot environment.
5. Boot the target node using the configured method.
6. Verify that the operating system and required node configuration are installed.
7. Validate connectivity between the node, xCAT, and Slurm.
8. Confirm that the compute node is available to the scheduler.

## Example Validation Commands

The following commands are examples; availability and exact syntax depend on the installed xCAT version and configuration.

```bash
lsdef -t node
lsdef -t osimage
tabdump site
```

Inspect relevant services on the management node:

```bash
systemctl --type=service --state=running
```

Check the node's network connectivity:

```bash
ping -c 3 <compute-node-address>
```

Replace placeholders with your lab's actual values. Do not publish sensitive IP addresses or network configuration.

## Evidence

Store screenshots in this directory:

```text
xcat/
├── README.md
└── screenshots/
    ├── node-definitions.png
    ├── os-image-details.png
    ├── provisioning-network.png
    └── node-validation.png
```

Use only filenames that correspond to actual screenshots. A node definition proves that a definition exists; successful provisioning requires separate evidence of the node booting and receiving its intended configuration.

## Validation Checklist

- [ ] xCAT management services are operational.
- [ ] Compute nodes are defined in xCAT.
- [ ] Required OS image or installation resources are available.
- [ ] Provisioning network settings are correct.
- [ ] The configured boot method works.
- [ ] A compute node boots with the expected operating system.
- [ ] The provisioned node can communicate with the management node.
- [ ] The provisioned node can register with Slurm.
- [ ] Provisioning can be repeated reliably.

## Current Status

Update this section based on actual tests:

- **Configured:** Record the xCAT components and node definitions that exist.
- **Validated:** Record successful boot and provisioning tests.
- **Pending:** Record services or workflows that have not yet been verified.

Do not mark the provisioning workflow as complete solely because xCAT is installed or node definitions exist.

## Relationship to Slurm

xCAT and Slurm have different responsibilities:

- **xCAT:** Infrastructure provisioning and node management.
- **Slurm:** Resource allocation, job scheduling, and workload execution.

A provisioned node still requires correct Slurm configuration before it can participate in scheduled workloads.

## Future Improvements

- Automate repeatable compute-node provisioning.
- Integrate node configuration with Ansible.
- Validate node recovery and reprovisioning.
- Document network boot troubleshooting.
- Automate post-provisioning Slurm health checks.
