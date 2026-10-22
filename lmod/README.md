# Lmod — HPC Environment Module Management

## Overview

This component documents Lmod environment modules in a virtualized HPC cluster running Rocky Linux 9.8.

Lmod allows users to select software versions through modulefiles instead of manually configuring executable paths and library variables.

The goal is to provide a consistent software environment for interactive sessions and Slurm batch jobs.

## Architecture

```text id="h4x6j8"
             Shared HPC Software
                      |
        +-------------+-------------+
        |             |             |
       GCC         OpenMPI       OpenFOAM
        |             |             |
        +-------------+-------------+
                      |
              Lmod Modulefiles
                      |
             +--------+--------+
             |                 |
         Login Node        Compute Nodes
             |                 |
        module load       module load
             |                 |
             +--------+--------+
                      |
                Slurm Job
                      |
              Application Execution
```

The diagram assumes software and modulefiles are accessible from each node where they are needed. Verify actual mount points and module paths in your environment.

## Responsibilities

- Expose available software versions through modulefiles.
- Configure environment variables for selected software.
- Help users switch between supported compiler and MPI environments.
- Reduce manual environment configuration in job scripts.
- Support repeatable software selection for HPC workloads.

## How Module Loading Works

1. Lmod searches the configured modulefile paths.
2. The user selects a module using `module load`.
3. The modulefile updates environment variables, such as `PATH` and, when required, library search paths.
4. The user launches the application using the selected software environment.
5. A batch script can load the same modules before starting a Slurm job.

Lmod does not automatically ensure that the selected software is compatible with every other module. Compiler, MPI, and application compatibility must be managed explicitly.

## Common Commands

### List available modules

```bash id="x19gkd"
module avail
```

### Inspect loaded modules

```bash id="c6q5su"
module list
```

### Load software

```bash id="r4c9jy"
module load gcc/13.0
module load mpi-5.0.10
```

Replace the example module names with the exact names available in your module tree.

### Remove loaded modules

```bash id="z4v0k1"
module purge
```

### Inspect an individual modulefile

```bash id="p5k9wf"
module show gcc/13.0
```

### Verify the compiler and MPI environment

```bash id="e7g4tb"
which gcc
gcc --version
which mpirun
mpirun --version
```

These checks verify executable resolution and reported versions. They do not, on their own, prove that a multi-node MPI job will execute successfully.

## Example Slurm Batch Job

```bash id="j8c2ym"
#!/bin/bash
#SBATCH --job-name=module-check
#SBATCH --partition=compute
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --time=00:02:00
#SBATCH --output=module-check-%j.out

module purge
module load gcc/13.0
module load mpi-5.0.10

echo "Loaded modules:"
module list

echo "Compiler:"
which gcc
gcc --version

echo "MPI runtime:"
which mpirun
mpirun --version
```

Submit the script using:

```bash id="g9j3fr"
sbatch module-check.sh
```

Check the job with `squeue` and inspect the output file after it finishes.

The example assumes those module names exist, the compute node can access their software paths, and the `compute` partition is configured.

## Modulefile and Software Paths

Document the actual modulefile and software locations used in your lab. For example:

```text id="d6y2tx"
/shared/apps/
├── gcc/
├── openmpi/
├── openfoam/
└── modulefiles/
```

This is an illustrative directory layout, not a claim about the current filesystem structure. Update it to match the real paths in your cluster.

Include only screenshots you actually have. A module appearing in `module avail` demonstrates discovery; a successful job demonstrates that the requested environment can be used in that execution context.

## Validation Checklist

- [ ] Lmod initializes correctly in user shells.
- [ ] Expected modulefiles are discoverable.
- [ ] GCC loads and reports the intended version.
- [ ] OpenMPI loads and reports the intended version.
- [ ] Modulefiles point to valid software installations.
- [ ] Compute nodes can access the required software paths.
- [ ] Slurm batch jobs load modules successfully.
- [ ] MPI execution is tested separately.
- [ ] Module and software versions are documented.

## Common Issues

### Module not found

Check module search paths and exact module names:

```bash id="t2h6mq"
module avail
module show <module-name>
```

### Module loads but executable is missing

Inspect the modulefile and verify that the referenced software installation exists and is accessible.

### Software works on the login node but not on compute nodes

Check shared mounts, permissions, environment initialization, and software dependencies on the compute nodes.

### MPI module loads but distributed execution fails

Investigate MPI build configuration, Slurm integration, network selection, runtime libraries, and node-to-node connectivity. Successful module loading does not prove MPI compatibility.

## Current Status

Record the modules that are installed, discoverable, and tested. Distinguish between modulefile validation, single-node execution, and multi-node MPI execution.

## Future Improvements

- Manage modulefiles using Ansible.
- Document compiler/MPI compatibility.
- Add module validation to Slurm test jobs.
- Establish versioning and maintenance procedures for shared software.
- Add HPC performance benchmarks after the execution environment is validated.
