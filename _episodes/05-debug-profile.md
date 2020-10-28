---
title: "Debugging and profiling on ARCHER2"
teaching: 30
exercises: 20
questions:
- "What debugging tools are available on ARCHER2 and how can I access them?"
- "What profiling tools are available on ARCHER2 and how can I access them?"
- "Where can I find more documentation on and get help with these tools?"
objectives:
- "Know what tools are available to help you debug and profile parallel applications on ARCHER2."
- "Know where to get further help."
keypoints:
- "The main debugging tool on ARCHER2 is *gdb4hpc*"
- "The main profiling tool on ARCHER2 is *CrayPat*"
---

ARCHER2 has a range of debugging and profiling software available. In this section we provide a brief
overview of the tools available, their applicability and links to more information. A detailed tutorial
on the use of these tools is beyond the scope of this course but if you are interested in this, then 
you may be interested in the following courses offered by the ARCHER2 service:

* Performance Analysis Workshop

## Debugging tools overview

The following debugging tools are available on ARCHER2:

* **gdb4hpc** is a command-line tool working similarly to [gdb](https://www.gnu.org/software/gdb/) 
  that allows users to debug parallel programs. It can launch parallel programs or attach to ones
  already running and allows the user to step through the execution to identify the causes of any
  unexpected behaviour. Available via ``module load gdb4hpc``.
* **valgrind4hpc** is a parallel memory debugging tool that aids in detection of memory leaks and
  errors in parallel applications. It aggregates like errors across processes and threads to simply
  debugging of parallel appliciations.
%* **STAT** generate merged stack traces for parallel applications. Also has visualisation tools.
%* **ATP** scalable core file and backtrace analysis when parallel programs crash.
%* **CCDB** Cray Comparative Debugger. Compare two versions of code side-by-side to analyse differences.

See [the Cray Performance Measurement and Analysis Tools User Guide](https://pubs.cray.com/bundle/Cray_Performance_Measurement_and_Analysis_Tools_User_Guide_644_S-2376/page/About_the_Cray_Performance_Measurement_and_Analysis_Tools_User_Guide.html)

## Using gdb4hpc to debug an application

For this exercise, we'll be debugging a short program using gdb4hpc. To start, we'll grab a copy of a buggy code from ARCHER2:

```bash
wget {{site.github.repository_url}}/gh_pages/files/gdb4hpc_exercise.c?raw=true
```

You can look at the code if you want -- you might even be able to debug it by inspection (but that defeats the purpose of this exercise). When you're ready, compile the code using the C compiler wrappers and the debugging flag `-g`:

```bash
 cc -g gdb4hpc_exercise.c -o my_exe
 ```

You can choose a different name for your executable, but I'll be using `my_exe` through this exercise for consistency -- if you use a different name, make the appropriate change wherever you see `my_exe`.

We'll be using ``gdb4hpc`` to go through this program and see where errors might arise.

Load and launch ``gdb4hpc``:

```bash
 module load gdb4hpc
 gdb4hpc
```
    
You will get some information about this version of the program and, eventually, you will get a command prompt:

```
 gdb4hpc 4.5 - Cray Line Mode Parallel Debugger
 With Cray Comparative Debugging Technology.
 Copyright 2007-2019 Cray Inc. All Rights Reserved.
 Copyright 1996-2016 University of Queensland. All Rights Reserved.
 Type "help" for a list of commands.
 Type "help <cmd>" for detailed help about a command.
 dbg all>
```
  
We will use ``launch`` to start an application within gdb4hpc. For now, we want to run our simulation on a single process, so we will type:

```bash
 dbg all> launch $my_prog{1} ./my_exe
```
    
This will launch an ``srun`` job on one of the compute nodes. The name `my_prog` is a dummy name to which this run-through of the program is linked -- you will not be able to launch another program using this name, and you can use any name you want instead. The number in the brackets ``{1}`` indicates the number of processes this job will be using (it's  1 here). You could use a larger number if you wanted. If you call for more processes than available on a single compute node, `gdb4hpc` will launch the program on an appropriate number of nodes. Note though that the more cores you ask for, the slower `gdb4hpc` will be.

Once the program is launched, gdb4hpc will load up the program and begin to run it. You will get output to screen something that looks like:

```
Starting application, please wait...
Creating MRNet communication network...
Waiting for debug servers to attach to MRNet communications network...
Timeout in 400 seconds. Please wait for the attach to complete.
Number of dbgsrvs connected: [0];  Timeout Counter: [1]
Number of dbgsrvs connected: [0];  Timeout Counter: [2]
Number of dbgsrvs connected: [1];  Timeout Counter: [0]
Finalizing setup...
Launch complete.
my_prog{0}: Initial breakpoint, main at /PATH/TO/gdb4hpc_exercise.c:9
```
    
The line number at which the initial breakpoint is made (in the above example, line 7) corresponds to the first line within the `main` function.

Once the code is loaded, you can use various commands to move through your code. The following lists and describes some of the most useful ones:

* ``help`` -- Lists all gdb4hpc commands. You can run ``help COMMAND_NAME`` to learn more about a specific command (*e.g.* ``help launch`` will tell you about the launch command
* ``list`` -- Will show the current line of code and the 9 lines following. Repeated use of ``list`` will move you down the code in ten-line chunks.
* ``next`` -- Will jump to the next step in the program for each process and output which line of code each process is on. It will not enter subroutines. Note that there is no reverse-step in gdb4hpc.
* ``step`` -- Like ``next``, but this will step into subroutines.
* ``up`` -- Go up one level in the program (*e.g.* from a subroutine back to main).
* ``print var`` -- Prints the value of variable ``var`` at this point in the code.
* ``watch var`` -- Like print, but will print whenever a variable changes value.
* ``quit`` -- Exits gdb4hpc.

For now, we will look at `list`, `next`, `print`, and `watch`. Running:

```bash
 dbg all> list
```

should output the first 10 lines of `main`:

```
 my_prog{0}: 9	
 my_prog{0}: 10	  // Initiallise MPI environment
 my_prog{0}: 11	  MPI_Init(NULL,NULL);
 my_prog{0}: 12	
 my_prog{0}: 13	  // Get processor rank
 my_prog{0}: 14	  int rank;
 my_prog{0}: 15	  MPI_Comm_rank(MPI_COMM_WORLD,&rank);
 my_prog{0}: 16	
 my_prog{0}: 17	  int count = rank + 1;
 my_prog{0}: 18	
```

Repeating `list` will bring show you the next 10 lines, *etc.*.

At the moment, we are at the start of the program. By running `next`, we will move to the next executable part of the program@

```bash
 dbg all> next
```
```
 my_prog{0}: main at /PATH/TO/gdb4hpc_exercise.c:13
```

Running `list` again will output the ten lines from 11-20. We can jump forward multiple lines by running `next N` -- by replacing *N* with a number, we will jump down *N* executable lines within our code. The `next` command will not allow us to move from one subroutine or function to another.

We can see on line 15 that there is a variable `count` about to be set. If we type:

```bash
 dbg all> print count
```

The current value of variable `count` is printed to screen. If we progress the code past line 15 and print this variable value again, it has changed to 1. If we wanted, we could have used the `watch` command to get a notification whenever the value of the variable changes.

> ## Exercise
> What happens if you keep using `next` and `list`?
> > ## Solution
> > The program will move from executable line to executable line until it reaches line 18, at which point the program is exited due to an MPI error
> {: .solution}
{: .challenge}

Let's now try launching across multiple processes:

```bash
 dbg all> launch $my_prog{2} ./my_exe
```

> ## Exercise
> The code seems to be trying to send the variable `count` from one process to another. Follow `count` (using `watch`) and see how it changes throughout the code. What happens?
> > ## Solution
> > Eventually, both processes will hang: process 0 hangs at an `MPI_Barrier` on line 19 and is stuck waiting for process 1 to reach its barrier. Process 1 is stuck at an `MPI_Recv` on line 21. Further investigation shows that it is waiting for an `MPI_Send` that does not exist -- the source is process 1 (which has not sent anything) and the tag is `1` (there is no MPI_Send with this tag). 
> {: .solution}
{: .challenge}

Let's `quit` our program, fix that bug, and go back into `gdb4hpc`. Again, we'll launch our program on 2 processes, and again, we'll watch the variable `count`. This time, both processes are able to get the same value for the variable `count`. There is one final part of the code to look at -- process 1 will try to get the sum of all even numbers between 0 and 20. However, the program begins to hang when process 1 reached line 28 (process 0 also hangs, but it's already at the `MPI_Finalise` part of the routine so we don't need to worry about it). Once we reach this hang, we can't easily keep going. Let's stop this debugging session and restart it by using `release`:

```bash
 dbg all> release $my_prog
 dbg all > launch $my_new_prog{2} ./my_exe
```

This time, instead of `next`, we will use `step` -- this does the same as `next` with the added feature that we can go into functions and subroutines where applicable. As the new bug appears to come from the `sum_even` function, let's see where exactly the program hangs.

> ## Exercise
> Having `step`ed into the `sum_even` function, can you find where the code hangs?
> > ## Solution
> > The `i++` should be brought outside of the `if` part of the `while` loop. Changing this will make the code work fully.
> {: .solution}
{: .challenge}

## Profiling tools overview

Profiling on ARCHER2 is provided through the Cray Performance Measurement and Analysis Tools (CrayPat). These have
a number of different components:

* **CrayPat** the full-featured program analysis tool set. CrayPat in turn consists of the following major components.
    * pat_build, the utility used to instrument programs
    * the CrayPat run time environment, which collects the specified performance data during program execution
    * pat_report, the first-level data analysis tool, used to produce text reports or export data for more sophisticated analysis
* **CrayPat-lite** a simplified and easy-to-use version of CrayPat that provides basic performance analysis information automatically, with a minimum of user interaction.
* **Reveal** the next-generation integrated performance analysis and code optimization tool, which enables the user to correlate performance data captured during program execution directly to the original source, and identify opportunities for further optimization.
* **Cray PAPI** components, which are support packages for those who want to access performance counters
* **Cray Apprentice2** the second-level data analysis tool, used to visualize, manipulate, explore, and compare sets of program performance data in a GUI environment.


See [the Cray Performance Measurament and Analysis Tools User Guide](https://pubs.cray.com/bundle/Cray_Performance_Measurement_and_Analysis_Tools_Installation_Guide_632_S-2474/page/Use_CrayPat_CrayPat-lite_Apprentice2_or_Reveal.html) 

## Using CrayPat Lite to profile an application
Let's grab and unpack a toy code for training purposes. To do this, we'll use `wget`.

```
auser@login01-nmn:~>  wget {{site.url}}{{site.baseurl}}/files/nbody-par.tar.gz
```
{: .language-bash}

To extract the files from a `.tar.gz` file, we run the command `tar -xvf filename.tar.gz`:
```
auser@login01-nmn:~> tar -xzf n-body-par.tar.gz
```
{: .bash}

Load CrayPat-lite module (`perftools-lite`)
```
auser@login01-nmn:~> module load perftools-lite
```
{: .bash}

and compile the application normally
```
auser@login01-nmn:~> make
```
{: .bash}
```
cc -DMPI -c main.c -o main.o
cc -DMPI -c utils.c -o utils.o
cc -DMPI -c serial.c -o serial.o
cc -DMPI -c parallel.c -o parallel.o
cc -DMPI main.o utils.o serial.o parallel.o -o nbody-parallel.exe
INFO: creating the PerfTools-instrumented executable 'nbody-parallel.exe' (lite-samples) ...OK
```
{: .output}

As the output of the compilation says, the executable `nbody-parallel.exe` has been instrumented by CrayPat-lite and therefore we can now run the application using the Slurm script provided with the source code.

Once our job has finished, we can get the performance data summarized at the end of the job STDOUT.
```
auser@login01-nmn:~> less slurm-out.txt
```
 {: .language-bash}
```
CrayPat/X:  Version 20.05.0 Revision accedbc7b  04/20/20 21:41:33

N Bodies =                     10240
Timestep dt =                  2.000e-01
Number of Timesteps =          10
Number of MPI Ranks =          64
BEGINNING N-BODY SIMULATION
SIMULATION COMPLETE
Runtime [s]:              6.759e+00
Runtime per Timestep [s]: 6.759e-01
interactions:             10
Interactions per sec:     1.551e+08

#################################################################
#                                                               #
#            CrayPat-lite Performance Statistics                #
#                                                               #
#################################################################

CrayPat/X:  Version 20.05.0 Revision accedbc7b  04/20/20 21:41:33
Experiment:                  lite  lite-samples
Number of PEs (MPI ranks):     64
Numbers of PEs per Node:       64
Numbers of Threads per PE:      1
Number of Cores per Socket:    64
Execution start time:  Wed Jul 22 14:01:02 2020
System name and speed:  nid000003  2.250 GHz (nominal)
AMD Rome       CPU  Family: 23  Model: 49  Stepping:  0
Core Performance Boost:  All 64 PEs have CPB capability
Avg Process Time:     7.84 secs
High Memory:       3,623.9 MiBytes     56.6 MiBytes per PE
I/O Write Rate:   9.319690 MiBytes/sec


...lots of output trimmed...

```	
{: .output}

## Using CrayPat to profile an application

We are now going to use the full CrayPat tools. To do so, we first need to load the required modules

```
auser@login01-nmn:~>  module unload perftools-lite
auser@login01-nmn:~>  module load perftools
```
{: .language-bash}

After loading the modules, we need to recompile the application
```
auser@login01-nmn:~> make clean; make
```
{: .bash}

Once the application has been built, we need to instrument the binary. We do this with `pat_build`
```
auser@login01-nmn:~> pat_build nbody-parallel.exe
```
{: .bash}

and a new binary called `nbody-parallel.exe+pat` it will be generated. In fact, this is the binary that we need
to run in order to obtain the performance data. Replace the ``srun`` line in the Slurm script with the following
```
srun --cpu-bind=cores ./nbody-parallel.exe+pat -n 10240 -i 10 -t 1
```
 {: .language-bash}

After the job has finished, files are stored in an experiment data directory with the following format: `exe+pat+PID-node[s|t]` where:

* `exe`: The name of the instrumented executable
* `PID`: The process ID assigned to the instrumented executable at runtime
* `node`: The physical node ID upon which the rank zero process was executed
* `[s|t]`: The type of experiment performed, either `s` for sampling or `t` for tracing
	
for example, in our case a new directory called `nbody-parallel.exe+pat+189193-3s` could be created. It is now time to obtain the performance report. We do this with the `pat_report` command and the new created directory
```
auser@login01-nmn:~> pat_report nbody-parallel.exe+pat+189193-3s
```
{: .bash}

This will command generate a full performance report and can generate a large amount of data, so you may wish to capture the data in an output file, either using a shell redirect like `>`,  or we could choose to see only some reports. If we want to see only a profile report by function we can do

```
auser@login01-nmn:~> pat_report -v -O samp_profile nbody-parallel.exe+pat+189193-3s/
```
{: .bash}

```
Table 1:  Profile by Function

  Samp% |    Samp |  Imb. |  Imb. | Group
        |         |  Samp | Samp% |  Function
        |         |       |       |   PE=HIDE

 100.0% | 1,232.9 |    -- |    -- | Total
|-----------------------------------------------------------
|  65.7% |   810.6 |    -- |    -- | MPI
||----------------------------------------------------------
||  29.5% |   364.3 |  17.7 |  4.7% | MPI_File_write_all
||  21.6% |   265.7 | 138.3 | 34.8% | MPI_Sendrecv
||  12.8% |   158.2 | 113.8 | 42.5% | MPI_File_set_view
||   1.8% |    22.2 |   5.8 | 21.0% | MPI_File_open
||==========================================================
|  27.2% |   335.0 |    -- |    -- | USER
||----------------------------------------------------------
||  26.9% |   331.5 | 138.5 | 29.9% | compute_forces_multi_set
||==========================================================
|   7.0% |    86.8 |  20.2 | 19.2% | MATH
||----------------------------------------------------------
||   7.0% |    86.8 |  20.2 | 19.2% | sqrt
|===========================================================
```
{: .output}

The table above shows the results from sampling the application. Program functions are separated out into different types, `USER` functions are those defined by the application, `MPI` functions contains the time spent in MPI library functions, `ETC` functions are generally library or miscellaneous functions included. `ETC` functions can include a variety of external functions, from mathematical functions called in by the library to system calls.

The raw number of samples for each code section is shown in the second column and the number as an absolute percentage of the total samples in the first. The third column is a measure of the imbalance between individual processors being sampled in this routine and is calculated as the difference between the average number of samples over all processors and the maximum samples an individual processor was in this routine.

Another useful table can be obtained profiling by Group, Function, and Line
```
auser@login01-nmn:~> pat_report -v -O samp_profile+src nbody-parallel.exe+pat+189193-3s/
```
{: .bash}

```
Table 3:  Profile by Group, Function, and Line

  Samp% |    Samp | Imb. |  Imb. | Group
        |         | Samp | Samp% |  Function
        |         |      |       |   Source
        |         |      |       |    Line
        |         |      |       |     PE=HIDE

 100.0% | 1,232.9 |   -- |    -- | Total
|-------------------------------------------------------------------
|  65.7% |   810.6 |   -- |    -- | MPI
||------------------------------------------------------------------
||  29.5% |   364.3 |   -- |    -- | MPI_File_write_all
||  21.6% |   265.7 |   -- |    -- | MPI_Sendrecv
|||-----------------------------------------------------------------
|||=================================================================
||  12.8% |   158.2 |   -- |    -- | MPI_File_set_view
||   1.8% |    22.2 |  5.8 | 21.0% | MPI_File_open
||==================================================================
|  27.2% |   335.0 |   -- |    -- | USER
||------------------------------------------------------------------
||  26.9% |   331.5 |   -- |    -- | compute_forces_multi_set
3|        |         |      |       |  z19/lcebaman/nbody-par/parallel.c
||||----------------------------------------------------------------
4|||   2.7% |    32.7 | 32.3 | 50.5% | line.142
4|||   5.7% |    70.8 | 16.2 | 18.9% | line.144
4|||   4.5% |    55.7 | 21.3 | 28.1% | line.145
4|||   6.3% |    77.6 | 21.4 | 22.0% | line.148
4|||   3.9% |    47.9 | 17.1 | 26.8% | line.149
||||================================================================
||==================================================================
|   7.0% |    86.8 | 20.2 | 19.2% | MATH
||------------------------------------------------------------------
||   7.0% |    86.8 | 20.2 | 19.2% | sqrt
|===================================================================
```
{: .output}

If we want to profile by Function and Callers, with Line Numbers then
```
auser@login01-nmn:~> pat_report -O ca+src nbody-parallel.exe+pat+189193-3s/
```
{: .bash}

```
Table 1:  Profile by Function and Callers, with Line Numbers

  Samp% |    Samp | Group
        |         |  Function
        |         |   Caller
        |         |    PE=HIDE

 100.0% | 1,232.9 | Total
|-----------------------------------------------------------------
|  65.7% |   810.6 | MPI
||----------------------------------------------------------------
||  29.5% |   364.3 | MPI_File_write_all
3|        |         |  distributed_write_timestep:parallel.c:line.218
4|        |         |   run_parallel_problem:parallel.c:line.73
5|        |         |    __real_main:main.c:line.81
6|        |         |     main
||  21.6% |   265.7 | MPI_Sendrecv
3|  21.5% |   265.7 |  run_parallel_problem:parallel.c:line.75
4|        |         |   __real_main:main.c:line.81
5|        |         |    main
||  12.8% |   158.2 | MPI_File_set_view
3|        |         |  distributed_write_timestep:parallel.c:line.216
4|        |         |   run_parallel_problem:parallel.c:line.73
5|        |         |    __real_main:main.c:line.81
6|        |         |     main
||   1.8% |    22.2 | MPI_File_open
3|        |         |  run_parallel_problem:parallel.c:line.59
4|        |         |   __real_main:main.c:line.81
5|        |         |    main
||================================================================
|  27.2% |   335.0 | USER
||----------------------------------------------------------------
||  26.9% |   331.5 | compute_forces_multi_set
3|  26.9% |   331.3 |  run_parallel_problem:parallel.c:line.81
4|        |         |   __real_main:main.c:line.81
5|        |         |    main
||================================================================
|   7.0% |    86.8 | MATH
||----------------------------------------------------------------
||   7.0% |    86.8 | sqrt
3|        |         |  run_parallel_problem:parallel.c:line.81
4|        |         |   __real_main:main.c:line.81
5|        |         |    main
|=================================================================
```
{: .output}

The reports will generate two more files, one with the extension `.ap2` which holds the same data as the report data (`.xf`) but in the post processed form. The other file is called `build-options.apa` and is a text file with a suggested configuration for generating a traced experiment. You are welcome and encouraged to review this file and modify its contents in subsequent iterations, however in this first case we will continue with the defaults.

This `build-options.apa` file acts as the input to the `pat_build` command and is supplied as the argument to the `-O` flag.

```
pat_build -O nbody-parallel.exe+pat+189193-3s/build-options.apa
```
{: .bash}

This will produce a third binary with extension `+apa`. This binary should once again be run on the back end, so the submission script should be modified and the name of the executable changed to `nbody-parallel.exe+apa`.
Similarly to the sampling process, a new directory called `exe+apa+PID-node[s|t]` will be generated by the application, which should be processed by the `pat_report` tool. The output format of this new directory is similar to the one obtained with sampling, but now this includes `apa` and `t` to indicate that this is a tracing experiment. For instance, with our code we could get a new directory called `nbody-parallel.exe+apa+69935-4t`.
```
auser@login01-nmn:~> pat_report nbody-parallel.exe+apa+69935-4t
```
{: .bash}

```
Table 1:  Profile by Function Group and Function

  Time% |     Time |     Imb. |  Imb. | Calls | Group
        |          |     Time | Time% |       |  Function
        |          |          |       |       |   PE=HIDE

 100.0% | 6.653825 |       -- |    -- | 679.0 | Total
|-----------------------------------------------------------------
|  64.3% | 4.277528 | 1.785103 | 29.9% |   1.0 | USER
||----------------------------------------------------------------
||  64.3% | 4.277528 | 1.785103 | 29.9% |   1.0 | __real_main
||================================================================
|  31.4% | 2.090579 |       -- |    -- | 675.0 | MPI
||----------------------------------------------------------------
||  21.4% | 1.425524 | 0.485545 | 25.8% | 640.0 | MPI_Sendrecv
Processing step 11 of 11
||   6.2% | 0.410421 | 0.404390 | 50.4% |  10.0 | MPI_File_set_view
||   1.5% | 0.100346 | 0.001022 |  1.0% |  10.0 | MPI_File_write_all
||   1.3% | 0.085070 | 0.000966 |  1.1% |   1.0 | MPI_File_open
||================================================================
|   4.3% | 0.285718 |       -- |    -- |   3.0 | MPI_SYNC
||----------------------------------------------------------------
||   4.0% | 0.268344 | 0.266963 | 99.5% |   1.0 | MPI_Init(sync)
|=================================================================
```
{: .output}

The new table above is the version generated from tracing data instead of the previous sampling data table. This version makes true timing information is available (averages per processor) and the number of times each function is called. Timings are more accurate and features like the number of calls are available.

We can also get very important performance data from the hardware (HW) counters
```
auser@login01-nmn:~> pat_report -v -O profile+hwpc nbody-parallel.exe+apa+69935-4t
```
{: .bash}
```
Table 1:  Profile by Function Group and Function

Group / Function / PE=HIDE


==============================================================================
  Total
------------------------------------------------------------------------------
  Time%                                                  100.0%
  Time                                                 6.653825 secs
  Imb. Time                                                  -- secs
  Imb. Time%                                                 --
  Calls                         102.044 /sec              679.0 calls
  PAPI_TOT_INS                    0.272G/sec  1,807,999,565.422 instr
  PAPI_TOT_CYC                                2,617,561,820.844 cycles
  L2_LATENCY                                          8,358,045 cycles
  REQUESTS_TO_L2_GROUP1:L2_HW_PF  0.035M/sec            230,983 ops
  REQUESTS_TO_L2_GROUP1:
    RD_BLK_L:RD_BLK_X:
    LS_RD_BLK_C_S:
    CACHEABLE_IC_READ:
    CHANGE_TO_X:PREFETCH_L2:
    L2_HW_PF                      0.370M/sec          2,463,366 ops
  L2 Avg Latency                                          13.57 cycles
  HW L2 Prefetch / L2 Requests                             9.4%
  Instr per cycle                                          0.69 inst/cycle
  MIPS                        17,390.30M/sec
  Average Time per Call                                0.009800 secs
  CrayPat Overhead : Time          0.2%
==============================================================================
```
{: .output}

## Getting help with debugging and profiling tools

You can find more information on the debugging and profiling tools available on ARCHER2 in the ARCHER2 Documentation
and the Cray documentation:

* [ARCHER2 Documentation](https://docs.archer2.ac.uk)
* [Cray Technical Documentation](https://pubs.cray.com/)

If the documentation does not answer your questions then please contact
[the ARCHER2 Service Desk](https://www.archer2.ac.uk/support-access/servicedesk.html) and they
will be able to assist you.

{% include links.md %}

