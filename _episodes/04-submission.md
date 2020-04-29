---
title: "ARCHER2 scheduler: SLURM"
teaching: 20
exercises: 10
questions:
- "How do I write job submission scripts?"
- "How do I control jobs?"
- "How do I find out what resources are available?"
objectives:
- "Understand the use of the basic SLURM commands."
- "Know what components make up and ARCHER2 scheduler."
- "Know where to look for further help on the scheduler."
keypoints:
- "ARCHER2 uses the SLURM scheduler."
- "`srun` is used to launch parallel executables in batch job submission scripts."
- "There are a number of different partitions (queues) available."
---

ARCHER2 uses the SLURM job submission system, or *scheduler*, to manage resources and how they are made
available to users. The main commands you should will use with SLURM on ARCHER2 are:

* `sinfo`: Query the current state of nodes
* `sbatch`: Submit non-interactive (batch) jobs to the scheduler
* `squeue`: List jobs in the queue
* `scancel`: Cancel a job
* `salloc`: Submit interactive jobs to the scheduler
* `srun`: Used within a batch job script or interactive job session to start a parallel program

Full documentation on SLURM on ARCHER2 cn be found in [the *Running Jobs on ARCHER2* section of the User
and Best Practice Guide](https://docs.archer2.ac.uk/user-guide/scheduler.html).

## Finding out what resources are available: `sinfo`

The `sinfo` command shows the current state of the compute nodes known to the scheduler:

```
[auser@archer2-login1 ~]$ sinfo
```
{: .language-bash}
```
PARTITION       AVAIL  TIMELIMIT  NODES  STATE NODELIST
standard           up 2-00:00:00      1  fail* cn580
standard           up 2-00:00:00    128  down$ cn[96,579,793,814,1025-1044,1081-1088]
standard           up 2-00:00:00     26  maint cn[27,93-95,206,232,310,492,568,577-578,585-588,813,815-816,818,846,889,921-924,956]
standard           up 2-00:00:00      2   fail cn[274,871]
standard           up 2-00:00:00      4  down* cn[528,614,637,845]
standard           up 2-00:00:00   1034  alloc cn[1-26,28-38,40-58,62-86,88-92,97-174,176-205,207-231,233-273,275-309,311-333,335-341,344-371,373-376,378-413,415-452,454-489,493-513,515-527,529-532,535-539,541-550,554-561,563-567,569,572-576,581-584,589-595,598-601,603-613,615,617-620,623-631,633-636,638-647,651-659,661-678,680-687,690-695,697-716,718-736,738-775,777-790,792,794-812,817,819-844,847-852,854-870,872-888,890-920,925-955,957-977,980-1014,1016-1020,1023-1024,1045,1047-1070,1072-1080,1089-1105,1107-1152]
standard           up 2-00:00:00     26   idle cn[61,490-491,540,551-552,562,570,596,602,621,632,648-650,660,688-689,696,853,978-979,1015,1021-1022,1071]
```
{: .output}

There is a row for each node state and partition combination. The default output shows the following columns:

* `PARTITION` - The system partition (**TODO** confirm partitions on ARCHER2)
* `AVAIL` - The status of the partition - `up` in normal operation
* `TIMELIMIT` - Maximum runtime as `days-hours:minutes:seconds`
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

> The `sinfo -N -l` command will produce a lot of output as there are over 5000 individual 
> nodes on ARCHER2!
{: .callout}

```
[auser@archer2-login1 ~]$ sinfo -N -l
```
{: .language-bash}
```
NODELIST           NODES       PARTITION       STATE  CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON  
    cn1                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn2                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn3                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn4                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn5                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn6                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn7                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn8                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn9                1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn10               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn11               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn12               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn13               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn14               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn15               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn16               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn17               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn18               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn19               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn20               1        standard    allocated  128   2:64:1 256800        0      5  standard none                
    cn21               1        standard    allocated  128   2:64:1 256800        0      5  standard none 

...lots of output trimmed...

```
{: .output}

> ## Explore a compute node
> Let’s look at the resources available on the compute nodes where your jobs will actually run. Try running this
> command to see the name, CPUs and memory available on the worker nodes (the instructors will give you the ID of
> the compute node to use):
> ```
> [auser@archer2-login1 ~]$ sinfo -n cn1 -o "%n %c %m"
> ```
> {: .language-bash}
> This should display the resources available for a standard node. Can you use `sinfo` to find out the range of
> node IDs for the high memory nodes?
> > ## Solution
> > The high memory nodes have IDs `cn100-cn300`. You can get this by using:
> > ```
> > [auser@archer2-login1 ~]$ sinfo -N -l | grep 513200
> > ```
> > {: .language-bash}
> {: .solution}
{: .challenge}

## Using batch job submission scripts

### Header section: `#SBATCH`

As for most other scheduler systems, job submission scripts in SLURM consist of a header section with the
shell specification and options to the submission command (`sbatch` in this case) followed by the body of
the script that actually runs the commands you want. In the header section, options to `sbatch` should 
be prepended with `#SBATCH`.

Here is a simple example script that runs the `xthi` program that shows process and thread placement across
two nodes.

**TODO** Check options. Is `--exclusive` needed?

```
#!/bin/bash
#SBATCH -job-name=my_mpi_job
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128
#SBATCH --time=0:10:0
#SBATCH --account=t01

module load xthi

export OMP_NUM_THREADS=1

# Load modules, etc.
# srun to launch the executable
srun --nodes=2 --ntasks-per-node=128 xthi
```
{: .language-bash}

The options shown here are:

* `--job-name=my_mpi_job` - Set the name for the job that will be displayed in SLURM output
* `--nodes=2` - Select two nodes for this job
* `--ntasks-per-node=128` - Set 128 parallel processes per node (usually corresponds to MPI ranks) 
* `--time=0:10:0` - Set 10 minutes maximum walltime for this job
* `--account=t01` - Charge the job to the `t01` budget

### Submitting jobs using `sbatch`

You use the `sbatch` command to submit job submission scripts to the scheduler. For example, if the
above script was saved in a file called `test_job.slurm`, you would submit it with:

```
[auser@archer2-login1 ~]$ sbatch test_job.slurm
```
{: .language-bash}
```
Submitted batch job 23996
```
{: .output}

SLURM reports back with the job ID for the job you have submitted

### Checking progress of your job with `squeue`

You use the `squeue` command to show the current state of the queues on ARCHER2. Without any options, it
will show all jobs in the queue:

```
[auser@archer2-login1 ~]$ squeue
```
{: .language-bash}
```
```
{: .output}

### Cancelling jobs with `scancel`

You can use the `scancel` command to cancel jobs that are queued or running. When used on running jobs
it stops them immediately.

> ## What are the default for `sbatch` options?
> If you do not specify job options, what are the defaults for SLURM on ARCHER2? Submit jobs to find out
> what the defaults are for:
> 
> Number of nodes?
> 
> Tasks per node?
> 
> Walltime?
> 
> Budget (or Account) the job is charged to?
> > ## Solution
> > Defaults are:
> > 
> > Number of nodes: 1
> > 
> > Tasks per node: 1
> > 
> > Walltime: 5 minutes
> > 
> > Budget: None - fails if submitted without a budget specified
> {: .solution}
{: .challenge}

**TODO** Check defaults for these answers

> ## Getting notified
> SLURM on ARCHER2 can also send e-mails to notify you when your job starts, ends, fails, etc. Can
> you find out how you would setup your job script to send you an e-mail when your job finishes and
> when it fails? Test your answer, does it work?
> > ## Solution
> > The option `--mail-type=END,FAIL` will send mailings to you when the job ends or fails. You can
> > also use the event `TIME_LIMIT` to notify you if a job reaches its walltime without finishing and
> > the events `TIME_LIMIT_50`, `TIME_LIMIT_80` and `TIME_LIMIT_90` to notify you when your job is
> > 50%, 80% and 90% of the way through the specified walltime.
> {: .solution}
{: .challenge}

### Running parallel applications using `srun`

Once past the header section your script consists of standard shell commands required to run your
job. These can be simple or complex depending on how you run your jobs but even the simplest job
script usually contains commands to:

* Load the required software modules
* Set appropriate environment variables (you should always set `OMP_NUM_THREADS`, even if you are
  not using OpenMP you should set this to `1`)

After this you will usually launch your parallel program using the `srun` command. At its simplest,
`srun` does not need any arguments (it will use the values supplied to `sbatch` to work out how many
parallel processes to launch) (**TODO** check this is really the case) but it is usually good 
practice to explicitly set the number of tasks and the number of tasks per node to ensure that you
are getting the parallel distribution you expect. In the example above, we want to use all the 
cores on 4 nodes; as there are 128 cores per node this gives us 512 tasks in total with 128 tasks
per node. Our `srun` command looks like:

```
srun --nodes=4 --ntasks-per-node=128 xthi
```
{: .language-bash}

> ## Underpopulation of nodes
> You may often want to *underpopulate* nodes on ARCHER2 to access more memory or more memory 
> bandwidth per task. Can you state the `srun` command and options you would use to run `xthi`:
> 
> On 4 nodes with 64 tasks per node?
> 
> On 8 nodes with 2 tasks per node, 1 task per socket?
> 
> On 4 nodes with 32 tasks per node, ensuring an even distribution across the 8 NUMA regions
> on the node?
> 
> Once you have your answers run them in job scripts and check that the binding of tasks to 
> nodes and cores output by `xthi` is what you expect.
> 
> > ## Solution
> > `srun --nodes=4 --ntasks-per-node=64 xthi`
> >
> > `srun --nodes=8 --ntasks-per-node=2 --ntasks-per-socket=1 xthi`
> >
> > `srun --nodes=4 --ntasks-per-node=32 --ntasks-per-socket=16 --cores-per-task=4 xthi`
> {: .solution}
{: .challenge}

### Hybrid MPI and OpenMP jobs

When running hybrid MPI (with the individual tasks also known as ranks or processes) and OpenMP
(with multiple threads) jobs you need to leave free cores between the parallel tasks launched
using `srun` for the multiple OpenMP threads that will be associated with each MPI task.

As we saw above, you can use the options to `srun` to control how many parallel tasks are
placed on each compute node and can use the `--cores-per-task` option to set the stride 
between parallel tasks to the right value to accommodate the OpenMP threads - the value
for `--cores-per-task` should usually be the same as that for `OMP_NUM_THREADS`.

As an example, consider the job script below that runs across 2 nodes with 8 MPI tasks
per node and 16 OpenMP threads per MPI task (so all 256 cores across both nodes are used).

```
#!/bin/bash
#SBATCH -job-name=my_hybrid_job
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128
#SBATCH --time=0:10:0
#SBATCH --account=t01

module load xthi

export OMP_NUM_THREADS=16

# Load modules, etc.
# srun to launch the executable
srun --nodes=2 --ntasks-per-node=8 --cores-per-task=${OMP_NUM_THREADS} xthi
```
{: .language-bash}

Each ARCHER2 compute node is made up of 8 NUMA (*Non Uniform Memory Access*) regions (4 per socket) 
with 16 cores in each region. Programs where the threads of a task span multiple NUMA regions
are likely to be *less* efficient so we recommend using thread counts that fit well into the
ARCHER2 compute node layout. Effectively, this means one of the following options for nodes
where all cores are used:

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

Similar to the batch jobs covered above, users can also run interactive jobs using the SLURM
command `salloc`. `salloc` takes the same arguments as `sbatch` but, obviously, these are 
specified on the command line rather than in a job submission script.

Once you the job requested with `salloc` starts, you will be returned to the command line 
and can now start parallel jobs on the compute nodes interactively with the `srun` command
in the same way as you would within a job submission script.

For example, to execute `xthi` across all cores on two nodes (1 MPI task per core and no
OpenMP threading) within an interactive job you would issue the following commands:

```
[auser@archer2-login1 ~]$ salloc --nodes=2 --ntasks-per-node=128 --time=0:10:0 --account=t01
salloc: Granted job allocation 24236
salloc: Waiting for resource configuration
salloc: Nodes cn13 are ready for job
[auser@archer2-login1 ~]$ module load xthi
[auser@archer2-login1 ~]$ srun --nodes=2 --ntasks-per-node=128 xthi
```
{: .language-bash}
```
**TODO** Add output from xthi
```
{: .output}

Once you hav finished your interactive commands, you exit the interactive job with `exit`:

```
[auser@archer2-login1 ~]$ exit
exit
salloc: Relinquishing job allocation 24236
[auser@archer2-login1 ~]$
```
{: .language-bash}

### Using the ARCHER2 solid state storage

**TODO** Add information on how to use solid state storage once it is known

{% include links.md %}


