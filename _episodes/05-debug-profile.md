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
- "The main profiling tool on ARCHER2 is *CrayPAT*"
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

* **CrayPAT** the full-featured program analysis tool set. CrayPat in turn consists of the following major components.
    * pat_build, the utility used to instrument programs
    * the CrayPat run time environment, which collects the specified performance data during program execution
    * pat_report, the first-level data analysis tool, used to produce text reports or export data for more sophisticated analysis
* **CrayPAT-lite** a simplified and easy-to-use version of CrayPat that provides basic performance analysis information automatically, with a minimum of user interaction.
* **Reveal** the next-generation integrated performance analysis and code optimization tool, which enables the user to correlate performance data captured during program execution directly to the original source, and identify opportunities for further optimization.
* **Cray PAPI** components, which are support packages for those who want to access performance counters
* **Cray Apprentice2** the second-level data analysis tool, used to visualize, manipulate, explore, and compare sets of program performance data in a GUI environment.

**TODO** Add more details once we have seen the TDS. Links to further documentation.

**TODO** Add exercise on using one or more of the tools (likely CrayPAT-lite and CrayPAT)

## Getting help with debugging and profiling tools

You can find more information on the debugging and profiling tools available on ARCHER2 in the ARCHER2 Documentation
and the Cray documentation:

* [ARCHER2 Documentation](https://docs.archer2.ac.uk)
* [Cray Technical Documentation](https://pubs.cray.com/)

If the documentation does not answer your questions then please contact
[the ARCHER2 Service Desk](https://www.archer2.ac.uk/support-access/servicedesk.html) and they
will be able to assist you.

{% include links.md %}

