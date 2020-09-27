---
layout: post
comments: true
title: Manually install WSL distribution to a custom location
feature-img: "assets/img/pexels/book-glass.jpeg"    # Add a feature-image to the post
#thumbnail: "assets/img/thumbnail/desk-messy.jpeg"   # Add a thumbnail image on blog view
categories: [blog]
tags: [wsl, windows]
#series: "Blogging with Jekyll"
---

## Table of contents
* TOC
{:toc}
----
Installing a linux distro in Windows with WSL will result in the linux virtual disk to be install in the C driver. If you have a small SSD like in my Windows tablet, you know why we need to move the distro to some other folder. This article will show you how.

WSL command line allows to export distribution to a tar file, or to import from a tar file as a new distribution.

## Get the linux distribution

### Export existing distribution

List a distribution installed from App Store:

PS E:\Temp> wsl --list --all -v
    NAME            STATE           VERSION
    * Ubuntu-20.04    Stopped         2

Export it:
    
    PS E:\Temp> wsl --export Ubuntu-20.04 E:\WSL2\ubuntu.tar


### Download a distribution 

Some wsl launcher projects like [wsldl](https://github.com/yuk7/wsldl) have root filesystem created and can be install with their launcher. We can download one like [FedoraWSL](https://github.com/yosukes-dev/FedoraWSL). Download and unzip the file to get root filesystem.
    rootfs.tar.gz

## Import and start the distribution

The tar file contains a root filesystem of the Ubuntu distribution. And I can import that as a new distribution:

    PS E:\Temp> wsl --import Ubuntu-New E:\WSL2\Ubuntu-New E:\WSL2\ubuntu.tar

or 
    
    PS E:\Temp> wsl --import Fedora32 E:\WSL2\Fedora E:\WSL2\rootfs.tar.gz

This will create a folder Ubuntu-New folder in the E:\WSL2 and the ext4.vhdx file in that folder. Virtual hard disk is an ext4 partitioned disk containing the contents of the tar file. Listing distributions now shows two of them.
    PS E:\Temp> wsl --list --all -v
    NAME            STATE           VERSION
    * Ubuntu-20.04    Stopped         2
    Ubuntu-New     Stopped         2


If I start the second distribution, I will be logged in as a root account:
    PS E:\Temp> wsl -d Ubuntu-Copy
    root@mDESKTOP-V38C2TR:/mnt/e/Temp#

By unregistering the distribution like wsl --unregister Ubuntu-20.04 it will delete the associated ext4.vhdx file.

## Change default user when starting wsl distro

Create and edit /etc/wsl.conf:

    [user]
    default=<string>

Restart wsl distro
    wsl --shutdown
    wsl
    [td@DESKTOP-V38C2TR Thuy Dang]$

## Using linux commands in Windows folder

Run Linux binaries from the Windows Command Prompt (CMD) or PowerShell using wsl <command> (or wsl.exe <command>).

For example:

    C:\temp> wsl ls -la
    <- contents of C:\temp ->

Binaries invoked in this way:

* Use the same working directory as the current CMD or PowerShell prompt.
* Run as the WSL default user.
* Have the same Windows administrative rights as the calling process and terminal.

The Linux command following wsl (or wsl.exe) is handled like any command run in WSL. Things such as sudo, piping, and file redirection work.

Example using sudo to update your default Linux distribution:

    C:\temp> wsl sudo apt-get update

[More tricks](https://docs.microsoft.com/en-us/windows/wsl/interop).


## References
* https://dev.to/milolav/manually-installing-wsl2-distributions-41b4
* https://github.com/yuk7/wsldl
* https://docs.microsoft.com/en-us/windows/wsl/interop
* https://www.sitepoint.com/wsl2/
