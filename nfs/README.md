# NFS Shared Storage for HPC

## Overview

This component documents Network File System (NFS) shared storage in a virtualized HPC cluster running Rocky Linux 9.8.

NFS allows multiple Linux systems to access files exported by a central server. In an HPC environment, it can provide shared home directories, job scripts, application data, and configuration files.

NFS is networked shared storage; it is not a parallel filesystem equivalent to Lustre.

## Architecture

```text
                 HPC Environment
                        |
               +------------------+
               |    NFS Server    |
               |                  |
               | Exported Paths   |
               | File Permissions|
               +------------------+
                        |
                 Storage Network
                        |
           +------------+------------+
           |                         |
           v                         v
    +--------------+          +--------------+
    | Master Nodes |          | Compute Nodes|
    | NFS Clients  |          | NFS Clients  |
    +--------------+          +--------------+
           |                         |
           +------------+------------+
                        |
                Shared File Access
```

This is a logical design. Document the hosts and directories actually exported and mounted in your environment.

## Responsibilities

- Export selected directories from a central server.
- Mount shared paths on authorized client nodes.
- Make common files available across the cluster.
- Preserve Linux ownership and permission semantics.
- Provide shared access to job scripts, user files, and application data where appropriate.

## Server Configuration

### Inspect exported directories

On the NFS server:

```bash
exportfs -v
cat /etc/exports
```

Review the export definitions and confirm that only the intended client hosts or networks are authorized.

### Inspect the NFS service

```bash
systemctl status nfs-server
```

Service names and configuration can vary depending on the installation.

## Client Validation

### 1. Inspect mounted filesystems

On each intended client:

```bash
findmnt -t nfs,nfs4
```

### 2. Inspect available exports

Where permitted by the server configuration:

```bash
showmount -e <nfs-server>
```

`showmount` is not a definitive test for every NFSv4-only configuration.

### 3. Validate shared access

After confirming the correct mount point:

```bash
findmnt <mount-point>
ls -ld <mount-point>
```

Use an approved test directory to create a file on one client and verify that it is visible on another client.

### 4. Validate identity and permissions

Test file access using an approved user account. Check that the user's UID and GID are consistent across the relevant clients, especially when centralized identity management is used.

Do not use broad permissions such as `chmod 777` to work around ownership problems.

## Example Shared-Storage Workflow

1. An administrator configures an export on the NFS server.
2. An authorized client mounts the export.
3. A user creates a job script or input file in the shared directory.
4. A compute node accesses the same path while executing a scheduled job.
5. Output files are written to the appropriate shared or job-specific directory.

Actual behavior depends on the exported paths, mount options, permissions, and client configuration.

## Evidence Directory

Suggested structure:

```text
nfs/
├── README.md
└── screenshots/
    ├── nfs-server-status.png
    ├── export-configuration.png
    ├── client-mount-status.png
    ├── shared-file-test.png
    └── ownership-permissions.png
```

Use filenames that match your real screenshots.
## Security and Reliability

- Restrict exports to authorized hosts or networks.
- Use appropriate filesystem permissions and ownership.
- Configure firewall rules for the selected NFS version and deployment.
- Document mount options and startup dependencies.
- Consider how active jobs behave if the NFS server becomes unavailable.
- Avoid publishing sensitive addresses, credentials, or private infrastructure details.

