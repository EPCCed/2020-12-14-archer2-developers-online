---
title: "ARCHER2 development environment "
teaching: 30
exercises: 40
questions:
- "What does the ARCHER2 development environment look like and how do I access different components?"
- "How can I find out what compilers, tools, and libraries are available?"
- "How can I capture my current environment for reuse or to share with others?"
- "How can I get help with compiling and developing software on ARCHER2?"
objectives:
- "Know how to access different parts of the development environment on ARCHER2 using Lmod modules."
- "Know how to find out what is installed and where to get help."
keypoints:
- "The development environment is controlled through Lmod modules."
- "ARCHER2 supports the GCC and Cray compilers."
- "Compilers are accessed through the `ftn`, `cc` and `CC` wrapper commands."
- "The CSE service can help with software development issues."
---

## Using software modules on ARCHER2

ARCHER2 software modules use the
[environment modules](https://modules.readthedocs.io/en/latest/index.html) system to provide
access to different software and versions on the system. The modules and versions available will
change across the lifetime of the service.

Software modules are provided by both HPE Cray and the ARCHER2 CSE team at
[EPCC](https://www.epcc.ed.ac.uk).

## What modules are loaded when you log into ARCHER2?

All users start with a default set of modules loaded into their environment. These include:

   - Cray Compiler Environment (CCE)
   - Cray MPICH MPI library
   - Cray LibSci scientific and numerical libraries
   - Cray lightweight performance analysis toolkit
   - System modules to enable use of the ARCHER2 hardware

You can see what modules you currently have loaded with the `module list` command:

```
auser@login01-nmn:~> module list
```
{: .language-bash}
```
Currently Loaded Modulefiles:
 1) cpe-cray                          7) cray-dsmml/0.1.2(default)                           
 2) cce/10.0.3(default)               8) perftools-base/20.09.0(default)                     
 3) craype/2.7.0(default)             9) xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta(default)  
 4) craype-x86-rome                  10) cray-mpich/8.0.15(default)                          
 5) libfabric/1.11.0.0.233(default)  11) cray-libsci/20.08.1.2(default)                      
 6) craype-network-ofi 
```
{: .output}

This means that your environment is already configured to build software using the Cray Compiling
Environment (CCE), the current default version of HPE Cray MPICH, and with numerical libraries provided
by the current default version of HPE Cray LibSci.

> ## Getting back if you purge or make a mistake
> 
> Unlike many other HPC systems you may have used, you should not generally use the
> `module purge` command before starting to use the system. Some of the modules
> loaded by default are required for you to be able to use the system correctly and
> so many things will not work if you use `module purge`. If you need to change the
> setup, you will generally use the `module load` or `module swap` commands instead.
>
> If you do find yourself with a broken environment you can usually fix things by
> logging out and logging back in again.
{: .callout}

## Finding out what software is available

You can query which software is provided by modules with the `module avail` command:

```
auser@login01-nmn:~> module avail
```
{: .language-bash}
```
----------------------- /opt/cray/pe/perftools/20.05.0/modulefiles ------------------------
perftools       perftools-lite-events  perftools-lite-hbm    perftools-nwpc     
perftools-lite  perftools-lite-gpu     perftools-lite-loops  perftools-preload  

-------------------------- /opt/cray/pe/craype/2.6.4/modulefiles --------------------------
craype-hugepages1G  craype-hugepages8M   craype-hugepages128M  craype-network-slingshot10  
craype-hugepages2G  craype-hugepages16M  craype-hugepages256M  craype-x86-rome             
craype-hugepages2M  craype-hugepages32M  craype-hugepages512M  
craype-hugepages4M  craype-hugepages64M  craype-network-none   

----------------------------- /usr/local/Modules/modulefiles ------------------------------
dot  module-git  module-info  modules  null  use.own  

-------------------------------- /opt/cray/pe/modulefiles ---------------------------------
atp/3.5.4(default)                         cray-openshmemx/10.1.0(default)         
cce/10.0.0(default)                        cray-parallel-netcdf/1.11.1.1(default)  
cray-ccdb/4.5.4(default)                   cray-pmi-lib/6.0.5(default)             
cray-cti/2.5.6(default)                    cray-pmi/6.0.5(default)                 
cray-dsmml/0.1.0(default)                  cray-stat/4.4.5(default)                
cray-fftw/3.3.8.4(default)                 craype-dl-plugin-py3/20.05.1            
cray-fftw/3.3.8.5                          craype/2.6.4(default)                   
cray-ga/5.7.0.3                            craypkg-gen/1.3.9(default)              
cray-hdf5-parallel/1.10.5.2(default)       gdb4hpc/4.5.6(default)                  
cray-hdf5/1.10.5.2(default)                iobuf/2.0.9(default)                    
cray-libsci/20.03.1.4(default)             papi/5.7.0.3(default)                   
cray-mpich-abi/8.0.10                      perftools-base/20.05.0(default)         
cray-mpich/8.0.10(default)                 PrgEnv-cray/7.0.0(default)              
cray-netcdf-hdf5parallel/4.6.3.2(default)  PrgEnv-gnu/7.0.0(default)               
cray-netcdf/4.6.3.2(default)               valgrind4hpc/2.5.5(default)             

---------------------------------- /opt/cray/modulefiles ----------------------------------
capsules/0.8.3(default)                                                                 
chapel/1.20.1(default)                                                                  
cray-lustre-client/2.12.0.5_cray_166_gf6711cf-7.0.1.0_2.24__gf6711cfb3.shasta(default)  
cray-shasta-mlnx-firmware/1.0.5(default)                                                
dvs/2.12_2.2.259-7.0.1.0_6.1__g6ee74127(default)                                        
libfabric/1.10.0.0.249(default)                                                         
spark/3.0.0(default)                                                                    
xpmem/2.2.35-7.0.1.0_3.9__gfa8d091.shasta(default)                                      

------------------------------------ /opt/modulefiles -------------------------------------
cray-python/3.8.2.0(default)  cray-R/3.6.3(default)  gcc/8.1.0  gcc/9.3.0(default)

```
{: .output}

The output lists the available modules and their versions. It also shows you which modules are
loaded by default (marked with `(default)`) when there are multiple versions available and you do
not specify the version when you load.

> ## Licensed software
> Some of the software installed on ARCHER2 requires the user to have their licence validated before they
> can use it on the service. More information on gaining access to licensed software through the SAFE
> is provided below.
{: .callout}

If you want more information on a particular module, you can use the `module help` command. For example,
to get more info on the `cray-netcdf` module:

```
aturner@login01-nmn:~> module help cray-netcdf
```
{: .language-bash}
```
-------------------------------------------------------------------
Module Specific Help for /opt/cray/pe/modulefiles/cray-netcdf/4.7.4.0:


cray-netcdf
===========

Release Date:
-------------
  August 2020

Purpose:
--------
  * Update to upstream release 4.7.4.

Product and OS Dependencies:
----------------------------
  The NetCDF release is supported on the following systems:
    * Cray XC systems with CLE 7.0 UP02 or later

  The NetCDF 4.7.4.0 release requires the following software products:

    Cray HDF5 1.12.0.*
    CrayPE 2.1.2 or later

    One or more compilers:
        CCE 9.0 or later
        GCC 8.0 or later
        Intel 19.0 or later
        PGI 20.1 or later
        Allinea 20.0 or later
        AOCC 2.1 or later

Notes and Limitations:
---------------------
    Unidata now packages Netcdf-4 and legacy netcdf-3 separately. Cray has
    decided not to continue supplying the legacy Netcdf-3 package. Due to CCE
    changes a version of netcdf built with "-sreal64" is neither needed nor
    provided.

    NetCDF is supported on the host CPU but not on the accelerator on
    Cray XC systems.

Documentation:
--------------
    http://www.unidata.ucar.edu/software/netcdf/docs

Modulefile:
-----------
    module load cray-netcdf
    OR
    module load cray-netcdf-hdf5parallel

Product description:
--------------------
  NetCDF (network Common Data Form) is a set of interfaces for array-oriented
  data access and a freely-distributed collection of data access libraries for
  C, Fortran, C++, Java, and other languages. The netCDF libraries support a
  machine-independent format for representing scientific data. Together, the
  interfaces, libraries, and format support the creation, access, and sharing
  of scientific data.
```
{: .output}


<!-- Commented out until Lmod is really available and we can test

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
{: .output} -->



## Loading and switching modules

Lets look at our environment before we change anything. To see just our loaded modules we use the
`module list` command:

```
auser@login01-nmn:~> module list
```
{: .language-bash}
```
Currently Loaded Modulefiles:
 1) cpe-cray                          7) cray-dsmml/0.1.2(default)                           
 2) cce/10.0.3(default)               8) perftools-base/20.09.0(default)                     
 3) craype/2.7.0(default)             9) xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta(default)  
 4) craype-x86-rome                  10) cray-mpich/8.0.15(default)                          
 5) libfabric/1.11.0.0.233(default)  11) cray-libsci/20.08.1.2(default)                      
 6) craype-network-ofi 
```
{: .output}

You load modules with the `module load` command. For example, to load the `cray-netcdf` module:

```
auser@login01-nmn:~> module load cray-netcdf
```
<!-- Commented out until Lmod is really available and we can test{: .language-bash}

> ## `ml` has two meanings
> Notice the the convenience command `ml` has a different meaning depending on whether or not you 
> supply an argument. Without an argument it means `module list` and with an argument, it means
> `module load`.
{: .callout} -->

Now, lets list our loaded modules again with `module list`:

```
auser@login01-nmn:~> module list
```
{: .language-bash}
```
Currently Loaded Modulefiles:
 1) cpe-cray                          7) cray-dsmml/0.1.2(default)                           
 2) cce/10.0.3(default)               8) perftools-base/20.09.0(default)                     
 3) craype/2.7.0(default)             9) xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta(default)  
 4) craype-x86-rome                  10) cray-mpich/8.0.15(default)                          
 5) libfabric/1.11.0.0.233(default)  11) cray-libsci/20.08.1.2(default)                      
 6) craype-network-ofi               12) cray-netcdf/4.7.4.0 
```
{: .output}

You can see that we now have the default `cray-netcdf` module (`cray-netcdf/4.7.4.0` as we did not specify a version
explicitly).

<!--
  Note: Once Lmod is working, we likely want to choose a module that has dependencies so we 
  can note that Lmod has automatically loaded dependencies. -->

If you want to swap two versions of the same module then you use the `module swap` command.

<!-- Need to add example once there is a module we can actually swap! -->

## Switching between compiler environments

You use the `module restore` command to switch between different compiler environments available
on ARCHER2. The available environments are:

* Cray Compiling Environment (CCE): `PrgEnv-cray`
* GNU Compiler Collection (GCC): `PrgEnv-gnu`
* AMD Optimizing Compilers (AOCC): `PrgEnv-aocc`

These programming environments are all defined by *module collections*. `PrgEnv-cray` is loaded by
default when you log in. You can change to the GCC compiler environment with the
`module restore PrgEnv-gnu` command (with `module list` afterwards to show how things have changed):

```
auser@login01-nmn:~> module restore PrgEnv-gnu
auser@login01-nmn:~> module list
```
{: .language-bash}
```
Unloading cray-libsci/20.08.1.2
Unloading cray-mpich/8.0.15
Unloading xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta

Unloading perftools-base/20.09.0
  WARNING: Did not unuse /opt/cray/pe/perftools/20.09.0/modulefiles

Unloading cray-dsmml/0.1.2
Unloading craype-network-ofi
Unloading libfabric/1.11.0.0.233
Unloading craype-x86-rome

Unloading craype/2.7.0
  WARNING: Did not unuse /opt/cray/pe/craype/2.7.0/modulefiles

Unloading cce/10.0.3
Unloading cpe-cray
Loading cpe-gnu
Loading gcc/10.1.0
Loading craype/2.7.0
Loading craype-x86-rome
Loading libfabric/1.11.0.0.233
Loading craype-network-ofi
Loading cray-dsmml/0.1.2
Loading perftools-base/20.09.0
Loading xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta
Loading cray-mpich/8.0.15
Loading cray-libsci/20.08.1.2

Currently Loaded Modulefiles:
 1) cpe-gnu                           7) perftools-base/20.09.0(default)                     
 2) craype/2.7.0(default)             8) xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta(default)  
 3) craype-x86-rome                   9) cray-mpich/8.0.15(default)                          
 4) libfabric/1.11.0.0.233(default)  10) cray-libsci/20.08.1.2(default)                      
 5) craype-network-ofi               11) gcc/10.1.0                                           
 6) cray-dsmml/0.1.2(default)   
```

## Switching between different compiler versions

Unlike the programming environments, you use the `module swap` command to switch between different
versions of the compilers within the programming environments. To do this, you need to know the
names of the compiler modules, they are:

* `cce` for the Cray Compiling Environment
* `gcc` for the GNU Compiler Collection
* `aocc` for the AMD Optimising Compilers

For example, to change the version of GCC you are using you would first switch to the Gnu programming
environment and then switch to a different version of GCC:

```
auser@login01-nmn:~> module restore PrgEnv-gnu
auser@login01-nmn:~> module swap gcc gcc/9.3.0
auser@login01-nmn:~> module list
```
{: .language-bash}
```
Unloading cray-libsci/20.08.1.2
Unloading cray-mpich/8.0.15
Unloading xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta

Unloading perftools-base/20.09.0
  WARNING: Did not unuse /opt/cray/pe/perftools/20.09.0/modulefiles

Unloading cray-dsmml/0.1.2
Unloading craype-network-ofi
Unloading libfabric/1.11.0.0.233
Unloading craype-x86-rome

Unloading craype/2.7.0
  WARNING: Did not unuse /opt/cray/pe/craype/2.7.0/modulefiles

Unloading cce/10.0.3
Unloading cpe-cray
Loading cpe-gnu
Loading gcc/10.1.0
Loading craype/2.7.0
Loading craype-x86-rome
Loading libfabric/1.11.0.0.233
Loading craype-network-ofi
Loading cray-dsmml/0.1.2
Loading perftools-base/20.09.0
Loading xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta
Loading cray-mpich/8.0.15
Loading cray-libsci/20.08.1.2

Currently Loaded Modulefiles:
 1) cpe-gnu                           7) perftools-base/20.09.0(default)                     
 2) craype/2.7.0(default)             8) xpmem/2.2.35-7.0.1.0_1.3__gd50fabf.shasta(default)  
 3) craype-x86-rome                   9) cray-mpich/8.0.15(default)                          
 4) libfabric/1.11.0.0.233(default)  10) cray-libsci/20.08.1.2(default)                      
 5) craype-network-ofi               11) gcc/9.3.0                                           
 6) cray-dsmml/0.1.2(default)         
```
{: .output}

> ## Not all versions are guaranteed to work
> Each Cray Programming Environment release is released as a coherent set of software that is 
> guaranteed to work together as a whole. This means that not all compiler version and library
> version combinations have been tested. If you select a combination from two different PE
> releases you may experience incompatibility issues.
{: .callout}

<!-- Add note on cdt modules once they are there -->

## Compiler wrapper scripts

Code compiled on ARCHER2 should use the compiler wrapper scripts to ensure that
all the correct libraries and header files are included in both the compile and
link stages.

> ## Linked libraries
> Loading a module will automatically set the necessary paths and link flags for
> that software, eliminating the need to specify them manually using the
> `LDFLAGS` variable or another mechanism. For example, the `cray-libsci` module
> is loaded by default on login and sets the the wrappers to link in BLAS,
> LAPACK, and ScaLAPACK functionality without the need for the user to provide
> an explicit `-llib_sci`.
{: .callout}

The compiler wrapper scripts are available for Fortran, C, and C++:
* `ftn` -- Fortran compiler
* `cc` -- C compiler
* `CC` -- C++ compiler

The wrapper scripts can be used to compile both sequential and parallel codes.

## Dynamic linking on ARCHER2

ARCHER2 currently only supports dynamic linking (you will see an error if you 
try to link applications statically). As well as linking dynamically, all 
HPE Cray software libraries are added to the executable *RUNPATH*. This means
that you do not need to restore programming environments or load particular
Cray modules in your job submission scripts to run on the compute nodes. However,
if you do load different versions of libraries used by your application in your
job submission scripts, these will take precedence over the versions encoded in
the binary RUNPATH.

**Note:** Software libraries provided by the CSE team via modules **are not**
captured in the binary RUNPATH so these **do** need to be loaded in your 
job submission scritps for the dynamic executable to be able to find them
successfully.

## Getting help with software

You can find more information on the software available on ARCHER2 in the ARCHER2 Documentation at:

* [ARCHER2 Documentation](https://docs.archer2.ac.uk)

This includes information on the software provided by Cray and the software provided by the 
ARCHER2 CSE Service at EPCC.

If the software you require is not currently available or you are having trouble with the installed
software then please contact
[the ARCHER2 Service Desk](https://www.archer2.ac.uk/support-access/servicedesk.html) and they
will be able to assist you.

{% include links.md %}

