---
title: 1.5 - Docker Installation and Configuration
layout: home
---

## Installing Docker
1. It is absolutely necessary to make sure that no Docker components have already been installed on this OS:
	```bash
	sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
	```
2. Add the official Docker repository to get the latest Docker version (as Debian repositories have an older version of Docker):
	1. Install Pre-requisites:
		```bash
		sudo apt update && sudo apt install ca-certificates curl gnupg
		```
	2. Create a directory to store GPG keys:
		```bash
		sudo install -m 0755 -d /etc/apt/keyrings
		```
	3. Download the Docker GPG key and store it in the new directory:
		```bash
		curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
		```
	4. Change the permissions of the GPG key:
		```bash
		sudo chmod a+r /etc/apt/keyrings/docker.gpg
		```
	5. Add the official Docker repository:
		```bash
		echo \
		  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
		  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \ tee /etc/apt/sources.list.d/docker.list > /dev/null
		```
	6. Update from the Docker repo and install Docker:
		```bash
		sudo apt update && sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
		```
	7. Test the Docker installation with a "hello world" container:
		```bash
		sudo docker run hello-world
		```
	8. If the "hello world" container ran successfully, the following output will be displayed:
		```
		Hello from Docker!
		This message shows that your installation appears to be working correctly.
		```

## Create a Docker group and Add a user
*NOTE: By default, Docker requires root permissions to manage containers. Therefore, a user must use `sudo` with every Docker command. This can get annoying, so adding the user to the "docker" group will mitigate this.*
1. Create a docker group:
	```
	sudo groupadd docker
	```
2. Add the user to the group:
	```bash
	sudo usermod -aG docker <non_root_username>
	```
3. Log out and log back in again as the non-root user.
4. Test with "hello world":
	```bash
	docker run hello-world
	```

### References
https://itsfoss.com/debian-install-docker/
https://www.simplilearn.com/tutorials/docker-tutorial/how-to-install-docker-on-ubuntu
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

## Docker Commands
https://docs.docker.com/engine/reference/commandline
https://docs.linuxserver.io/general/docker-compose
