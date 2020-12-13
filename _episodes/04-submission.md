---
title: "ARCHER2 scheduler: Slurm"
teaching: 20
exercises: 20
questions:
- "How do I write job submission scripts?"
- "How do I control jobs?"
- "How do I find out what resources are available?"
objectives:
- "Understand the use of the basic Slurm commands."
- "Know what components make up and ARCHER2 scheduler."
- "Know where to look for further help on the scheduler."
keypoints:
- "ARCHER2 uses the Slurm scheduler."
- "`srun` is used to launch parallel executables in batch job submission scripts."
- "There are a number of different partitions (queues) available."
---

ARCHER2 uses the Slurm job submission system, or *scheduler*, to manage resources and how they are made
available to users. The main commands you will use with Slurm on ARCHER2 are:

* `sinfo`: Query the current state of nodes
* `sbatch`: Submit non-interactive (batch) jobs to the scheduler
* `squeue`: List jobs in the queue
* `scancel`: Cancel a job
* `salloc`: Submit interactive jobs to the scheduler
* `srun`: Used within a batch job script or interactive job session to start a parallel program

Full documentation on Slurm on ARCHER2 can be found in [the *Running Jobs on ARCHER2* section of the User
and Best Practice Guide](https://docs.archer2.ac.uk/user-guide/scheduler.html).

## Finding out what resources are available: `sinfo`

The `sinfo` command shows the current state of the compute nodes known to the scheduler:

```
auser@login01-nmn:~> sinfo
```
{: .language-bash}
```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
standard     up 1-00:00:00     60  down* nid[001006,001033,001045-001047,001061,001068,001074,001109,001125,001138,001149,001163,001171,001227-001228,001241,001255,001262,001273,001287,001326,001336,001347,001366,001369,001395,001435,001462,001478,001490,001505,001539,001546,001552,001581,001614,001642,001644-001645,001647,001652,001664,001669,001709,001719,001723,001729,001747,001751,001757,001810,001817,001839,001903,001919,001932,001950,001955,002014]
standard     up 1-00:00:00     11  drain nid[001016,001069,001092,001468,001520-001521,001812,001833-001835,001838]
standard     up 1-00:00:00      5   resv nid[001001-001004,001021]
standard     up 1-00:00:00    565  alloc nid[001000,001005,001007-001015,001018-001020,001022-001032,001034-001044,001048-001060,001062-001067,001070-001073,001075-001091,001093-001108,001110-001124,001126-001137,001139-001148,001150-001155,001158-001162,001164-001170,001172-001226,001229-001240,001242-001254,001256-001261,001263-001272,001274-001286,001288-001317,001319-001325,001327-001335,001337-001346,001348-001365,001367-001368,001370-001394,001396-001434,001436-001461,001463-001467,001469-001477,001491-001504,001547-001551,001553-001580,001582-001613,001615-001641,001648-001651,001653-001663,001665-001668,001951-001954]
standard     up 1-00:00:00    380   idle nid[001017,001156-001157,001479-001489,001506-001519,001522-001538,001540-001545,001643,001646,001670-001688,001690-001708,001710-001718,001720-001722,001724-001728,001730-001746,001748-001750,001752-001756,001758-001809,001811,001813-001816,001818-001824,001826-001832,001836-001837,001840-001902,001904-001918,001920-001931,001933-001949,001956-002013,002015-002023]
```
{: .output}

There is a row for each node state and partition combination. The default output shows the following columns:

* `PARTITION` - The system partition
* `AVAIL` - The status of the partition - `up` in normal operation
* `TIMELIMIT` - Maximum runtime as `days-hours:minutes:seconds`: on ARCHER2, these are set using *QoS*
  (Quality of Service) rather than on partitions
* `NODES` - The number of nodes in the partition/state combination
* `STATE` - The state of the listed nodes (more information below)
* `NODELIST` - A list of the nodes in the partition/state combination

The nodes can be in many different states, the most common you will see are:

* `idle` - Nodes that are not currently allocated to jobs
* `alloc` - Nodes currently allocated to jobs
* `draining` - Nodes draining and will not run further jobs until released by the systems team
* `down` - Node unavailable
* `fail` - Node is in fail state and not available for jobs
* `reserved` - Node is in an advanced reservation and is not generally available
* `maint` - Node is in a maintenance reservation and is not generally available

If you prefer to see the state of individual nodes, you can use the `sinfo -N -l` command.

> ## Lots to look at!
> Warning! The `sinfo -N -l` command will produce a lot of output as there are over 1000 individual 
> nodes on the current ARCHER2 system!
{: .callout}

```
auser@login01-nmn:~> sinfo -N -l
```
{: .language-bash}
```
Fri Jul 10 09:45:54 2020
NODELIST   NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON              
nid001001      1    standard        idle  256   2:64:2 244046        0      1   (null) none                
nid001002      1    standard        idle  256   2:64:2 244046        0      1   (null) none                
nid001003      1    standard        idle  256   2:64:2 244046        0      1   (null) none                
nid001004      1    standard        idle  256   2:64:2 244046        0      1   (null) none                
nid001005      1    standard        idle  256   2:64:2 244046        0      1   (null) none                
nid001006      1    standard        idle  256   2:64:2 244046        0      1   (null) none                
nid001007      1    standard        idle  256   2:64:2 244046        0      1   (null) none                
nid001008      1    standard        idle  256   2:64:2 244046        0      1   (null) none  

...lots of output trimmed...

```
{: .output}

> ## Explore a compute node
> Let’s look at the resources available on the compute nodes where your jobs will actually run. Try running this
> command to see the name, CPUs and memory available on the worker nodes (the instructors will give you the ID of
> the compute node to use):
> ```
> [auser@login01-nmn:~> sinfo -n nid001005 -o "%n %c %m"
> ```
> {: .language-bash}
> This should display the resources available for a standard node. Can you use `sinfo` to find out the range of
> node IDs for the high memory nodes?
> > ## Solution
> > The high memory nodes have IDs `nid001001-nid001004`. You can get this by using:
> >
> > ```
> > auser@login01-nmn:~> sinfo -N -l -S "-m" | less
> > ```
> > {: .language-bash}
> >
> > The `-S "-m"` option tells `sinfo` to print the node list sorted by decreasing memory per node. This
> > output is then piped into `less` so we can examine the output a page at a time without it scrolling
> > off the screen.
> >
> {: .solution}
> It is also possible to search nodes by state. Can you find all the free nodes in the system?
> > ## Solution
> > `sinfo` lets you specify the state of a node to search for, so to get all the free nodes in the system you can use:
> > ```
> > sinfo -N -l --state=idle
> > ```
> > More information on what `sinfo` can display can be found in the `sinfo` manual page, i.e. `man sinfo`
> {: .solution}

{: .challenge}

## Using batch job submission scripts

### Header section: `#SBATCH`

As for most other scheduler systems, job submission scripts in Slurm consist of a header section with the
shell specification and options to the submission command (`sbatch` in this case) followed by the body of
the script that actually runs the commands you want. In the header section, options to `sbatch` should 
be prepended with `#SBATCH`.

Here is a simple example script that runs the `xthi` program, which shows
process and thread placement. Here we consider only MPI and assume there
is no OpenMP involved.

The intention is to run using two nodes (nodes will always be allocated on
and exclusive basis) and use 128 MPI tasks per node (i.e., one per
physical core).

```
#!/bin/bash

#SBATCH --partition=standard
#SBATCH --qos=standard

#SBATCH --job-name=my_mpi_job
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128

#SBATCH --time=00:10:00

# This module needs to be loaded in ALL scripts
module load epcc-job-env

# Load the "xthi" module
module load xthi

# srun to launch the executable
srun  xthi
```
{: .language-bash}

The options shown here are:

* `--partition=standard` - Submit to the standard set of nodes
* `--qos=standard` - Submit with the standard quality of service settings.
* `--job-name=my_mpi_job` - Set the name for the job that will be displayed in Slurm output
* `--nodes=2` - Select two nodes for this job
* `--ntasks-per-node=128` - Set 128 parallel processes per node (usually corresponds to MPI ranks)
* `--time=00:10:00` - Set 10 minutes maximum walltime for this job

We will discuss the `srun` command further below.

### Submitting jobs using `sbatch`

You use the `sbatch` command to submit job submission scripts to the scheduler.
For example, if the above script was saved in a file called `test_job.slurm`,
you would submit it with:

```
auser@login01-nmn:~> sbatch test_job.slurm
```
{: .language-bash}
```
Submitted batch job 23996
```
{: .output}

Slurm reports back with the job ID for the job you have submitted

> ## What are the default for `sbatch` options?
> Make sure you can submit the batch script example given above.
> By removing specific `sbatch` options, can you work out the
> default values for:
> 
> 
> 1. The number of nodes used by the job?
> 2. The number of tasks per node?
> 3. The wall time limit? (Hint: you need the command: `sacct`)
> 4. What options cannot be omitted without error?
> 
> > ## Solution
> > 
> > (1) If `--nodes` is omitted, the default is 1 node.
> > 
> > (2) If `--ntasks-per-node` is omitted, the default is 1 task per node.
> >
> > (3) Getting the default time limit is more difficult. We need to use the
> >     `sacct` command to query the time limit set for the job.
> > For example, if the job ID was "12345", then we could query the time limit with:
> > 
> > ```
> > auser@login01-nmn:~> sacct -o "TimeLimit" -j 12345
> > ```
> > {: .language-bash}
> > ```
> >  Timelimit 
> > ---------- 
> >   01:00:00
> > ```
> > {: .output}
> > showing that the default time limit is 1 hour.
> >
> > (4) The `--partition` and `--qos` options must be specified. An error
> >     will be generated at the point of submission if either is omitted.
> {: .solution}
{: .challenge}

### Checking progress of your job with `squeue`

You use the `squeue` command to show the current state of the queues on
ARCHER2. Without any options, it will show all jobs in the queue:

```
auser@login01-nmn:~> squeue
```
{: .language-bash}
```
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```
{: .output}

### Cancelling jobs with `scancel`

You can use the `scancel` command to cancel jobs that are queued or running. When used on running jobs
it stops them immediately.

<!-- Great content, not currently available
> ## Getting notified
> Slurm on ARCHER2 can also send e-mails to notify you when your job starts, ends, fails, etc. Can
> you find out how you would setup your job script to send you an e-mail when your job finishes and
> when it fails? Test your answer, does it work?
> > ## Solution
> > The option `--mail-type=END,FAIL` will send mailings to you when the job ends or fails. You can
> > also use the event `TIME_LIMIT` to notify you if a job reaches its walltime without finishing and
> > the events `TIME_LIMIT_50`, `TIME_LIMIT_80` and `TIME_LIMIT_90` to notify you when your job is
> > 50%, 80% and 90% of the way through the specified walltime.
> {: .solution}
{: .challenge}
-->

### Running parallel applications using `srun`

A submission script consists of standard shell commands: those
required to run your job. These can be simple or complex depending on how
you run your jobs, but even the simplest job
script usually contains commands to:

* Load the required software modules
* Set appropriate environment variables

After this you will usually launch your parallel program using the `srun`
command. ``srun`` uses information provided by ``sbatch`` options to
determine the configuration of the launch: the number of MPI tasks, their
placemant, and so on.

In the example above, our `srun` command simply looks like:
```
srun xthi
```

If we wish to control the placement explicitly, we can use the ``srun``
option
```
srun --cpu_bind=type
```
where different values of *type* are avaialble.

{: .language-bash}

> ## An MPI program with ``--cpu_bind=rank``
> 
> Return to our simple example using the `xthi` example. If we wished to
> bind a given MPI rank to the corresponding core, we could use the
> following script:
>
> ```
> #!/bin/bash
> 
> #SBATCH --partition=standard
> #SBATCH --qos=standard
>
> #SBATCH --time=00:10:00
> #SBATCH --nodes=2
> #SBATCH --ntasks-per-node=128
>
> # This module needs to be loaded in ALL scripts
> module load epcc-job-env
>
> # Now load the "xthi" package
> module load xthi
>
> srun --cpu_bind=rank xthi
> ```
> 
> 1. Check you can run the script and see how the placement differs from
>    the previous attempt in the first exercise.
> 2. What happens if one tries fewer MPI tasks per node, e.g.,
>    ``--ntasks-per-node=8``
> 
> 
> > ## Solution
> > 1. You should see affinity now matches the rank 
> > 2. You should see that tasks 0-7 remain bound to cores 0-7
> {: .solution}
{: .challenge}


### MPI programs using fewer than 128 cores per node

Suppose we have an MPI program (still no OpenMP threads) and we wish to
run with 8 MPI tasks per node with one MPI task per NUMA region (this
might be appropriate is each MPI task has a significant memory footprint,
for example).

Explicit control of the binding is this case is possible via
`--cpu_bin=map_cpu:list`, where the *list* specficies which
cores consecutive MPI tasks will run on. For example, for 1 MPI
task per NUMA region, we could specify:
```
srun --cpu_bind=map_cpu:0,16,32,48,64,80,96,112 xthi
```
This specifies the pattern on a per-node basis.

> ## Fewer tasks then cores
> 
> 
> 1. Check the above `--cpu_bind=map_cpu` option for `srun` produces
>    the expected result when requesting 8 MPI tasks per node.
> 2. What is the appropriate option if you want 1 MPI task per socket?
> 
> Check that the binding of tasks to 
> nodes and cores output by `xthi` is what you expect.
> 
> > ## Solution
> > 1. The affinity should match the map selected (0,16,32,48,64,80,96,112) on
> >    each node.
> > 2. `srun --cpu_bind=map_cpu:0,64 xthi`
> {: .solution}
{: .challenge}


### Hybrid MPI and OpenMP jobs

Consider a hybrid job with both MPI (the individual tasks can also
be referred to as ranks or processes) and OpenMP
(with multiple threads). Here, we need to allocate an appropriate number
of cores to each MPI task so that the relevant number of threads can be
accommodated by `srun`.

As we saw above, you can use the options to `sbatch` to control how many
parallel tasks are placed on each compute node. The number of cores
(strictly, the number of CPUs in SLURM) per MPI task is then set with
the option
```
--cpus-per-task
```
This sets the stride  between parallel tasks to the right value to accommodate
the OpenMP threads - the value for `--cpus-per-task` should usually be the
same as that for `OMP_NUM_THREADS`.

As an example, consider the job script below that runs on 2 nodes with 16
MPI tasks per node and 8 OpenMP threads per MPI task. The aim here is to
obtain one thread per physical core.

Here we use the standard OpenMP control setting `OMP_PLACES=cores`
to specify that placement should be on the basis of cores (each
core accommodating one or more hardware threads). Note there
is no `--cpu_bind` options to `srun`.

```
#!/bin/bash

#SBATCH --partition=standard
#SBATCH --qos=standard

#SBATCH --time=00:10:00
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=16

#SBATCH --cpus-per-task=8

module load epcc-job-env
module load xthi

export OMP_NUM_THREADS=8
export OMP_PLACES=cores

srun xthi
```
{: .language-bash}


> ## Hybrid MPI/OpenMP jobs
>
> Using the above script, check the placement of tasks/threads. You
> should find that SLURM has allocated 2 threads to each physical
> core.
>
> We need a way to tell SLURM to ignore the presence of the additional
> "CPUs' (128-255) and just use physical cores (0-127). This can be
> done with the `--hint=nomultithread` option.
>
> Add this a check what has happened.
>
> The result should now look more reasonable, except you may notice
> that MPI tasks have been distributed to cores within nodes in a
> cyclic fashion. The final step is to change to a block distribution
> via `--distribution=block:block`. This is the distribition on
> between nodes and within nodes, the default being `block:cyclic`.
>
> The solution shows the final script:
>> ## Solution
>> ```
>> #!/bin/bash
>>
>> #SBATCH --partition=standard
>> #SBATCH --qos=standard
>>
>> #SBATCH --time=00:10:00
>> #SBATCH --nodes=2
>> #SBATCH --ntasks-per-node=16
>>
>> #SBATCH --cpus-per-task=8
>> #SBATCH --hint=nomultithread
>> #SBATCH --distribution=block:block
>>
>> # This module needs to be loaded in ALL scripts
>> module load epcc-job-env
>> 
>> # Now load the "xthi" package
>> module load xthi
>>
>> export OMP_NUM_THREADS=8
>> export OMP_PLACES=cores
>>
>> srun xthi
>> ```
> {: .solution}
{: .challenge}

Each compute node is made up of 8 NUMA (*Non Uniform Memory Access*) regions
(4 per socket)  with physical 16 cores in each region. Programs where the
threads of a task span more than one NUMA region may be *less* efficient,
so we recommend using thread counts that fit well into the
ARCHER2 compute node layout. Effectively, this means one of the following
options for nodes where all cores are used:

* 8 MPI tasks per node and 16 OpenMP threads per task: equivalent to 1 MPI task per NUMA region
* 16 MPI tasks per node and 8 OpenMP threads per task: equivalent to 2 MPI tasks per NUMA region
* 32 MPI tasks per node and 4 OpenMP threads per task: equivalent to 4 MPI tasks per NUMA region
* 64 MPI tasks per node and 2 OpenMP threads per task: equivalent to 8 MPI tasks per NUMA region 


## STDOUT/STDERR from jobs

STDOUT and STDERR from jobs are, by default, written to a file called `slurm-<jobid>.out` in the
working directory for the job (unless the job script changes this, this will be the directory
where you submitted the job). So for a job with ID `12345` STDOUT and STDERR would be in
`slurm-12345.out`.

If you run into issues with your jobs, the Service Desk will often ask you to send your job
submission script and the contents of this file to help debug the issue.

If you need to change the location of STDOUT and STDERR you can use the `--output=<filename>`
and the `--error=<filename>` options to `sbatch` to split the streams and output to the named
locations.

## Other useful information

In this section we briefly introduce other scheduler topics that may be useful to users. We
provide links to more information on these areas for people who may want to explore these 
areas more. 

### Interactive jobs: `salloc` 

Similar to the batch jobs covered above, users can also run interactive jobs using the Slurm
command `salloc`. `salloc` takes the same arguments as `sbatch` but, obviously, these are 
specified on the command line rather than in a job submission script.

Once the job requested with `salloc` starts, you will be returned to the command line 
and can now start parallel jobs on the compute nodes interactively with the `srun` command
in the same way as you would within a job submission script.

For example, to execute `xthi` across all cores on two nodes (1 MPI task per core and no
OpenMP threading) within an interactive job you would issue the following commands:

```
auser@login01-nmn:~> salloc --nodes=2 --ntasks-per-node=128 --cpus-per-task=1 --time=0:10:0 --account=t01
salloc: Granted job allocation 24236
auser@login01-nmn:~> module load xthi
auser@login01-nmn:~> srun xthi
```
{: .language-bash}
```
Hello from rank 242, thread 0, on nid001002. (core affinity = 46,174)
Hello from rank 249, thread 0, on nid001002. (core affinity = 31,159)
Hello from rank 225, thread 0, on nid001002. (core affinity = 28,156)
Hello from rank 231, thread 0, on nid001002. (core affinity = 124,252)
Hello from rank 233, thread 0, on nid001002. (core affinity = 29,157)
Hello from rank 234, thread 0, on nid001002. (core affinity = 45,173)
Hello from rank 240, thread 0, on nid001002. (core affinity = 14,142)
Hello from rank 246, thread 0, on nid001002. (core affinity = 110,238)
Hello from rank 248, thread 0, on nid001002. (core affinity = 15,143)
Hello from rank 251, thread 0, on nid001002. (core affinity = 63,191)
Hello from rank 252, thread 0, on nid001002. (core affinity = 79,207)
Hello from rank 223, thread 0, on nid001002. (core affinity = 123,251)
Hello from rank 71, thread 0, on nid001001. (core affinity = 120,248)
Hello from rank 227, thread 0, on nid001002. (core affinity = 60,188)
Hello from rank 243, thread 0, on nid001002. (core affinity = 62,190)
Hello from rank 250, thread 0, on nid001002. (core affinity = 47,175)
Hello from rank 53, thread 0, on nid001001. (core affinity = 86,214)

...long output trimmed...
```
{: .output}

Once you have finished your interactive commands, you exit the interactive job with `exit`:

```
auser@login01-nmn:~> exit
exit
salloc: Relinquishing job allocation 24236
auser@login01-nmn:~>
```
{: .language-bash}

<!-- Need to add information on the solid state storage and Slurm once it is in place

### Using the ARCHER2 solid state storage

-->


{% include links.md %}


