---
title: "Connecting to ARCHER2 and transferring data"
teaching: 10
exercises: 10
questions:
- "How can I access ARCHER2 interactively and transfer data?"
objectives:
- "Understand how to connect to ARCHER2."
- "Know how to transfer data onto and off of ARCHER2 efficiently."
keypoints:
- "ARCHER2's login address is `login.archer2.ac.uk`."
- "The password policy for ARCHER2 is well documented."
- "There are a number of ways to transfer data to/from ARCHER2."
---

## Connecting using SSH

The ARCHER2 login address is

```
login.archer2.ac.uk
```
{: .bash}

Access to ARCHER2 is via SSH using either password or SSH key pair.

## Passwords and password policy

When you first get an ARCHER2 account, you will get a single-use password from the 
SAFE which you will be asked to change to a password of your choice. Your chosen 
password must have the required complexity as specified in the
[ARCHER2 Password Policy](https://www.archer2.ac.uk/about/policies/passwords_usernames.html).

The password policy has been chosen to allow users to use both complex, shorter passwords and
long, but comparatively simple passwords. For example both `LA10!£lsty` and `horsebatterystaple`
would be supported.

> ## Picking a good password
> Which of these passwords would be a good, valid choice according to the ARCHER2 Password
> Policy?
> * `rainbowllamajumping`
> * `horsebatterystaple`
> 
{: .challenge}

## SSH keys

As well as password access, users can add the public part of an SSH key pair to access ARCHER2.
Once you are able to log in using your password you can append the public part of an SSH key 
pair to the `~/.ssh/authorized_keys` file on ARCHER2 to allow you to access the system using 
SSH keys.

The ARCHER2 User and Best Practice Guide has more information on how to create SSH key pairs
and how to use SSH Agents to make access more convenient while remaining secure. See:

* [Connecting to ARCHER2](https://docs.archer2.ac.uk/user-guide/connecting.html)

## Data transfer services: scp, rsync, Globus Online

ARCHER2 supports a number of different data transfer mechanisms. The one you choose depends
on the amount and structure of the data you want to transfer and where you want to transfer
the data to. The three main options are:

* `scp`: The standard way to transfer small to medium amounts of data off ARCHER2 to any other location
* `rsync`: Used if you need to keep small to medium datasets synchronised between two different locations
* *Globus Online*: Used to transfer large amounts of data to other sites which are Globus Online enabled

More information on data transfer mechanisms can be found in the ARCHER2 User and Best Practice Guide:

* [Data management and transfer](https://docs.archer2.ac.uk/user-guide/data.html)

## Data transfer best practice

There is a lot of information available in the ARCHER2 documentation on how to transfer data using the
methods above and how to make it efficient in the documentation linked above.

Here are the main points you should consider:

* **Not all data are created equal, understand your data.** Know what data you have. What is your
  critical data that needs to be copied to a secure location? Which data do you need in a different
  location to analyse? Which data would it be easier to regenerate rather than transfer? You should
  create a brief data management plan laying this out as this will allow you to understand which
  tools to use and when.
* **Minimise the data you are transferring.** Transferring large amounts of data is costly in both
  researcher time and actual time. Make sure you are only transferring the data you need to transfer.
* **Minimise the number of files you are transferring.** Each individual file has a static overhead in
  data transfers so it is efficient to bundle multiple files together into a single large
  archive file for transfer.
* **Does compression help or hinder?** Many tools have the option to use compression (e.g. `rsync`,
  `tar`, `zip`) and generally encourage you to use them to reduce data volumes. However, in some cases,
  the time spent compressing the data can take longer than actually transferring the uncompressed
  data; particularly when transferring data between two locations that both have large data transfer
  bandwidth available.
* **Be aware of encryption overheads.** When transferring data using `scp` (and `rsync` over `scp`)
  your data will be encrypted introducing a static overhead per file. This issue can be minimised by
  reducing the number files to be transferred by creating archives. You can also change the encryption
  algorithm to one that involves minimal encryption. (**TODO** Check what minimal ciphers are available
  on ARCHER2 as arcfour may be disabled in newer OpenSSL.)

> ## Creating an uncompressed zip archive and verifying the contents
> Using the documentation above, find the command you would use to create an uncompressed zip archive
> file of all data within a directory called `large_data_output/`. What command would you use to verify
> that the archive file you have created is not corrupt so you can safely delete the original data?
{: .challenge}

**TODO** Add the answer for the challenge

{% include links.md %}

