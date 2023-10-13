---
title: 1.2 - Securing the Proxmox Server
layout: home
---

## Update Proxmox (and inherently Debian 11)
*NOTE: Because Proxmox is built on Debian 11 (as for the current release), SSH can be used from a client machine to interact with the server. Also, the Proxmox web GUI provides a browser-based shell to interact with it directly.*

*Keeping the Proxmox server up-to-date is an important security related task, as some updates do include security fixes.*

To perform an update, do the following:
1. Find the Proxmox server name under the "Datacenter" section.
2. Click the "Shell" button to create an SSH session to the server.
3. Login as the default root account (using the password from initial setup).
4. Update all repositories and upgrade them:
	```bash
	apt update && apt upgrade
	```
	Example Output: `22 packages can be upgraded. Run 'apt list --upgradable' to see them.`
5. After the upgrade is complete, run the command to see if any other packages still need to be upgraded:
	```bash
	apt list --upgradeable
	```
	Example Output: `Listing... Done` shows no packages need an upgrade.

## Change the root password to something strong
1. If a strong root password was not setup during the initial installation of Proxmox, now would be the best time.
2. While logged into root, change the password to something strong:
	```bash
	passwd
	```

## Create a super-user with a strong password
*NOTE: It is a best practice to avoid using the root account unless it is **absolutely necessary**.*
1. Create a new user with `adduser USERNAME`
2. Make a strong password for the user account using `passwd USERNAME`
3. Add the user to the "sudo" group using `usermod -aG sudo USERNAME`. 
	*This will allow us to execute commands as root when it is required.*
4. Switch to the new user, and try to run some commands prefixed with "sudo" to see if you can run it as root:
	```bash
	whoami
	sudo whoami
	```

## Harden SSH Access
*NOTE: SSH is the way to administrate the server remotely. It is extremely important to keep access limited and secure to prevent misuse.*

### Use Public-Private Key Pairs to Authenticate SSH Login
*NOTE: Strong passwords are good deterrents against brute forcing, but what works even better are public-private key pairs. They are very convenient and provide extreme brute forcing protection as key lengths can be thousands of characters long.*
1. On the client machine that will access the server via SSH, create a 4096-bit keypair:
	```bash
	ssh-keygen -b 4096
	```
2. Save the file in the default directory.
3. Optionally, a passphrase can be used to secure the key incase someone else tries to use that key to SSH into the Proxmox server.
4. Next, the public key has to be copied from the client machine to the server:
	```bash
	ssh-copy-id <Proxmox_username>@<Proxmox_Server_IP>
	```
	*Using this command will also give the key the correct permissions on the server for it to work properly.*

### Test SSH Connection
1. From the local client machine, SSH into the Proxmox server using PowerShell or another terminal tool:
	```bash
	ssh <non_root_username>@<Proxmox_server_IP>
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
	ssh root@<Proxmox_server_IP>
	```
	- The above command should fail.

### Allow Only Specific IPs to SSH In
*NOTE: Please refer to [[1.2 - Securing the Proxmox Server#Use UFW (Uncomplicated Firewall) to lockdown access to the server]]*

### References
https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-debian-11

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

### References
https://support.nordvpn.com/Connectivity/Linux/1047409212/How-to-disable-IPv6-on-Linux.htm

## Disable any services that aren't needed (ie. telnet)
*NOTE: Sometimes Linux distros or other packages that are installed may enable and start services that aren't needed. This is a problem as these services will affect performance and can be misused by attackers if they are vulnerable.*

*The best solution is to stop and disable any services that you do not require.*
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

### Services Disabled on Proxmox Server

#### CUPS (Printing Service)
*NOTE: Since there is no need to use a printing service on the server, disable the built-in CUPS printing service.*
1. Check the service status:
	```bash
	sudo systemctl status cups.service
	```
2. Stop and Disable the service:
	```bash
	sudo systemctl disable --now cups.service
	```
3. Verify the status:
	```bash
	sudo systemctl status cups.service
	```

### References
https://documentation.suse.com/smart/systems-management/html/reference-systemctl-enable-disable-services/index.html

## Monitor Network Connections and Look for Any Suspicious Ports
*NOTE: All network connections going in or out of the server can be monitored with the "Socket Statistics" command. Anything that looks suspicious should be investigated further to see if it is important.*

*There are many ports that are listening for connections by default. However, for best security practices, it is important to block these ports if they are not being used.*
- Lists TCP/UDP ports that are listening, with port numbers:
	```bash
	sudo ss -tulpn
	```

- The following ports are used by Proxmox and should not be blocked unless they are not used:
	- Web interface: 8006 (TCP, HTTP/1.1 over TLS)
	- VNC Web console: 5900-5999 (TCP, WebSocket)
	- SPICE proxy: 3128 (TCP)
	- sshd (used for cluster actions): 22 (TCP)
	- rpcbind: 111 (UDP)
	- sendmail (SMTP): 25 (TCP, outgoing)
	- corosync cluster traffic: 5405-5412 UDP
	- live migration (VM memory and local-disk data): 60000-60050 (TCP)

## Use UFW (Uncomplicated Firewall) to lockdown access to the server
*NOTE: UFW stands for Uncomplicated Firewall and uses the built-in netfilter feature already in Linux. iptables is a suite of commands used to control netfilter, however it can be quite complicated. Thankfully, UFW simplifies the use of iptables commands.*
1. Deny all incoming traffic:
	 - By default, all traffic is denied if it does not match an "allow" rule.
	 - If there are no rules made, all incoming traffic is denied.
2. Allow ports for Proxmox to communicate:
	 - Proxmox uses ports 8006 (HTTPS) and 2020 (VNC) for Web GUI management, and therefore they must be allowed.
	 - The following two rules will allow the local client machine to access the Proxmox Web GUI to manage virtual machines:
		```bash
		sudo ufw allow from <local_client_machine_IP> proto tcp to <proxmox_machine_IP> port 8006 comment "Proxmox HTTPS Web GUI"
		```
		```bash
		sudo ufw allow from <local_client_machine_IP> proto tcp to <proxmox_machine_IP> port 2020 comment "Proxmox HTTPS VNC"
		```
3. Only allow SSH from specific local network IPs
	```bash
	sudo ufw allow from <local_client_machine_IP> proto tcp to <proxmox_machine_IP> port 22 comment "Allow SSH to Proxmox Server"
	```

### References
https://wiki.ubuntu.com/UncomplicatedFirewall
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-debian-11-243261243130246d443771547031794d72784e6b36656d4a326e49732e

## Use CrowdSec Host-based IPS
*NOTE: CrowdSec is an open-source host-based intrusion prevention system (HIPS) that gathers information from log sources on the host system, and will apply remediation based on that log information. CrowdSec is an open community and all threat intelligence gathered from host machines adds to different blacklists that can block known threats, usually based off IP addresses.*

### Installation
Installing CrowdSec happens in two parts:
1. The first is to download and install the security engine, which will be responsible for detection.
2. The second is to download and install the remediation components that will ultimately modify the firewall automatically to block suspected malicious connections.

### Remediation Components
Remediation Components are software packages in charge of acting upon decisions provided by the Security Engine.
- [nginx bouncer](https://doc.crowdsec.net/docs/next/bouncers/nginx) will check requester IP against the local API before granting or denying access.
- [firewall bouncer](https://doc.crowdsec.net/docs/next/bouncers/firewall) will add IPs to nftables/ipset set.
- [cloudflare bouncer](https://doc.crowdsec.net/docs/next/bouncers/cloudflare) will add IPs to the Cloudflare firewall.
- [blocklist mirror](https://doc.crowdsec.net/docs/next/bouncers/blocklist-mirror) will serve the blocklist to an appliance such as pfsense, fortinet, untangle via a http server.

Remediation Components interact with [crowdsec's Local API](https://doc.crowdsec.net/docs/next/local_api/intro) to retrieve active decisions and remediate appropriately.

### Collections, Parsers, and Scenarios 
*NOTE: A collection contains parsers and scenarios to form a coherent ensemble. Most of the time, this is the only thing you will need to install.*
*[Parsers](https://doc.crowdsec.net/docs/next/parsers/intro) are yaml files in `/etc/crowdsec/parsers/<STAGE>/parser.yaml`.*
- Use the following command to see what is installed: 
	```bash
	sudo cscli hub list
	```
- By default, SSH is really the only service that is being protected.
- What is really nice about CrowdSec is that it can look through other logs from other services running on the system and protect against certain scenarios such as brute forcing.
	- A great example for this would be the fact that Proxmox runs a Web GUI that is technically accessible to anyone on the same local area network.
	- If an attacker were to gain access, they can try to brute force their way in to the Proxmox server.
	- To monitor and block these attempts, download and install the Proxmox log parsers, scenarios, and collections: 
		```bash
		sudo cscli <configuration_type> install <item>
		```

### Protecting the Proxmox Server / Web GUI
1. Install the Proxmox collection: 
	```bash
	sudo cscli collections install fulljackz/proxmox
	```
2. Reload the CrowdSec config:
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

### Using the Web Interface Console for Management
*NOTE: The console is a web interface hosted by CrowdSec that allows admins to get more information, such as: tag and classify instances, view/filter/export alerts in real-time, get statistics and insights on alerts, organize management, MFA, and more.*

*In a nutshell, it simplifies management and monitoring of multiple servers.*
1. Adding a server is really simple. First log into CrowdSec (or create an account) https://www.crowdsec.net/
2. Click the "Add Security Engine button"
3. It will give a command to run on the server:
	```bash
	sudo cscli console enroll <some_unique_string>
	```
4. The unique string is there so CrowdSec knows that the server wants to join your CrowdSec account without requiring authentication on the server.
5. Back on the Web GUI, an "Enrollment Request" will appear, from which it can be accepted or denied. Make sure to accept the request from the server.
6. Restart the CrowdSec service:
	```bash
	sudo systemctl restart crowdsec.service
	```
7. Verify it is back up and running:
	```bash
	sudo systemctl status crowdsec.service
	```
8. Now on the CrowdSec Web GUI, the server's name can be changed and the server can be easily managed to perform tasks like: adding a blocklist of IPs, seeing alerts, managing scenarios, installing bouncers, etc.

### References
https://www.crowdsec.net/
https://doc.crowdsec.net/docs/next/getting_started/install_crowdsec
https://www.youtube.com/watch?v=2Ec-FYmK4zg

## Monitor Logs
*NOTE: Monitoring logs is probably one of the most crucial methods when securing a system. Logs will always tell the story of what happens to an application or what goes on in the network.*
`systemd-journald` is a system service that comes with systemd on modern Linux distros. It collects and stores logging data. In addition, it creates and maintains structured, indexed journals based on logging information received from various sources such as Linux Kernel log messages via `kmsg`. Therefore, the `journalctl` command will be used to query the contents of the `systemd-journald`.
1. View all boot messages with: 
	```bash
	sudo journalctl -b
	```
2. Display a live log display from a system service apache.service or nginx.service:
	```bash
	sudo journalctl -f -u apache
	```
	```bash
	sudo journalctl -f -u nginx
	```
3. Similar to the `tail -f /var/log/nginx/foo.log` command. View logs in real time:
	```bash
	sudo journalctl -u mysqld.service -f
	```
	```bash
	sudo journalctl -u nginx.service -f
	```
	```bash
	sudo journalctl -f
	```
4. Only display last 10 log entries:
	```bash
	sudo journalctl -n 10 -u nginx.service
	```
5. Reverse the output so the newest entries are displayed first with the `-r` switch:
	```bash
	sudo journalctl -r -u apache.service
	```
	```bash
	sudo journalctl -r
	```
6. See logs for specific User IDs (UID) or Process IDs (PID):
	```bash
	sudo journalctl _UID=300
	```
	```bash
	sudo journalctl _PID=4242
	```

### References
https://www.cyberciti.biz/faq/linux-log-files-location-and-how-do-i-view-logs-files/
https://www.cyberciti.biz/tips/linux-security.html