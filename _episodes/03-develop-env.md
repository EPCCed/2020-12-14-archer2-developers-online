---
title: "ARCHER2 software environment "
teaching: 10
exercises: 0
questions:
- "What does the ARCHER2 software environment look like and how do I access software?"
- "How can I find out what software is available?"
- "How can I request, install or get help with software on ARCHER2?"
objectives:
- "Know how to access different software on ARCHER2 using Lmod modules."
- "Know how to find out what is installed and where to get help."
keypoints:
- "Software is available through modules."
- "The CSE service can help with software issues."
---

## Using software modules on ARCHER2

ARCHER2 software modules use the [Lmod](https://lmod.readthedocs.io/en/latest/) system to provide
access to different software and versions on the system. The modules and versions available will
change across the lifetime of the service.

Software modules are provided by both Cray and the ARCHER2 CSE team at [EPCC](https://www.epcc.ed.ac.uk).

## What modules are loaded when you log into ARCHER2?

All users start with a default set of modules loaded into their environment

## Finding out what software is available

You can query which software is provided by modules with the `module avail` command:

```
module avail
```
{: .bash}
```
[auser@archer2-login1 ~]$ module avail

------------------------------------------------ /home/software/modulefiles ------------------------------------------------
   anaconda2/2019-GNU7        mercury/latest                   parmetis/4.0.3
   anaconda2/2019      (D)    mercury/1.0.1             (D)    phdf5/1.10.5/gnu
   anaconda3/2019             mesa/18.0.0                      phdf5/1.10.5/gnu7          (D)
   arm-forge/h2020            metis/5.1.0                      phdf5/1.10.5/gnu8
   boost/1.69.0-GNU           mpi4py/python2-openmpi           pmdk/1.6
   cmake/3.14.4               mpi4py/python2                   python2-dill/0.2.9
   embree/3.5.2               mpi4py/python3-openmpi           python2/2.7.13-debug-forge
   fftw/3.3.8/gnu             mpi4py/python3            (D)    python2/2.7.13-debug
   fftw/3.3.8/gnu7            ncurses/6.1                      python2/2.7.13             (D)
   fftw/3.3.8/gnu8            ndctl/v67                        python3-dill/0.2.9
   fftw/3.3.8/intel    (D)    netcdf-fortran/4.4.5/gnu         python3-future/0.17.0
   gdb/8.3                    netcdf-fortran/4.4.5/gnu7        python3-six/1.12.0
   glfw/3.3                   netcdf-fortran/4.4.5/gnu8 (D)    python3-sphinx/2.0.1
   go/1.12.7                  netcdf/4.7.0/gnu                 python3/3.6.8-debug-forge
   gperftools/2.7-GNU         netcdf/4.7.0/gnu7                python3/3.6.10             (D)
   htop/2.2.0                 netcdf/4.7.0/gnu8         (D)    rapidjson/1.1.0
   ispc/1.11.0                openblas/0.3.6-GNU        (D)    scipy/1.2.3
   java/13-JDK                openblas/0.3.6-GNU7              scons/3.0.5
   lapack/3.9.0-GNU7          openmpi/1.10.7-GNU               scotch/6.0.7
   libfabric/latest           openmpi/1.10.7-GNU7              sionlib/1.7.4
   libfabric/1.6.2            openmpi/1.10.7-Intel             snappy/1.1.7
   libfabric/1.7.1     (D)    openmpi/2.0.4-GNU7               vampir/9.7.1
   libfabric/1.8.0            openmpi/4.0.3-GNU7               vtk/5.10.1-GNU7
   libunwind/1.3.1            otf2/2.1.1                       vtk/5.10.1
   likwid/5.0.1               papi/5.7.1.0                     vtk/8.2.0                  (D)
   memkind/1.9.0              paraview/5.7.0                   zoltan/3.8.3

  Where:
   D:  Default Module
   L:  Module is loaded

Module defaults are chosen based on Find First Rules due to Name/Version/Version modules found in the module tree.
See https://lmod.readthedocs.io/en/latest/060_locating.html for details.

Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".


```
{: .output}

The output lists the available modules and their versions. It also shows you which modules you currently
have loaded: those marked with `(L)`  and which modules are loaded by default (marked with `(D)`) when there
are multiple versions available and you do not specify the version when you load.

> ## Licensed software
> Some of the software installed on ARCHER2 requires the user to have their licence validated before they
> can use it on the service. More information on gaining access to licensed software through the SAFE
> is provided below.
{: .callout}

If you want more information on a particular module, you can use the `module spider` command. For example,
to get more info on the `netcdf` module:

```
[auser@archer2-login1 ~]$ module spider netcdf
```
{: .bash}
```
------------------------------------------------------------------------------------------------------------------------
  netcdf:
------------------------------------------------------------------------------------------------------------------------
    Description:
      C Libraries for the Unidata network Common Data Form 

     Versions:
        netcdf/4.5.0
        netcdf/4.6.2
     Other possible modules matches:
        netcdf-cxx  netcdf-fortran  netcdf-fortran/4.4.5  netcdf/4.7.0

------------------------------------------------------------------------------------------------------------------------
  To find other possible module matches execute:

      $ module -r spider '.*netcdf.*'

------------------------------------------------------------------------------------------------------------------------
  For detailed information about a specific "netcdf" module (including how to load the modules) use the module's full name.
  For example:

     $ module spider netcdf/4.6.2
------------------------------------------------------------------------------------------------------------------------
```
{: .output}

If you ask for more detailed information on a module, it will also tell you any modules that need to be
loaded as prerequisites to loading your chosen modules (i.e. the `netcdf/4.6.2` module's *dependencies*).
The Lmod tool will load these dependencies for you when you load the module (see below for more on this).

```
[auser@archer2-login1 ~]$ module spider netcdf/4.6.2
```
{: .bash}
```
------------------------------------------------------------------------------------------------------------------------
  netcdf: netcdf/4.6.2
------------------------------------------------------------------------------------------------------------------------
    Description:
      C Libraries for the Unidata network Common Data Form 


    You will need to load all module(s) on any one of the lines below before the "netcdf/4.6.2" module is available to load.

      intel/19.0.3.199  impi/2019.3.199
      intel/19.0.3.199  mpich/3.3
      intel/19.0.3.199  mvapich2/2.3
 
```
{: .output}

## Loading and switching modules

Lets look at our environment before we change anything. To see just our loaded modules we use the
`module list` (or `ml`) command:

```
[auser@archer2-login1 ~]$ ml
```
{: .bash}
```
Currently Loaded Modules:
  1) autotools   2) prun/1.3   3) intel/19.0.3.199   4) impi/2019.3.199   5) ohpc   6) packages-nextgenio
```
{: .output}

You load modules with the `module load` (or `ml`) command. For example, to load the `netcdf` module:

```
[auser@archer2-login1 ~]$ ml netcdf
```
{: .bash}

> ## `ml` has two meanings
> Notice the the convenience command `ml` has a different meaning depending on whether or not you 
> supply an argument. Without an argument it means `module list` and with an argument, it means
> `module load`.
{: .callout

}

Now, lets list our loaded modules again with `ml`:

```
[auser@archer2-login1 ~]$ ml
```
{: .bash}
```
Currently Loaded Modules:
  1) autotools   3) intel/19.0.3.199   5) ohpc                 7) phdf5/1.10.4
  2) prun/1.3    4) impi/2019.3.199    6) packages-nextgenio   8) netcdf/4.6.2
```
{: .output}

You can see that as well as the default `netcdf` module (`netcdf/4.6.2` as we did not specify a version
explicitly) Lmod has also loaded the dependencies for the default `netcdf` module.

If you want to swap two versions of the same module then you simply load the version that you 
want to swap for the currently loaded version, Lmod recognises that they are the same module 
with different versions and swaps them for you. 

## Licensed software

Some of the software installed on ARCHER2 requires a user to have a valid licence agreed with the 
software owners/developers to be able to use it (for example, VASP). Although you will be able to
load this software on ARCHER2 you will be barred from actually using it until your licence has been
verified.

You request access to licensed software through the EPCC SAFE (the web administration tool you used
to apply for your account and retrieve your initial password) by being added to the appropriate
*Package Group*. To request access to licensed software:

1. Log in to [SAFE](https://www.archer.ac.uk/safe/) (**TODO** Update SAFE link)
2. Go to the Menu *Login accounts* and select the login account which requires access to the software
3. Click *New Package Group Request*
4. Select the software from the list of available packages and click *Select Package Group*
5. Fill in as much information as possible about your license; at the very least provide the information
   requested at the top of the screen such as the licence holder's name and contact details. If you are
   covered by the license because the licence holder is your supervisor, for example, please state this.
6. Click *Submit*
7. 
Your request will then be processed by the ARCHER2 Service Desk who will confirm your license with the
software owners/developers before enabling your access to the software on ARCHER2. This can take several
days (depending on how quickly the software owners/developers take to respond) but you will be advised
once this has been done.

## Getting help with software

You can find more information on the software available on ARCHER2 in the ARCHER2 Documentation at:

* https://docs.archer2.ac.uk

This includes information on the software provided by Cray and the software provided by the 
ARCHER2 CSE Service at EPCC.

If the software you require is not currently available or you are having trouble with the installed
software then please contact
[the ARCHER2 Service Desk](https://www.archer2.ac.uk/support-access/servicedesk.html) and they
will be able to assist you.

{% include links.md %}

