---
title: 1.7 - Creating the new psDooM Docker Image
layout: home
---

## What is a Docker container and image?

### Container
According to Docker's website:
> A **container** is a *sandboxed process* running on a host machine that is *isolated from all other processes* running on that host machine. That isolation leverages [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504), features that have been in Linux for a long time. Docker makes these capabilities approachable and easy to use.
To summarize, a container:
- Is a runnable instance of an *image*. You can create, start, stop, move, or delete a container using the Docker API or CLI.
- Can be run on local machines, virtual machines, or deployed to the cloud.
- Is *portable* (and can be run on any OS).
- Is *isolated* from other containers and runs its own software, binaries, configurations, etc.

### Image
According to Docker's website:
> A running container uses an isolated filesystem. This isolated filesystem is provided by an image, and the image *must contain everything needed to run an application* - all dependencies, configurations, scripts, binaries, etc. The image also contains other configurations for the container, such as environment variables, a default command to run, and other metadata.

### References
https://docs.docker.com/get-started/

## Creating a new Docker image for Kasm
*NOTE: Kasm offers advice on how to properly create containers that will run on its streaming platform with little to no hiccups. This is because Kasm generally uses its own version of `Ubuntu 20.0.4 (Focal Fossa)` operating system.*

*The modified Ubuntu OS is meant to be lightweight and comes preloaded with the necessary configurations and packages for Docker containers to operate and stream.*

### Build a Basic Container for Testing
*NOTE: Before going on to build a custom container image, it's best to test out a basic image and make it work on Kasm before doing anything more complicated.*
1. On the Debian 12 VM, Create a directory in the home folder called "Docker" and change into that directory.
	```bash
	mkdir ~/Docker && cd ~/Docker
	```
2. Create another directory called "Test" and change into that directory.
	```bash
	mkdir Test && cd Test
	```
	- The current working directory will be `~/Docker/Test`
3. Using a text editor, create a file named `Dockerfile`:
	```bash
	vim Dockerfile
	```
4. Copy and paste the following information into the file:
	```bash
	# Specify the kasm version of Ubuntu to pull
	FROM kasmweb/core-ubuntu-focal:1.14.0
	# Use the root user for the following tasks
	USER root
	
	# Create new variables to specify locations
	ENV HOME /home/kasm-default-profile
	ENV STARTUPDIR /dockerstartup
	ENV INST_SCRIPTS $STARTUPDIR/install
	# Make the working directory the home folder
	WORKDIR $HOME
	
	######### Customize Container Here ###########
	
	# Once the Kasm version of Ubuntu is loaded, run any code that needs to execute to create the environment.
	# A simple test of creating a "hello.txt" file on the desktop will do.
	RUN touch $HOME/Desktop/hello.txt
	
	######### End Customizations ###########
	
	# Change ownership of the home folder to user 1000 (kasm-user)
	RUN chown 1000:0 $HOME
	RUN $STARTUPDIR/set_user_permission.sh $HOME
	
	# Change the $HOME variable to user 1000 (kasm-user)
	ENV HOME /home/kasm-user
	# Make the working directory the home folder for user 1000 (kasm-user)
	WORKDIR $HOME
	# Make the new home directory and change ownership to user 1000 (kasm-user)
	RUN mkdir -p $HOME && chown -R 1000:0 $HOME
	
	# Change to user 1000 (kasm-user)
	USER 1000
	```
5. Save the file.
6. This file is a "blueprint" for Docker to build an image file. Build the new image:
	```bash
	sudo docker build -t dockerfile-test:latest -f Dockerfile .
	```
7. Verify the image has been created by listing all current Docker images:
	```bash
	sudo docker image ls
	```
8. Open Kasm Web GUI and login to the "Admin" account.
9. In the left pane, expand "Workspaces", click "Workspaces"
10. Click "Add Workspace"
11. Fill out all of the required information on the form. Use the following criteria:
	```
	Workspace Type: Container
	Friendly Name: Dockerfile Test
	Description: Test of our simple Dockerfile
	Enabled: Yes
	Docker Image: dockerfile-test:latest
	Cores: 2
	Memory: 2048
	GPU Count: 0
	CPU Allocation Method: Inherit
	```
12. Save the changes.
13. At the top of the web page, click "Workspaces"
14. The "Dockerfile Test" workspace should be there.
15. Select it and then launch the workspace in the current tab.
16. Verify that a desktop environment loads with a text file called "hello.txt"
17. Once verified, delete the session from the control panel on the left side of the window.

### Creating a Custom Container

#### Choosing an application to run
Knowing that the basic container image built and ran, a custom container can now be designed. However, any other application can be "containerized" for this purpose.

A popular application and game known as "DooM" would be a perfect and fun experiment to turn into a container for Docker to run. However, Kasm has already done this with a freeware version of DooM, but some custom versions of DooM exist that would be even more exciting to try out in a container. Namely, "psDooM" that is a modification for DooM that ties enemies into actually running processes.

It is an ingenious idea but getting it to function in a Kasm specific Docker container will take some elbow grease.

Also, there are many people associated with the development of psDooM and the platform it uses called "Chocolate Doom". To find out more information about the game, here is a link for reading: https://github.com/orsonteodoro/psdoom-ng/tree/master

#### Creating a Custom Docker Image for psDooM
1. On the Debian 12 VM, In the `~/Docker` directory that was created earlier, create another folder called `psdoom-kasm` and change into that directory:
	```bash
	mkdir ~/Docker/psdoom-kasm && cd ~/Docker/psdoom-kasm
	```
2. Using a text editor, create a file called `psdoom-kasm-image`:
	```bash
	vim psdoom-kasm-image
	```
3. This will be the Dockerfile that will be used to build the image. Copy and paste the following information into the file:
	```bash
	# Use the Kasm modified Ubuntu image as a base.
	FROM kasmweb/core-ubuntu-focal:1.14.0
	
	# Do all of the following as the root user.
	USER root
	
	# Create environment variables.
	ENV HOME /home/kasm-default-profile
	ENV STARTUPDIR /dockerstartup
	ENV INST_SCRIPTS $STARTUPDIR/install
	ENV PSDOOM_LOG /tmp/psdoom_install_log.txt
	ENV DOOM_WAD /usr/share/games/doom
	
	# The working directory will be the home folder.
	WORKDIR $HOME
	
	######### Customize Container Here ###########
	
	# Create the PSDOOM Log file
	RUN touch $PSDOOM_LOG
	
	# Clone the working version of psDooM 1.6.0
	RUN git clone --depth 1 --branch 1.6.0 https://github.com/orsonteodoro/psdoom-ng.git \
		&& echo "GIT CLONE SUCCESSFUL" >> $PSDOOM_LOG
	
	# Install dependencies and developer tools.
	RUN apt update \
		&& apt install -y make \
		&& apt install -y gcc \
		&& apt install -y libsdl-mixer1.2 \
		&& apt install -y libsdl-mixer1.2-dev \
		&& apt install -y libsdl-net1.2 \
		&& apt install -y libsdl-net1.2-dev \
		&& apt install -y libsdl1.2debian \
		&& apt install -y libsdl1.2-dev \
		&& apt install -y libsdl-sound1.2 \
		&& apt install -y libsdl-sound1.2-dev \
		&& echo "DEPENDENCIES INSTALLED SUCCESSFULLY" >> $PSDOOM_LOG \
		&& apt install -y autotools-dev \
		&& apt install -y autoconf \
		&& echo "TOOLS INSTALLED SUCCESSFULLY" >> $PSDOOM_LOG
	
	# Change working directory to the cloned directory and install psDooM. After install, create directory for WAD files.
	RUN cd ~/psdoom-ng/trunk \
	        && ./configure \
	        && make \
	        && make install \
			&& mkdir -p $DOOM_WAD \
	        && echo "DOOM INSTALLED SUCCESSFULLY" >> $PSDOOM_LOG
	
	# Copy the IWAD and Custom map WAD files from the local machine to the container
	COPY wad/doom2.wad $DOOM_WAD
	COPY wad/psdoom2.wad $DOOM_WAD
	
	# Copy the custom startup script for the game to launch properly
	COPY scripts/custom_startup.sh $STARTUPDIR/custom_startup.sh
	RUN chmod 755 $STARTUPDIR/custom_startup.sh
	
	# Update the desktop environment to be optimized for a single application
	RUN cp $HOME/.config/xfce4/xfconf/single-application-xfce-perchannel-xml/* $HOME/.config/xfce4/xfconf/xfce-perchannel-xml/ && cp /usr/share/extra/backgrounds/bg_kasm.png /usr/share/extra/backgrounds/bg_default.png && apt-get remove -y xfce4-panel
	
	######### End Customizations ###########
	```
4. The container will not be built with Docker just yet because there are a few prerequisite tasks that must be completed.

#### Game Files

##### Acquire a copy of Doom 2
In order to run the psDooM game, there must be a legitimate copy of DooM 2 present on the system, otherwise obtaining the game by copying files from someplace else could result in prosecution by the law as the original DooM games are copyrighted material.

The game can be purchased in many ways, but the easiest way is to buy a copy from Steam: https://store.steampowered.com/app/2300/DOOM_II/

*NOTE: The game may need to be installed on a Windows client.*

##### Retrieve WAD Files
DooM games use a "WAD" file that contains the necessary data to run the game. Ultimately, "psDooM" is just a modification of DooM, so that's why the WAD files are needed.
1. Once DooM 2 is installed on the local Windows client computer, go into the directory where the game is installed.
2. Look for a file name called `doom2.wad` and copy it somewhere where it can be found easily.
3. Download this file from the "psDooM" Git repo: https://github.com/orsonteodoro/psdoom-ng/blob/8a27458baf3d098df7436281b5a2ed80f9007ccf/extras/psdoom-2000.05.03-data.tar.gz
	- This file contains a custom map that will be needed for the psDooM mod.
4. Once downloaded, extract the tarball and find the `psdoom2.wad` file, and save it to the same place where the other `doom2.wad` file is located.
5. Both the `doom2.wad` and `psdoom2.wad` files will be used for the next steps.

##### Copy the WAD files
1. On the Debian 12 VM, In the same directory as the `psdoom-kasm-image` Dockerfile that was create earlier, create another directory called "wad":
	```bash
	mkdir ~/Docker/wad/
	```
2. Using any method, copy both the `doom2.wad` and `psdoom2.wad` files from the local Windows client machine into the `wad/` directory on the Debian 12 VM.
	- One method could be to use secure copy or `scp` to copy them to the Proxmox server, and then to the Debian 12 VM.
	- Another method could be uploading them to a website, and then using `curl` to download them onto the Debian 12 VM.

#### Create a Kasm Custom Startup Script
*NOTE: Every image that Kasm uses to start containers can contain a `custom_startup.sh` script that will be automatically called when the container starts up. This script is mainly used to clean up the desktop environment, start up the desired program, and log anything that is necessary.*
1. On the Debian 12 VM, in the same directory as the `psdoom-kasm-image` Dockerfile, create another directory called "scripts" and change into that directory:
	```bash
	mkdir ~/Docker/scripts && cd ~/Docker/scripts
	```
2. Create a custom bash script called `custom_startup.sh`:
	```bash
	vim custom_startup.sh
	```
3. Copy and paste the following information into the file:
	```bash
	#!/usr/bin/env bash
	
	# Create a log file to monitor the execution of this script.
	$CUSTOMLOG = "/tmp/custom_startup.log"
	touch $CUSTOMLOG
	
	# Run the desktop_ready Kasm script to before executing main program.
	/usr/bin/desktop_ready
	echo "desktop_ready command ran" >> $CUSTOMLOG
	
	# Assign the start command for psDooM to the "START_COMMAND" variable.
	START_COMMAND="/home/kasm-default-profile/psdoom-ng/trunk/src/psdoom -iwad /usr/share/games/doom/doom2.wad -file /usr/share/games/doom/psdoom2.wad -psuser root -nomonsters -window -geometry 1600x1200"
	echo "START_COMMAND variable assigned" >> $CUSTOMLOG
	
	# Start psDooM.
	$START_COMMAND &
	echo "Starting psDooM with START_COMMAND variable" >> $CUSTOMLOG
	
	# Maximize the psDooM (Chocolate Doom) Window (NOTE: It will only run at a specific resolution until fixed).
	# echo "wmctrl executing..." >> $CUSTOMLOG
	# wmctrl -r "DOOM 2: Hell on Earth - psdoom 2012.02.05-1.6.0" -b add,maximized_vert,maximized_horz -v &> /tmp/custom_startup.log
	# echo "wmctrl executed." >> $CUSTOMLOG
	```
4. Save the file.
5. Built the image with Docker:
	```bash
	sudo docker build -t psdoom-kasm:1.0 -f psdoom-kasm-image .
	```
5. Verify the image has been created by listing all current Docker images:
	```bash
	sudo docker image ls
	```
8. Open Kasm Web GUI and login to the "Admin" account.
9. In the left pane, expand "Workspaces", click "Workspaces"
10. Click "Add Workspace"
11. Fill out all of the required information on the form. Use the following criteria:
	```
	Workspace Type: Container
	Friendly Name: psDooM
	Description: psDooM Version 1.6.0
	Enabled: Yes
	Docker Image: psdoom-kasm:1.0
	Cores: 2
	Memory: 4096
	GPU Count: 0
	CPU Allocation Method: Inherit
	Docker Run Config Override (JSON): {"user":"root"}
	```
12. Save the changes.
13. At the top of the web page, click "Workspaces"
14. The "Dockerfile Test" workspace should be there.
15. Select it and then launch the workspace in the current tab.
16. Verify that psDooM loads up.
17. Once verified, delete the session from the control panel on the left side of the window.

### References
https://kasmweb.com/docs/latest/how_to/building_images.html
https://devopscube.com/build-docker-image/
https://github.com/kasmtech/workspaces-images/blob/develop/src/ubuntu/install/doom/install_doom.sh
https://github.com/kasmtech/workspaces-images/blob/develop/dockerfile-kasm-doom

## Upload the image to Docker Hub
*NOTE: Storing the image locally is nice, but what if something ever happened to the storage on the server? It is a good idea to back up the image to a cloud service such as Docker Hub. Create a Docker Hub account by visiting:* https://hub.docker.com/

*Docker Hub is a remote repository for Docker images, similar to how GitHub is a remote repository for code.*

*Keep in mind that copyrighted game files are now within the Docker image, and it is important to not release the Docker image to the public. To do this, a private Docker repository must be made where the Docker image is uploaded to.*

### Create a private repository
*NOTE: Use the local Windows client machine to perform the following steps.*
1. Login to Docker Hub (create an account if necessary).
2. Click "Create Repository"
3. Make the repository name "psdoom-kasm"
4. In the description, enter "psDooM version 1.6.0 for Kasm"
5. Change the visibility to "Private"
6. Click "Create"

### Create an Access Token
*NOTE: An access token is like a password, but temporary. It is used to authenticate into a Docker Hub account, but will be deleted after a certain amount of time or uses.*
1. On the Docker Hub website, go to "Account Settings"
2. Navigate to "Security"
3. Create a "New Access Token"
4. Access token description can be "psdoom"
5. Make sure it has "Read, Write, Delete" permissions.
6. Copy the access token to a safe place (i.e. an encrypted file on the local desktop computer).

### Log into Docker on through the Debian 12 command line
1. On the Debian 12 VM command line, log into Docker Hub:
	```bash
	docker login -u <username>
	```
2. Enter the token as the password.
3. It should say "Login Succeeded".
4. Never share the token and always make sure to disable it or delete it on Docker Hub once it is no longer needed.

### Pushing the Docker image
1. Tag the image for the private repository:
	```bash
	sudo docker tag psdoom-kasm:1.0 <username>/psdoom-kasm:1.0
	```
2. Push the image to the private repository:
	```bash
	sudo docker push <username>/psdoom-kasm:1.0
	```

## Configure Kasm to use Docker Hub image
*NOTE: Kasm can either use local storage, Docker Hub storage, or even third-party registry's for image storage.*
1. Open up the Kasm Web GUI and login as "Admin".
2. At the top of the web page, click "Admin"
3. On the left pane, expand "Workspaces", then click "Workspaces"
4. Click "Add Workspace"
5. Fill out the form with the following criteria:
```
	Workspace Type: Container
	Friendly Name: psDooM
	Description: psDooM Version 1.6.0
	Enabled: Yes
	Docker Image: <DockerHub_username>/psdoom-kasm:1.0
	Cores: 2
	Memory: 4096
	GPU Count: 0
	CPU Allocation Method: Inherit
	Docker Registry: https://index.docker.io/v1/
	Docker Registry Username: <DockerHub_username>
	Docker Registry Password: <DockerHub_token>
	Docker Run Config Override (JSON): {"user":"root"}
```
5. Save the changes.
6. The image will take time to download. 
7. View the installation progress by navigating to "Registry" and then selecting "Installed Workspaces".
8. This process could take up to 20 minutes (depending on internet connection and storage speed).
9. Once it is installed, navigate to "Workspaces" at the top of the screen.
10. Launch the "psDooM" workspace in the current tab.
11. Verify it works.
12. Once verified, delete the session from the control panel on the left side of the window.