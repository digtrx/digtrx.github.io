---
title: 1.3 - Creating a Guest VM
layout: home
---

## Choosing a Linux distro (Debian 12)
The reasons for choosing a distribution of Linux for a server are based on:
- Stability of the operating system
- Longevity of support
- Performance

There are many good contenders for a server OS. However, this project will use Debian 12 as it has just released in June of 2023 with good reviews. It is planned to have 5 years of guaranteed support.

Debian also has a good track record, like other distros, about being performant, reliable, and stable, which are core characteristics for server operation.

### References
https://news.itsfoss.com/debian-12-features/
https://www.debian.org/News/2023/20230610

## Configuring the Guest VM in Proxmox
Creating a virtual machine in Proxmox is similar to creating one in VirtualBox, with a few configuration changes that should be made. What has to be noted here is that Proxmox is made to be highly configurable as it was designed for an enterprise datacenter environment and most settings can be ignored or set to default. However, some settings will be changed so our VM works accordingly.

### Uploading the Linux Distro ISO Image
1. Visit https://www.debian.org/download
2. Download the latest version of Debian 12 "Bookworm".
3. Verify the file against the provided SHA512 Checksum on the Debian website.
4. Login to the Proxmox web GUI.
5. Under server settings, in "local storage", click "ISO images".
6. From this window, an ISO can be uploaded from the local machine or it can be downloaded directly from a URL.
7. Since the Debian 12 ISO was downloaded already, upload it from the local machine.

### Create a Virtual Network
1. In the Proxmox web GUI, click on the Proxmox server in the left hand pane.
2. A middle pane will open up with many more sub-sections. Expand "System".
3. Click on "Network"
4. There should be a network named "vmbr0" and the type should be "Linux Bridge"
5. Double click "vmbr0" to open up the settings editor.
6. Set *IPv4/CIDR* to 192.168.100.1/24
	- This will be the network that the VM will be on.
7. Click "OK"

### Create a Virtual Machine
*NOTE: In the VM creation process, there are many settings. Only change the ones that are listed below. Leave all other settings set to default.*
1. **General**
	1. *Node*: This is the node that will host the VM. (There is only one physical server at this time.)
	2. *VM ID*: This is a unique number automatically assigned to the VM for administration.
	3. *Name*: The name of the VM.
2. **OS**
	1. Select "local Proxmox storage"
	2. Choose the Debian 12 OS that was uploaded earlier.
	3. In "*Guest OS*" settings, select "Linux: 6.x-2.6 Kernel"
3. **System**
	1. *QEMU Agent*: Enabled (tick the box)
		- The QEMU agent lets Proxmox VE know that it can use QEMU features to show some more information, and complete some actions more intelligently (i.e. shutdown or take snapshots).
4. **Disks**
	1. Enable "Advanced" at the bottom of the window (tick the box)
	2. Enable *Discard* and *SSD Emulation* (tick both boxes) for LVM Thin provisioning.
		- LVM Thin provisioning allows Proxmox to take snapshots of the VM, which is good for rolling back to an earlier version in case there are issues.
5. **CPU**
	1. Set *Cores* to 4
	- This number will different depending on the physical processor (CPU) used in the server. The CPU in this project has 8 CPUs available for use, so assigning it 4 cores will be ok.
	- This number can be edited later in case the VM needs more CPU performance. The only way to check is to monitor the CPU performance once the VM is started and running.
	- The ideal situation is to only give it enough CPUs to run properly.
6. **Memory**
	1. Set *Memory (MiB)* to 12288
	- This will set the amount of memory (RAM) to 12 GB.
	- Again, this is a number that depends on how much physical RAM the server has. For this project, 16 GB is equipped. Therefore, 12 GB is a safe number.
	- Like the CPU, memory usage can be monitored once the VM is running to determine if the VM needs more memory.
7. **Network**
	1. Set *Bridge* to "vmbr0"
	2. Set *Firewall* to Disabled (untick the box)
		- The firewall will be managed from UFW and iptables inside the VM.
8. **Confirm**
	1. If the settings look good, click "Finish"

### References
https://pve.proxmox.com/pve-docs/chapter-qm.html#qm_virtual_machines_settings
https://www.makeuseof.com/best-virtual-machine-linux-kvm-virtualbox-qemu-hyper-v/

## Installing Debian 12
1. In Proxmox web GUI, once the VM is created, it will be listed underneath the Proxmox server name in the left hand pane.
2. Right click on the VM and click "Start"
3. Then navigate to the "Console" view in the middle pane to interact with the VM.
4. Skip the network setup, as an IPv4 address will be configured manually later.
5. The account setup will ask for passwords for a root user and a non-root user. Enter simple credentials. Passwords can be changed later.
	- The non-root user was named `capstone` for this project.
6. For disk partitioning, leave everything set to default.
7. Skip the update section.
8. For Desktop Environments, De-select *GNOME* and *Debian desktop environment*.
	- This means there will be no desktop to interact with, but this preferred to keep the server as lightweight as possible.
9. Debian will finish installing and reboot automatically.

### References
https://ostechnix.com/install-debian-12-bookworm/