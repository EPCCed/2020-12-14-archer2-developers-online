---
title: "Debugging and profiling on ARCHER2"
teaching: 15
exercises: 0
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

* **gdb4hpc** is a command-line debugging tool provided by Cray. It works similarly to
  [gdb](https://www.gnu.org/software/gdb/), but allows the user to debug multiple parallel processes
  without multiple windows. gdb4hpc can be used to investigate deadlocked code, segfaults, and other
  errors for C/C++ and Fortran code. Users can single-step code and focus on specific processes groups
  to help identify unexpected code behavior. (text from [ALCF](https://www.alcf.anl.gov/support-center/theta/gdb)).
* **valgrind4hpc** is a parallel memory debugging tool that aids in detection of memory leaks and
  errors in parallel applications. It aggregates like errors across processes and threads to simply
  debugging of parallel appliciations.
* **STAT** generate merged stack traces for parallel applications. Also has visualisation tools.
* **ATP** scalable core file and backtrace analysis when parallel programs crash.
* **CCDB** Cray Comparative Debugger. Compare two versions of code side-by-side to analyse differences.

See [the Cray Performance Measurement and Analysis Tools User Guide](https://pubs.cray.com/content/S-2376/7.0.0/cray-performance-measurement-and-analysis-tools-user-guide/about-the-cray-performance-measurement-and-analysis-tools-user-guide)

**TODO** Add more details once we have seen the TDS. Links to further documentation.

**TODO** Add exercise on using one or more of the tools (likely gdb4hpc and valgrind4hpc)

## Profiling tools overview

Profiling on ARCHER2 is provide through the Cray Performance Measurement and Analysis Tools (CrayPat). This has
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

As the output of the compilation says, the executable `nbody-parallel.exe` has been instrumented by CrayPat-lite and therefore we can now run the application using the following script

```
#!/bin/bash
#SBATCH --job-name=my_mpi_job
#SBATCH --account=t01
#SBATCH --time=0:05:0
#SBATCH --exclusive
#SBATCH --nodes=1
#SBATCH --tasks-per-node=64
#SBATCH --cpus-per-task=1
#SBATCH --output=slurm-out.txt

export OMP_NUM_THREADS=1

srun --cpu-bind=cores ./nbody-parallel.exe -n 10240 -i 10 -t 1
```
 {: .language-bash}

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
to run in order to obtain the performance data

```
#!/bin/bash
#SBATCH --job-name=my_mpi_job
#SBATCH --account=t01
#SBATCH --time=0:05:0
#SBATCH --exclusive
#SBATCH --nodes=1
#SBATCH --tasks-per-node=64
#SBATCH --cpus-per-task=1
export OMP_NUM_THREADS=1

srun --cpu-bind=cores ./nbody-parallel.exe+pat -n 10240 -i 10 -t 1
```
 {: .language-bash}

After the job has finished, a new directory called `nbody-parallel.exe+pat+...` will be created. It is now time to obtain the performance report. We do this with the `pat_report` command and the new created directory
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
A new directory `nbody-parallel.exe+apa+...`  will be generated by the application, which should be processed by the `pat_report` tool. As this is now a tracing experiment it will provide more information than before
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

