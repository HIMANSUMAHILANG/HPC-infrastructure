# Samba File Server for HPC Infrastructure

## Overview

This component documents Samba-based file sharing in a virtualized HPC environment running Rocky Linux 9.8.

Samba implements SMB file-sharing services and allows authorized users to access shared directories from compatible Linux and Windows clients.

Within an HPC environment, Samba can be useful for exchanging job scripts, input datasets, application files, and results between a workstation and the Linux cluster.

## Architecture

```text id="f5e4c1"
                 HPC Users
                     |
          +----------+----------+
          |                     |
     Windows Client        Linux Client
          |                     |
          +----------+----------+
                     |
                SMB Protocol
                     |
             +---------------+
             | Samba Server  |
             |               |
             | smbd          |
             | SMB Shares    |
             | Access Rules  |
             +---------------+
                     |
             Server Filesystem
                     |
             Shared Directories
```

This is a logical architecture. Update it to match the actual client types, share paths, and authentication configuration used in your lab.

## Responsibilities

- Publish selected directories as SMB shares.
- Authenticate users according to the configured security mode.
- Enforce share-level and filesystem-level access restrictions.
- Provide a convenient file-transfer path between workstations and Linux systems.
- Support controlled access to scripts, input files, and output data.

## Configuration Inspection

### 1. Validate the Samba configuration

On the Samba server:

```bash id="f02v34"
testparm -s
```

This checks the configuration syntax and displays the effective configuration. It does not prove that every client can authenticate or access a share.

### 2. Inspect the service

```bash id="v6m2f1"
systemctl status smb
```

On systems using a separate NetBIOS name service, `nmb` may also be present. Its requirement depends on the configured SMB environment.

### 3. List available shares

From an authorized client:

```bash id="y7w2z4"
smbclient -L //<server-name> -U <username>
```

Enter credentials when prompted. Avoid putting passwords directly in command arguments or committing credentials to scripts.

### 4. Connect to a specific share

```bash id="j8x3p5"
smbclient //<server-name>/<share-name> -U <username>
```

Use the actual server and share names from your configuration.

### 5. Inspect filesystem permissions

On the server:

```bash id="p3k5v7"
ls -ld <share-path>
namei -l <share-path>
```

Share permissions and Linux filesystem permissions both affect access. A share being visible does not necessarily mean that a user can read or write its contents.

## File Access Workflow

1. An administrator configures a directory and defines the SMB share.
2. Samba loads the configuration and makes the share available.
3. An authorized client connects using the configured authentication method.
4. Samba checks share access rules and the underlying filesystem permissions.
5. The client reads or writes files according to its effective permissions.
6. Users can transfer files to the HPC environment or retrieve job results.

