---
title: "Overview of the ARCHER2 system"
teaching: 20
exercises: 0
questions:
- "What hardware and software is available on ARCHER2?"
- "How does the hardware fit together?"
objectives:
- "Gain an overview of the technology available on the ARCHER2 service."
keypoints:
- "ARCHER2 consists of high performance login nodes, compute nodes, storage systems and interconnect."
- "There is a wide range of software available on ARCHER2."
- "The system is based on standard Linux with command line access."
---

## Architecture

The ARCHER2 Cray Shasta system consists of a number of different node types. The ones visible
to users are:

* Login nodes
* Compute nodes
* Data analysis (pre-/post- processing) nodes

All of the node types have the same processors: AMD EPYC Zen2 7742, 2.25GHz, 64-cores. All nodes
are dual socket nodes so there are 128 cores per node.

**TODO** Add diagram of the ARCHER2 architecture

## Compute nodes

There are 5,585 compute nodes in total giving 748,856 compute cores on ARCHER2. There
are 5,300 standard compute nodes with 256 GiB memory per node and 300 high memory 
compute nodes with 512 GiB of memory per node. All of the compute nodes are linked
together using the high-performance Cray Slingshot interconnect.

Access to the compute nodes is controlled by the SLURM scheduling system which supports
both batch jobs and interactive jobs.

Compute node summary:

| Processors | 2x AMD EPYC Zen2 (Rome) 7742, 2.25GHz, 64-core |
| Cores per node | 128 |
| NUMA | 8 NUMA regions per node, 16 cores per NUMA region |
| Memory Capacity | 256/512 GB DDR 3200, 8 memory channels |
| Memory Bandwidth | >380 GB/s per node |
| Interconnect Bandwidth | 25 GB/s per node bi-directional |

## Storage

There are three different storage systems available on ARCHER2:

* Home
* Work
* Solid State

### Home

The home file systems are available on the login nodes only and are designed for the storage
of critical source code and data for ARCHER2 users. They are backed-up regularly offsite for
disaster recovery purposes - restoration of accidentally deleted files is not supported. There is a
total of 1 PB usable space available on the home file systems.

All users have their own directory on the home file systems at:

```
/home/<projectID>/<subprojectID>/<userID>
```

For example, if your username is `auser` and you are in the project `t01` then your main home
directory will be at:

```
/home/t01/t01/auser
```

> ## Subprojects?
> Some large projects may choose to split their resources into multiple subprojects. These 
> subprojects will have identifiers prepended with the main project ID. For example, the
> `rse` subgroup of the `t01` project would have the ID `t01-rse`. If the main project has
> allocated storage quotas to the subproject the directories for this storage will be 
> found at, for example:
> ```
> /home/t01/t01-rse/auser
> ```
> Your default home directory will generally not be changed when you are made a member of 
> a subproject so you must change diectories manually (or change the ownership of files)
> to make use of this different storage quota allocation.
{: .callout}

### Work

The work file systems are available on the home, compute and data analysis nodes, are
designed for high performance parallel access and are the primary location that jobs running on
the compute nodes will read data from and write data to. They are based on the Lustre parallel
file system technology. The work file systems are not backed up in any way. There is a total of 
14.5 PB usable space available on the work file systems.

All users have their own directory on the work file systems at:

```
/work/<projectID>/<subprojectID>/<userID>
```

For example, if your username is `auser` and you are in the project `t01` then your main home
directory will be at:

```
/work/t01/t01/auser
```

> ## Jobs can't see your data?
> If your jobs are having trouble accessing your data make sure you have placed it on work
> rather than home. Remember, the home file systems are not visible from the compute nodes.
{: .callout}

### Solid State

The solid state storage system is available on the compute nodes and is designed for
the highest read and write performance to improve performance of workloads that are I/O bound in
some way. Access to solid state storage resources is controlled through the SLURM scheduling 
system. The solid state storage is not backed up in any way. There is a total of 1.1 PB usable
space available on the solid state storage system.

Data on the solid state storage is transient so all data you require before a job starts or
after a job finishes must be *staged* on to or off of the solid state storage. We discuss how
this works in the Scheduler episode later.

**TODO** Add RDF details.

## System software

The ARCHER2 system runs the *Cray Linux Environment* which is based on SUSE Enterprise Linux.
The service officially supports the *bash* shell for interactive access, shell scripting and
job submission scripts. The scheduling software is SLURM.

As well as the hardware and system software, Cray supply the Cray Programming Environment which
contains:
* Compilers: GCC, Cray Compilers (CCE), AMD Compilers (AOCC)
* Parallel libraries: Cray MPI (MPICH2-based), OpenSHMEM, Global Arrays
* Scientific and numerical libraries: BLAS/LAPACK/BLACS/ScaLAPACK (Cray LibSci, AMD AOCL), FFTW3, HDF5, NetCDF
* Debugging and profiling tools: gdb4hpc, valgrind4hpc, CrayPAT + others
* Optimised Python 3 and R environments:
  * Python 3: numpy, scipy, mpi4py, dask
  * R: standard packages (including "parallel")

On top of the Cray-provided software, the EPCC ARCHER2 CSE service have installed a wide range 
of modelling and simulation software, additional scientific and numeric libraries, data analysis
tools and other useful software.

> ## Licensed software
> For licensed software installed on ARCHER2, users are expected to bring their own licences to
> the service with them. The ARCHER2 service does not provide software licences for use by 
> users.
{: .callout}

More information on the software available on ARCHER2 can be found in
[the ARCHER2 Documentation](https://docs.archer2.ac.uk).

ARCHER2 also supports the use of Singularity containers for single and multi-node jobs.

> ## What about your research?
>
> Speak to your neighbour about your planned use of ARCHER2. Given what you now know about the system,
> what do you think the biggest opportunities are for your research in using ARCHER2? What do you think
> the largest challenges are going to be for you?
{: .challenge}

{% include links.md %}

