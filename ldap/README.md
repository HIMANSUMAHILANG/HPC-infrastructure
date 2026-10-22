# Centralized Identity Management with LDAP

## Overview

This component documents centralized Linux identity management for a virtualized HPC cluster running Rocky Linux 9.8.

LDAP provides a central directory for user and group information. Linux clients integrate with the directory through NSS, PAM, and nslcd, where configured.

The objective is to provide consistent user identity information across the cluster so that a user's UID and primary GID remain consistent on the systems that execute jobs and access shared storage.

## Architecture

```text
                  HPC Administrator
                         |
                         v
                +------------------+
                |   LDAP Server    |
                |                  |
                | User entries     |
                | Group entries    |
                | UID / GID        |
                +------------------+
                         |
                  LDAP connection
                         |
            +------------+------------+
            |                         |
            v                         v
    +----------------+       +----------------+
    | Master Nodes   |       | Compute Nodes   |
    |                |       |                |
    | NSS            |       | NSS            |
    | PAM            |       | PAM            |
    | nslcd          |       | nslcd          |
    +----------------+       +----------------+
            |                         |
            +------------+------------+
                         |
                  Consistent identity
                  for HPC workloads
```

This is a logical design. Include only the clients and authentication paths that are actually configured and tested.

## Component Responsibilities

| Component | Purpose |
|---|---|
| LDAP server | Stores directory entries for users and groups |
| LDAP client configuration | Defines how clients connect to and search the directory |
| nslcd | Serves NSS and supported PAM requests using LDAP |
| NSS | Resolves user and group information |
| PAM | Integrates authentication and session policies with login services |
| UID/GID | Identifies users and groups consistently across Linux systems |

## User Lookup Workflow

1. An application or command requests a user or group lookup.
2. NSS checks the sources configured in `/etc/nsswitch.conf`.
3. If LDAP is configured as a source, the request is handled by the configured LDAP client integration.
4. nslcd queries the LDAP directory.
5. The directory entry is returned to the client if the lookup succeeds.
6. Linux applications receive the corresponding user or group information.

For authentication, PAM follows its configured authentication stack. Its behavior must be validated separately from NSS lookups.

## Validation

Run these commands on an LDAP client.

### 1. Check the client service

```bash
systemctl status nslcd
```

### 2. Verify user lookup

```bash
getent passwd <ldap-username>
```

### 3. Verify group lookup

```bash
getent group <ldap-group>
```

### 4. Inspect NSS configuration

```bash
grep -E '^(passwd|group|shadow):' /etc/nsswitch.conf
```

### 5. Inspect LDAP client configuration

Review the relevant settings in `/etc/nslcd.conf` and `/etc/nsswitch.conf`. Redact bind credentials, secrets, and sensitive directory details before sharing configuration.

### 6. Test authentication

Use a controlled test account and the approved login method to verify authentication and session creation. Do not include passwords in shell history, screenshots, or repository files.

## Evidence Directory

Recommended structure:

```text
ldap/
├── README.md
└── screenshots/
    ├── ldap-service-status.png
    ├── ldap-user-entry.png
    ├── client-user-lookup.png
    ├── client-group-lookup.png
    └── authentication-test.png
```

These names are examples. Include only screenshots that exist and demonstrate the stated behavior.

## Validation Checklist

- [ ] LDAP service is running on the directory server.
- [ ] Test user and group entries exist.
- [ ] LDAP clients can reach the directory server.
- [ ] NSS resolves the test user.
- [ ] NSS resolves the test group.
- [ ] PAM authentication succeeds for an approved test account.
- [ ] UID and GID values match across the relevant nodes.
- [ ] Shared-storage permissions behave correctly for the test user.
- [ ] LDAP traffic is protected according to the deployment's security requirements.
- [ ] Client behavior during LDAP server unavailability is understood and tested.

## Security Considerations

- Do not commit LDAP bind passwords or other credentials.
- Prefer TLS-protected LDAP connections where supported and configured.
- Restrict directory access using appropriate access controls.
- Use least-privilege service accounts where required.
- Avoid publishing real user details or sensitive directory structure.
- Test authentication changes with a recovery path so administrators do not lock themselves out.

## Relationship to HPC Workloads

Consistent UIDs and GIDs are important when users submit jobs on one node, execute them on compute nodes, and access shared filesystems.

LDAP does not itself schedule jobs, mount shared filesystems, or distribute software. These functions are provided by Slurm, the storage system, and the software module environment, respectively.

## Current Status

Record which functions have been configured and which have been validated. Successful LDAP service startup alone does not establish that user lookup, authentication, cross-node identity consistency, and shared-storage permissions all work.

## Future Improvements

- Automate LDAP client configuration with Ansible.
- Test user and group consistency across all cluster nodes.
- Document directory backup and recovery.
- Validate behavior during directory-server outages.
- Integrate identity validation into cluster health checks.
