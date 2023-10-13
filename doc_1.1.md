---
title: 1.1 - Configuring a Type 1 Hypervisor
layout: home
---

## What is a Hypervisor?
A hypervisor is simply a program that "abstracts operating systems and applications from their underlying hardware" ( Reference 1). In other words, this means the hypervisor is able to carve up system resources like CPU, memory, and storage, and provision them to virtual machines.

These virtual machines are known as "guests" while the physical machine is known as the "host". In theory, the host can manage as many guests as it can handle. This strongly depends on the resources available by the host. For example, if I wanted to create 3 virtual machines using 4 cores each, but my host machine only had a 4 core processor, then I would soon run into trouble.

It is important to plan out the deployment of virtual machines by selecting the appropriate physical hardware, but also the type of hypervisor to be used. There are two types of hypervisors available: Type 1 and Type 2.

## Type 1 vs Type 2 Hypervisor
A Type 2 hypervisor is what we commonly use in school (an example would be Oracle VirtualBox). It is installed overtop of the current running operating system, which could be a Windows, Mac, or Linux machine. These hypervisors are good for launching quick and easy virtual machines, but are not good for managing lots of running virtual machines.

This is where Type 1 hypervisors come into play. They do not run overtop of an existing operating system, instead they act like an operating system. They are installed onto the "bare metal" of a physical machine. When the machine is booted up, the hypervisor environment loads without an additional operating system.

Type 1's are great because they do not have the overhead of the additional operating system, which makes them more efficient and frees up some physical resources to be provisioned. Also, Type 1's are designed for data centers that use different computing technologies compared to average home computers (ie. different storage solutions, mass amounts of RAM, various CPU designs, etc.)

## Popular Brands of Type 1 Hypervisors
Organizations have many options of type 1 hypervisors to choose from. However, through some basic research, the popular choices for hypervisors are:
- VMware ESX-i
- Microsoft Hyper-V
- Citrix XenServer
- Red Hat Enterprise Virtualization (RHEV)
- Proxmox Virtual Environment (PVE)

Although VMware ESX-i is by far the most popular choice of type 1 hypervisor, and can be obtained for free for personal use, I chose to use Proxmox Virtual Environment (PVE).

The main reason why I chose Proxmox is because it seemed to be the most user-friendly and it is a free and open-source software that is available to everybody.

## What is Proxmox?
Proxmox Virtual Environment (PVE) is a free open-source type 1 hypervisor that is actually based on Debian Linux and uses a modified Ubuntu LTS (long term support) kernel. When it is up and running, Proxmox can be accessed over the network through a web browser for administration. It uses an API to allow the web console to make any necessary changes to the server without having to physically be at the server. This makes it easy to manage and deploy virtual machines.

Although I will only be using Proxmox to do a couple of things with this project, it is highly capable of managing data centers and is a very dependable product. It has been around since 2008 and uses long-lasting stable releases of Debian as its platform.

Some of the other hypervisors operate similar to Proxmox, in that they also use web based GUI consoles to help administrators manage their server infrastructure.

## Configuring Installation Media
Proxmox, like most other operating systems, requires a bootable installation media (such as a bootable USB drive) to install itself onto the server.

First, the latest ISO image of Proxmox must be downloaded. After which, a clean USB drive will be needed. There are some different methods for making the USB drive bootable, depending on the operating system that you're using to do this.

Since I am on Windows, I will be using either a program called "Rufus" or "Etcher" to make it bootable. They're very easy programs to use and within minutes you will have a bootable USB drive with the Proxmox ISO image.

## Installing Proxmox on the Server
Prior to installing Proxmox, it should be noted that any data will be erased on the current storage drive that I choose to install Proxmox on, unless I make another partition. However, I no longer need any of the data on that drive so it can be wiped and re-partitioned using the Proxmox VE installer.

Going through the installer, it is very simple to follow and is similar to any of the Windows or other Linux distributions that we have installed in class before. It will first ask which drive to install onto, along with which file system to use. 

My choice was to use the EXT4 file system as it is fairly standard. There are some different file systems available like XFS, BTRFS, and ZFS. However, some of those are more complicated and have some draw backs that should be strongly considered before implementing them.

Next, is choosing the time zone and setting a password for the root admin account of the machine.

The actual installation begins, after which it will ask you to supply network information (if it is not automatically detected).

## Accessing Proxmox
After the installation has completed, another workstation with a web browser must be used to access the Proxmox server at its local address to start configuring it.

Upon navigating to the IP address and port, the credentials will be "root" and what ever password was entered during installation setup.

For ease of use, it is recommended to assign the Proxmox server a static IPv4 address. This can be done through the network router in which the server connects to.

## References
https://pve.proxmox.com/wiki/Installation
https://pve.proxmox.com/wiki/Prepare_Installation_Media
https://www.techtarget.com/searchitoperations/tip/Whats-the-difference-between-Type-1-vs-Type-2-hypervisor
https://en.wikipedia.org/wiki/Proxmox_Virtual_Environment