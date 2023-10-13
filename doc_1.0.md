---
title: 1.0 - Choosing the Correct Server
layout: home
---

## What is a Server?
In a nutshell, a server is like any other computer, except it is dedicated to hosting applications or services over an extensive period of time. Unlike a home computer that may be turned on for a few hours a day to perform medial tasks, a server is usually up 24/7 and is constantly under many different loads to host web sites, file sharing services, authentication services, etc.

Servers also differentiate in hardware, as they typically have CPU's with more cores, higher amounts of RAM, higher amounts of storage, etc. Servers are also designed to be extremely redundant and power efficient. They need to be redundant because of the highly sensitive information they store, and power efficient because of their 24/7 operation.

## Server for the Project
Choosing a server to use for the project is not too difficult. Basically, I have two options: 
1. Use a cloud service provider on the internet
2. Use a physical computer on my own network

A cloud service provider would be a good choice if it was affordable. Some providers give free credits to newcomers, and these free credits could provide up to a few months of free hosting. The only downside, is that they generally give you a piece of a shared server which includes 1 virtual CPU and 1 GB of RAM. This might not sound bad if the intention is to run a simple service or application, but I would rather have more computing power.

In my possession, I have an old gaming computer that should be more than capable of fitting the criteria for a server. The requirements for Proxmox are:
- Intel EMT64 or AMD64 with Intel VT/AMD-V CPU flag.
- Memory, minimum 2 GB for OS and Proxmox VE services. Plus designated memory for guests.

The specs of the old gaming computer are:
- Intel Core i7 (8 CPUs @ 4.20 GHz) with Intel VT enabled
- 16 GB of RAM
- 250 GB of SSD storage (fast storage)
- 14 TB of HDD storage (regular storage)
- NVidia 10 Series Graphics Card (GPU passthrough for applications that need GPU rendering)
- Gigabit Ethernet NIC
- Wireless NIC

## References
https://www.proxmox.com/en/proxmox-ve/requirements