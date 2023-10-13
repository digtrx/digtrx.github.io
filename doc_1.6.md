---
title: 1.6 - Kasm Installation and Configuration
layout: home
---

## What is Kasm Workspaces?
Kasm Workspaces is a docker container streaming platform for delivering browser-based access to desktops, applications, and web services. 

Kasm uses devops-enabled Containerized Desktop Infrastructure (CDI) to create on-demand, disposable, docker containers that are accessible via a web browser. 

Example use-cases include: Remote Browser Isolation (RBI), Data Loss Prevention (DLP), Desktop as a Service (DaaS), Secure Remote Access Services (RAS), and Open Source Intelligence (OSINT) collections.

The rendering of the graphical-based containers is powered by the open-source project KasmVNC.

### Why would this be needed?
There are several reasons why streaming containers may be suitable in an enterprise environment:
- First, there is **centralization**. Instead of installing software on many different computers, it can be centralized into a data center running on a few different servers. Also, centralization makes management easier as most of what the user will interact with will be on a server close to technical support.
- Second is **security**. By having users authenticate and log in to the Kasm server, working environments can be better controlled and monitored for malicious activity.
- Third is **cost**. There will definitely be cost savings as employees will no longer need powerful laptops or computers to run the software. They just need a device that can securely connect across the web using HTTPS through a modern web browser (ie. Chrome, Firefox, Edge, etc.)

### References
https://kasmweb.com/
https://docs.linuxserver.io/images/docker-kasm


## Add Routes to access Kasm from Local machine
*NOTE: Because the local machine is not able to directly connect to the Debian 12 VM, the Proxmox Server must act as a router. Therefore it will forward any connections it sees destined for Kasm to the Debain 12 VM.*

*<Proxmox_server_IP> refers to the static IP address assigned by the physical routing device in [[1.1 - Configuring a Type 1 Hypervisor#Accessing Proxmox]]*.
1. On the Proxmox Server, add a route that will forward traffic destined to Proxmox server at port 443 to the Debian 12 VM (192.168.100.100) at port 443:
	```bash
	sudo iptables -t nat -A PREROUTING -p tcp -d <Proxmox_server_IP> --dport 443 -i wlan0 -j DNAT --to-destination 192.168.100.100:443
	```

Alternatively, updating the `/etc/network/interfaces` file will also work:
1. Edit the interfaces configuration file:
	```bash
	sudo vim /etc/network/interfaces
	```
2. Append the following data to the end of the file:
	```
	# Kasm Web GUI
	post-up   iptables -t nat -A PREROUTING -p tcp -d <Proxmox_server_IP> --dport 443 -i wlan0 -j DNAT --to-destination 192.168.100.100:443
	post-down   iptables -t nat -D PREROUTING -p tcp -d <Proxmox_server_IP> --dport 443 -i wlan0 -j DNAT --to-destination 192.168.100.100:443
	```
3. Save the file.
4. Reload the network configuration:
	```bash
	sudo systemctl reload networking
	```

###  References
https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands

## Add UFW Firewall Rules to access Kasm from Local machine

### Proxmox Server
*NOTE: Because traffic is being forwarded through the Proxmox server, it has to be allowed through the Proxmox firewall. Otherwise, the default forward policy is to drop all packets.*
1. Allow traffic to be forwarded from an incoming connection to the Proxmox server on port 443 (for Kasm web GUI) from the local desktop to the Debian 12 VM:
	```bash
	sudo ufw route allow in on wlan0 from <local_desktop_IP> out on vmbr0 to 192.168.100.100 port 443 proto tcp comment "Allow Kasm Web GUI Access from Desktop"
	```

### Debian 12 VM
1. Add a rule to allow the forwarded HTTP traffic from the Proxmox Server to reach the Kasm web GUI on the Debian 12 VM:
	```bash
	sudo ufw allow from 192.168.100.1 to 192.168.100.100 proto tcp port 443 comment "Allow Kasm Web GUI access from Proxmox Server"
	```

### References
https://www.cyberciti.biz/faq/how-to-configure-ufw-to-forward-port-80443-to-internal-server-hosted-on-lan/
https://www.digitalocean.com/community/tutorials/how-to-forward-ports-through-a-linux-gateway-with-iptables
https://askubuntu.com/questions/161346/how-to-configure-ufw-to-allow-ip-forwarding

## Installing Kasm
*NOTE: See the official Kasm installation documentation for more information:* https://kasmweb.com/docs/latest/install/single_server_install.html#installation-guide
1. Change into the `/tmp` directory:
	```bash
	cd /tmp
	```
2. Download the Kasm 1.14.0 release tar ball:
	```bash
	curl -O https://kasm-static content.s3.amazonaws.com/kasm_release_1.14.0.3a7abb.tar.gz
	```
3. Unzip the tarball:
	```bash
	tar -xf kasm_release_1.14.0.3a7abb.tar.gz
	```
4. Run the Kasm install script with root privileges:
	```bash
	sudo bash kasm_release/install.sh
	```
5. Accept the license agreement.
6. The installation will take a long time.
7. After the installation is complete, a list of credentials printed on the command line.
8. Copy those credentials to a safe place (i.e. on the local desktop in an encrypted file).

## Configuring Kasm
1. From a local computer, access the Kasm web GUI:
	```bash
	https://<Proxmox_server_IP>:443
	```
2. The browser will display a warning message, because the self-signed Kasm certificate is not trusted. However, the connection is still encrypted with TLS 1.3.
3. Login using the `admin@kasm.local` credentials.
4. Click "Admin" at the top of the page.

### Add a new user
1. In the left pane, many administrative settings are available. Expand "Access Management"
2. Click "Users"
3. Click "Add User"
4. Create a new user that will be the new administrator (Fill out only the required fields).
	- *Make sure the chosen username is in the format: `<username>@kasm.local`*
5. Click "Save"

### Add user to Admin group
1. In the left pane, under "Access Management", click "Groups"
2. Select "Administators" and then click "Edit"
3. In the top row, click "Users"
4. Click "Add User"
5. Find the newly created user and click "Submit"
6. Log out of the default admin account and log in as the new user. Verify the "Admin" panel can be accessed.

### Disable default accounts
*NOTE: It is always good practice to change default credentials or disable default accounts to prevent brute-force attacks using default credentials.*
1. In the left pane, under "Access Management", click "Users"
2. For the `admin@kasm.local` and `user@kasm.local`, Disable both accounts.

## Test Kasm Functionality

### Add a Workspace
1. In the "Admin" panel, in the left pane, expand "Workspaces"
2. Click "Registry"
3. Any application from the official Kasm registry can be used to test the functionality. For this task, Brave will be installed and tested.
4. Select "Brave", and click "Install"
5. Click "Installed Workspaces" at the top of the list.
6. Observe the installation of "Brave" and wait for completion.
7. Once completed, at the top of the page, click "Workspaces"
8. Click "Brave"
9. Select "Current Tab" and click "Launch Session"
10. Once "Brave" opens, try navigating to https://www.google.ca/
	- If Google loads, this means that the "Brave" Docker container is able to reach out to the Internet successfully.
	- If not, this means that there is a connectivity issue in the network