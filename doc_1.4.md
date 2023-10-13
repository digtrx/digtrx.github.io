---
title: 1.4 - Configuring and Securing the Guest VM
layout: home
---

## Change the root password
1. Login as root into the VM
2. Change the root password to something strong:
	```bash
	passwd
	```

## Make a Super User
*NOTE: Creating a super user is a good idea. Sometimes as the root user, you may execute commands that were not intended to run as root and could cause system issues. Also for security reasons, only the super user account will be enabled for SSH access.*
1. Login as the non-root user that was created during installation.
2. Change the password to something strong for the non-root user:
	```bash
	passwd
	```
3. Install `sudo` command:
	```bash
	apt install sudo
	```
4. To add a user to the sudo group:
	```bash
	usermod -aG sudo <non_root_username>
	```
5. Login as the non-root user, and run: 
	```bash
	sudo whoami
	```
6. If it shows root, this means the user now has super user privileges.
	- Super users who run commands prefixed with "sudo" are running commands with root permissions. 
	- If super users run commands without that prefix, the command will run with whatever permissions that user has normally.

### References
https://linuxize.com/post/how-to-add-user-to-sudoers-in-debian/

## Configure Networking

### Assigning an IPv4 Address and Gateway
1. The VM is using the "vmbr0" adapter on Proxmox, which uses the `192.168.100.0/24` network.
2. Find out which interface is enabled on the VM:
	```bash
	ip a
	```
3. After listing the interfaces, there should only be two interfaces: 
	- "lo" which stands for loopback.
	- "ens18" which will be the Ethernet interface to connect to the network.
4. To assign a manual IPv4 address to the interface, Edit `/etc/network/interfaces` as root user. Copy and paste the following information into the file:
	```
	# The primary network interface
	auto ens18
	iface ens18 inet static
	 address 192.168.100.100
	 netmask 255.255.255.0
	 gateway 192.168.100.1
	 dns-nameservers 1.1.1.1 8.8.8.8
	```
5. Restart the networking service:
	```bash
	sudo systemctl restart networking.service
	```
6. Enable the "ens18" network adapter:
	```bash
	sudo ifup ens18
	```
7. Verify the interface has the proper configuration with: 
	```bash
	ip a
	```

### Fixing DNS Issues
*NOTE: Sometimes DNS nameservers may not save accordingly. Therefore, a file used to find DNS servers needs to be updated.*
1. Create a `resolv.conf` file:
	```bash
	sudo vi /etc/resolv.conf
	```
2. Add name servers by copying the following information into the file:
	```
	nameserver 1.1.1.1
	nameserver 8.8.8.8
	```
3. Save the file.
4. Test the configuration by pinging common websites like "google.ca", "mohawkcollege.ca", etc... 

### References
https://wiki.debian.org/NetworkConfiguration
https://www.cyberciti.biz/faq/add-configure-set-up-static-ip-address-on-debianlinux/
https://askubuntu.com/questions/134121/how-to-restore-recreate-etc-resolv-conf-files

## Update and Upgrade

### Fixing APT Repository Issues
*NOTE: There may issues with getting the proper repositories for updates. Verify the following information and perform any necessary steps.*
1. Open up the repository list file:
	```bash
	vi /etc/apt/sources.list
	```
2. Delete the "cdrom" line. The Debian CD ROM will not be used update packages.
3. Copy and paste the following information into the file:
	```
	deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
	deb http://deb.debian.org/debian-security/ bookworm-security non-free contrib main non-free-firmware
	deb http://deb.debian.org/debian/ bookworm-updates non-free contrib main non-free-firmware
	```
4. Update the repositories:
	```bash
	sudo apt update
	```
5. No errors should return.

### Upgrading the System
*NOTE: Debian, like most distros, should be upgraded on a regular basis. To do that, the repositories must be updated to get the latest package versions. Then listing out and verifying what packages need to be upgraded before upgrading.*
1. Update repositories (skip this step if already updated): 
	```bash
	sudo apt update
	```
2. List all packages that have newer versions available: 
	```bash
	sudo apt list --upgradeable
	```
3. Upgrade all packages:
	```bash
	sudo apt upgrade
	```
4. User interaction may need be needed to accept the upgrade of certain packages. It's there as a safety precaution.
5. After upgrading is completed, verify that no packages need upgrading:
	```bash
	sudo apt list --upgradeable
	```

### Unattended Upgrades
*NOTE: Manually updating and upgrading can be quite tedious, especially for upgrades that are always going to be needed, like security upgrades. Having an automated program will help keep the server up-to-date.*
1. Install Unattended Upgrades:
	```bash
	sudo apt install unattended-upgrades
	```
2. Enable Unattended Upgrades and create the `20auto-upgrades` file:
	```bash
	sudo dpkg-reconfigure -plow unattended-upgrades
	```
3. Edit the following information in the `/etc/apt/apt.conf.d/20auto-upgrades` file:
	```
	APT::Periodic::Update-Package-Lists "1";
	APT::Periodic::Unattended-Upgrade "1";
	APT::Periodic::Download-Upgradeable-Packages "1";
	APT::Periodic::AutocleanInterval "7";
	```
4. Verify Unattended Upgrades is running:
	```bash
	sudo systemctl status unattended-upgrades
	```
5. Manually run Unattended Upgrades in debugging mode to see if any errors occur: 
	```bash
	sudo unattended-upgrade -d
	```

### References
https://www.fosslinux.com/113801/fixing-repository-does-not-have-a-release-file-error-in-ubuntu-and-debian.htm
https://wiki.debian.org/SourcesList
https://gist.github.com/hakerdefo/5e1f51fa93ff37871b9ff738b05ba30f
https://unix.stackexchange.com/questions/20504/the-difference-between-deb-versus-deb-src-in-sources-list
https://wiki.debian.org/UnattendedUpgrades

## Install Basic Tools
*NOTE: Although Debian comes with many packages pre-installed, there are still some that are necessary and are missing.*

*It's also possible that some required packages may not be in the Debian repositories. If this is the case, other repositories would need to be added to the [[1.4 - Configuring and Securing the Guest VM#Update and Upgrade#Fixing APT Repository Issues|Repositories File]] that was mentioned earlier.

### Installing VIM
1. Search for the VIM package to see if it is currently in the Debian repository:
	```bash
	sudo apt search vim
	```
2. If needed, a package can be inspected further to view details:
	```bash
	sudo apt show vim
	```
3. Install vim: 
	```bash
	sudo apt install vim
	```

### References
https://itsfoss.com/apt-search-command/

## Install QEMU Guest Agent
*NOTE: The qemu-guest-agent is a helper daemon, which is installed on the guest VM. It is used to exchange information between the host and the guest, and to execute commands in the guest from the host.*

### Check for QEMU Guest Installation on VM
1. Check if it is installed already:
	```bash
	sudo apt list --installed | grep qemu
	```
2. Install guest agent:
	```bash
	sudo apt install qemu-guest-agent
	```

### References
https://pve.proxmox.com/wiki/Qemu-guest-agent

## Disable the use of IPv6 as it's not needed
*NOTE: Although IPv6 is the newest version of the IP protocol, it is not always used. For this project, IPv6 will be disabled. Keeping IPv6 enabled will make administrating the firewall more tedious, and because it is definitely not being used for this project, there is no point in enabling it.*
1. Using the command to edit this configuration file:
	```bash
	sudo vim /etc/sysctl.conf
	```
2. Add the following lines to the bottom of the file:
```bash
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
3. Save the file.
4. Reboot the machine.

## Disable any services that aren't needed (ie. telnet)
*NOTE: Sometimes Linux distros or other packages that are installed may enable and start services that aren't needed. This is a problem as these services will affect performance and can be misused by attackers if they are vulnerable.*

*The best solution is to stop and disable any services that you do not require.*

*In the Debian 12 VM used for this project, there were no unneeded services running. However, it is still good practice to check and verify that no unnecessary services are running and to disable them.*
1. List services: 
	- List all services:
		```bash
		sudo systemctl list-unit-files --type service -all
		```
	- List loaded services, with status and description: 
		```bash
		sudo systemctl list-units --type=service
		```
	- List active services: 
		```bash
		sudo systemctl list-units --type=service --state=active
		```
	- List running services: 
		```bash
		sudo systemctl list-units --type=service --state=running
		```
2. Disabling/Enabling Services
	- Check status of service: 
		```bash
		sudo systemctl status service.name
		```
	- Check if service is enabled: 
		```bash
		sudo systemctl is-enabled service.name
		```
	- Stop a service: 
		```bash
		sudo systemctl stop service.name
		```
	- Start a service: 
		```bash
		sudo systemctl start service.name
		```
	- Disable and Stop a service: 
		```bash
		sudo systemctl disable --now service.name
		```
	- Enable and Start a service: 
		```bash
		sudo systemctl enable --now service.name
		```
	- *A service can be masked using the `mask` or `unmask` command-line arguments so that it cannot ever be manually started.*

### References
https://documentation.suse.com/smart/systems-management/html/reference-systemctl-enable-disable-services/index.html

## Install and Configure SSH
*NOTE: Unlike Proxmox, which already had SSH pre-installed, Debian 12 needs to have it installed. The process is simple and securing the SSH server is similar to that of the Proxmox server.*

### Install OpenSSH Server
1. Install the OpenSSH server: 
	```bash
	sudo apt install openssh-server
	```
2. Check if the SSH server is running: 
	```bash
	sudo systemctl status sshd
	```
3. Confirm that the SSH server is now listening on the default port "22": 
	```bash
	sudo ss -tulpn
	```
4. Using the Proxmox server (not the VM), login using SSH to the VM:
	```bash
	ssh <non_root_username>@192.168.100.100
	```
	- Alternatively the root account can be used to login to test the connection. However, root login will be disabled in the next steps.

### Configure SSH Server
1. Edit the SSH Server configuration:
	```bash
	sudo vim /etc/ssh/sshd_config
	```
2. There are many settings to change within the file. Find the settings listed below and make the changes:
	```
	# Since we are not using a GUI environment, disable X11Forwarding
	X11Forwarding No
	
	# This changes the default port from 22 to 2022. This is considered a security deterrent.
	Port 2022
	
	# The server will only listen for IPv4 (inet) connections.
	AddressFamily inet
	
	# This ensures that no one can ever SSH in as the root user.
	PermitRootLogin no
	
	# Set a timeout interval for SSH sessions.
	ClientAliveInterval 180
	```
3. Save the changes.
4. Reload or Restart the SSHd service to update the configuration: 
	```bash
	sudo systemctl reload sshd
	```

### Generate SSH Keypair
*NOTE: Strong passwords are good deterrents against brute forcing, but what works even better are public-private key pairs. They are very convenient and provide extreme brute forcing protection as key lengths can be thousands of characters long.*
1. On the Proxmox server that will access the Debian 12 VM, create a 4096-bit keypair:
	```bash
	ssh-keygen -b 4096
	```
2. Save the file in the default directory.
3. Optionally, a passphrase can be used to secure the key incase someone else tries to use that key to SSH into the Proxmox server.
4. Next, the public key has to be copied from Proxmox server to the Debian 12 VM:
	```bash
	ssh-copy-id <non_root_username>@192.168.100.100
	```
	*Using this command will also give the key the correct permissions on the server for it to work properly.*

### Test SSH Connection
1. From the Proxmox server, SSH into the Debian 12 VM:
	```bash
	ssh <non_root_username>@192.168.100.100
	```
	- *NOTE: There should be no prompt for a password unless a passphrase was entered during SSH key setup.*
2. If SSH was successful, edit the SSHd configuration file:
	```bash
	sudo vim /etc/ssh/sshd_config
	```
3. From the information below, make the following changes in the configuration file:
	```
	# Password based logins are disabled - only public key based logins are allowed.
	AuthenticationMethods publickey
	
	# Disable Password Authentication
	PasswordAuthentication no
	
	# LogLevel VERBOSE logs user's key fingerprint on login. Needed to have a clear audit track of which key was using to log in.
	LogLevel VERBOSE
	
	# Log sftp level file access (read/write/etc.) that would not be easily logged otherwise.
	Subsystem sftp  /usr/lib/ssh/sftp-server -f AUTHPRIV -l INFO
	```

### Disable Root Account Login
*NOTE: Disabling root login means no one will ever be able to login in using "root" as the username through SSH.*
1. Edit the SSHd configuration file:
	```bash
	sudo vim /etc/ssh/sshd_config
	```
2. Find the line that says `PermitRootLogin Yes`
3. Change the `Yes` to `No` so it shows `PermitRootLogin No`
4. Save the file.
5. Test by trying to SSH in as "root" user:
	```bash
	ssh root@192.168.100.100
	```
	- The above command should fail.

### References
https://devconnected.com/how-to-install-and-enable-ssh-server-on-debian-10/
https://wiki.debian.org/SSH#Installation_of_the_server
https://www.ssh.com/academy/ssh/sshd_config

https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html
https://infosec.mozilla.org/guidelines/openssh
https://linuxhandbook.com/ssh-hardening-tips/
https://www.virtuesecurity.com/kb/ssh-weak-key-exchange-algorithms-enabled/

## Install and Configure UFW
*NOTE: The main advantage to running a Firewall on the server is to limit the connections coming in or going out of the machine. Some services may require access into the machine, such as SSH. Other services should be blocked unless they are needed.*

*As mentioned before, UFW uses iptables or nftables (depending on the OS) and simplifies managing rules.*
1. Install UFW: 
	```bash
	sudo apt install ufw
	```
2. Check the status of UFW:
	```bash
	sudo ufw status
	```
	 - *By default, UFW is always inactive to allow administrators to set up rules and then manually enable the firewall.*
3. Edit the following file:
	```bash
	sudo vim /etc/default/ufw
	```
4. Change the following line in the UFW configuration file:
	```
	IPV6=no
	```
	- *This will disable IPv6 firewall rules.*
5. Check to see what connection is currently being used by SSH:
	```bash
	ss -tpn
	```
	Results:
	```
	Local Address:Port          Peer Address:Port
	192.168.100.100:2022        192.168.100.1:51292
	```
	- *The Proxmox machine at "192.168.100.1" port 51292 is currently using SSH to access the Debian 12 VM at "192.168.100.100" port 2022.*
	- *Using this information, a firewall rule can be generated to prevent SSH access from being blocked when UFW is enabled.*
6. Allow SSH from the Proxmox server to the Debian 12 VM:
	```bash
	ufw allow from 192.168.100.1 proto tcp to 192.168.100.100 port 2022
	```
7. By default, all other inbound connections will be dropped. Firewalls generally **allow all outgoing connections**, and **drop all incoming connections**.
8. Enable the firewall:
	```bash
	sudo ufw enable
	```
9. Ensure that SSH is still working by logging out and logging back in again.
10. Check `journalctl` logs for UFW:
	```bash
	sudo journalctl | grep -i ufw
	```

## Install and Configure CrowdSec
*NOTE: For more information on CrowdSec, please refer to [[1.2 - Securing the Proxmox Server#Use CrowdSec Host-based IPS]]*

### Installation
1. Add the official CrowdSec repos:
	```bash
	curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
	```
2. Install CrowdSec:
	```bash
	sudo apt install crowdsec
	```
3. Install the bouncers (for iptables and nftables):
	```bash
	sudo apt install crowdsec-firewall-bouncer-iptables && apt install crowdsec-firewall-bouncer-nftables
	```
4. List everything currently installed by CrowdSec Hub:
	```bash
	sudo cscli hub list
	```
5. List currently installed collections:
	```bash
	sudo cscli collections list
	```

### Join the Debian 12 VM to the Managed CrowdSec Account
*NOTE: Once CrowdSec is installed, the server can then be added to the user's CrowdSec web GUI console for easy management and monitoring.*
1. Log into CrowdSec (or create an account) https://www.crowdsec.net/
2. Click "Add Security Engine"
3. It will give a command to run on the Debian 12 VM:
	```bash
	sudo cscli console enroll <some_unique_string>
	```
4. The unique string is there so CrowdSec knows that the Debian 12 VM wants to join the CrowdSec account.
5. Back on the Web GUI, an "Enrollment Request" will appear, from which it can be accepted or denied. Make sure to accept the request from the Debian 12 VM.
6. Restart the CrowdSec service:
	```bash
	sudo systemctl restart crowdsec.service
	```
7. Verify CrowdSec service is running:
	```bash
	sudo systemctl status crowdsec.service
	```
8. Now on the CrowdSec Web GUI, the Debian 12 VM's name can be changed and the VM can be easily managed to perform tasks like: adding a blocklist of IPs, seeing alerts, managing scenarios, installing bouncers, etc.

### Install Collections
*NOTE: By default, SSH is already covered by crowdsec collection.*

#### Kasm
*NOTE: Kasm can be protected from brute force attempts on its out-facing web GUI.*
1. Find Kasm collections:
```bash
sudo cscli collections list -a | grep kasm
```
2. Install the collections:
```bash
sudo cscli collections install crowdsecurity/kasm
```
3. Reload the CrowdSec config:
```bash
sudo systemctl reload crowdsec
```

### Upgrading Collections
*NOTE: Similar to how the OS is updated and upgraded with `sudo apt update && sudo apt upgrade`, the collections from CrowdSec can also be updated and upgraded.*
1. List all collections with: 
	```bash
	sudo cscli collections list
	```
2. Update from CrowdSec with: 
	```bash
	sudo cscli hub update
	```
3. Choose an individual collection to upgrade with: 
	```bash
	sudo cscli collections upgrade <collection_name>
	```
4. Upgrade all collections with: 
	```bash
	sudo cscli hub upgrade
	```

### Allowing CrowdSec to Read Journald
*NOTE: Traditionally, Debian has used `syslog` to log all events into files and store them in predictable locations for users and log parsers to view. However, with the new Debian 12 release, they have decided to use `journald` as a replacement. *

*Journald uses `journalctl` to easily parse through log information and is considered a better alternative to `syslog`*
1. Edit the following the file to change how CrowdSec acquires its log information:
```bash
sudo vim /etc/crowdsec/acquis.yaml
```
2. Verify or add the following information for SSH:
```
journalctl_filter:
 - _SYSTEMD_UNIT=ssh.service
labels:
  type: syslog
```
3. For every service that uses `journald` to log files that is to be monitored by CrowdSec must have a similar entry to the above in the `acquis.yaml` file.

### References
https://www.crowdsec.net/
https://doc.crowdsec.net/docs/next/getting_started/install_crowdsec
https://www.youtube.com/watch?v=2Ec-FYmK4zg
