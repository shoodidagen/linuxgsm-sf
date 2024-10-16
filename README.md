
# Linux GSM, Satisfactory Docker Container (linuxgsm-sf)

I DON'T TAKE RESPONSIBILITY FOR ANY OF THE FOLLOWING CODE - SEE https://github.com/GameServerManagers/docker-gameserver FOR THE SUPPORTED VERSION OF THIS.

## About
LinuxGSM is a command-line tool for quick, simple deployment and management of Linux dedicated game servers. 

Satisfactory can be run on LinuxGSM. This is also easy via Docker containers.

The following assumes you have at least the following prereq
- Ubuntu 24.02
- Docker 27.2.0

## Install instructions

You need the following files in the following structure on your Linux Ubuntu OS. These files are available to download via this Git page.
```bash
/linuxgsm-sf/
│── Dockerfile
│── docker-compose.yml
└── linuxgsm-sf.env
```
### Dockerfile

The following was taken from https://github.com/GameServerManagers/docker-gameserver/blob/main/dockerfiles/Dockerfile.sf
Github should be checked for any changes/updates.

The following has had a lot of comments added to help understand what each command will do.

```bash
#
# LinuxGSM Satisfactory Dockerfile
#
# https://github.com/GameServerManagers/docker-gameserver
#

# This is the base image we're starting with. It comes from LinuxGSM, a tool for managing game servers.
FROM gameservermanagers/linuxgsm:ubuntu-22.04

# Adding a label to identify who maintains this Docker image.
LABEL maintainer="LinuxGSM <me@danielgibbs.co.uk>"

# Setting an argument (for later use), which defines a short name for the game server (Satisfactory, in this case).
ARG SHORTNAME=sf

# Setting an environment variable for the game server.
ENV GAMESERVER=sfserver

# The working directory for the application. Everything that follows will happen inside the /app directory.
WORKDIR /app

# This block installs the dependencies required for the game server automatically.
# It fetches the list of dependencies for Ubuntu 22.04 from a remote CSV file, looks for the ones related to "sf" (Satisfactory),
# and then installs them using apt-get. It also cleans up temporary files afterwards to reduce image size.
RUN depshortname=$(curl --connect-timeout 10 -s https://raw.githubusercontent.com/GameServerManagers/LinuxGSM/master/lgsm/data/ubuntu-22.04.csv | \
  awk -v shortname="sf" -F, '$1==shortname {$1=""; print $0}') \
  && if [ -n "${depshortname}" ]; then \
  echo "**** Install ${depshortname} ****" \
  && apt-get update \
  && apt-get install -y ${depshortname} \
  && apt-get -y autoremove \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*; \
  fi

# This sets up a health check for the container to monitor whether the game server is running properly. It runs a custom health check script every minute.
HEALTHCHECK --interval=1m --timeout=1m --start-period=2m --retries=1 CMD /app/entrypoint-healthcheck.sh || exit 1

# This command saves the current date and time to a file called /build-time.txt. It's useful to track when the image was built.
RUN date > /build-time.txt

# This defines the default command that runs when the container starts. It runs a script called entrypoint.sh.
ENTRYPOINT ["/bin/bash", "./entrypoint.sh"]

```

### docker-compose.yml

The following is currently using the Host network, which means you'll also need to allow the port 7777 on your Host firewall. Later on the 'Bridge' Network mode will be tested. In Bridge mode, you will not need to open the port on your host OS firewall as it will bypass the OS Firewall (UFW).

```bash
name: linuxgsm-sf
services:
  linuxgsm-sf:
    image: gameservermanagers/gameserver:sf
    container_name: linuxgsm-sf
    hostname: linuxgsm-sf
    network_mode: host
    volumes:
      - data:/data
    env_file:
      - linuxgsm-sf.env
    restart: unless-stopped
volumes:
  data:
```

- Service Name: Defines a service called linuxgsm-sf for the Satisfactory game server.
- Image: Uses the Docker image gameservermanagers/gameserver:sf for the game server.
- Container and Hostname: The container is named linuxgsm-sf, and the hostname inside the container is the same.
- Network Mode: network_mode: host allows the game server to interact directly with the host's network, which is often necessary for game servers to handle traffic properly.
- Volume: The data:/data volume ensures that game server data (like saved files) is stored persistently.
- Environment File: Loads environment variables from a file called linuxgsm-sf.env (this file should be created and contains variables to configure the game server).
- Restart Policy: The container will automatically restart unless you manually stop it, ensuring the game server stays online.
- Volume Definition: Declares a named volume called data for persistent storage.


### environment file

The environment file is just a best practice, this could also be put straight into the docker-compose.yml but this is a best practice.

The linuxgsm-sf.env file should contain the following single line. This can be added to in future.

```bash
TZ=Europe/London
```

## Building the image

We use the Dockerfile to create a local custom image just for our use. 

To build the image, navigate to the directory where your Dockerfile is located.
```bash
cd ~/linuxgsm-sf
```

In this example, the project folder is located in '/home/main/docker/~'. Adjust to suite.

Once ready, run the following command to build the image

```bash
docker build --no-cache -t gameservermanagers/gameserver:sf .
```

Breakdown:
- docker build: This command tells Docker to build an image using the Dockerfile in the current directory.
- --no-cache: Forces Docker to ignore any cached layers and build the image from scratch. This ensures that any updates to the base image or dependencies are included.
- -t gameservermanagers/gameserver:sf: Tags the built image with the name gameservermanagers/gameserver and gives it the tag sf. This helps you reference the image later when you want to run it.
- . (dot): Specifies that the Dockerfile is located in the current directory.

<example of the command being run>

![dockerComposeUp1](https://github.com/user-attachments/assets/cf5e3876-826f-443f-9939-692ca88c794e)


## Deploy and Run the container for the first time
To deploy the container from the custom image, navigate to the directory where your Dockerfile is located.
```bash
cd ~/linuxgsm-sf
```

Run the docker-compose up command to start the container in detached mode:
```bash
docker compose up -d
```
The above command will:
- Start a container based on the image gameservermanagers/gameserver:sf.
- Set up the environment from linuxgsm-sf.env.
- Mount the specified volume to /data.
- Use the host network mode (allowing the container to share the host's networking).
- Automatically restart the container unless it’s explicitly stopped (restart: unless-stopped).

<below is an example of this code being run>

![dockerComposeUp2](https://github.com/user-attachments/assets/067d209c-b78e-4fff-828a-623f8ca8c6f0)

You can run the following command to tail the logs of the container and see the console output. 
```bash
docker logs -f linuxgsm-sf
```

Note - it will look like it has frozen at this point but it has not
![image](https://github.com/user-attachments/assets/68368484-f472-481e-aef4-5ae16ebdcfaa)

It will eventually contiinue to download the game and configs for Satisfactory

![image](https://github.com/user-attachments/assets/87b32cbd-452a-4975-924d-4088e2447fde)

![image](https://github.com/user-attachments/assets/a0dad355-956d-4b2a-85d0-83b0531793af)


Eventually, the game has loaded and is waiting for initial connection.

![image](https://github.com/user-attachments/assets/5c62a3a2-2cc8-4438-beec-c3c8732d3b83)


## Firewall and Port Forwards

Both TCP and UDP of port 7777 must be opened and forwarded as appropriate.

If you are using the Uncomplicated Firewall (UFW) that comes shipped with Ubuntu, you can use the following line to open the firewall port required to allow peers to connect to the Satisfactory server.

```bash
sudo ufw allow 7777 comment 'Satisfactory Container'
```

## Setting up Admin Passwords and Game Settings

Open Satisfactory
![image](https://github.com/user-attachments/assets/dfe37ad5-e2db-4d5e-840f-b60154439983)

Click on 'Server Manager' and then select 'Add Server'.

![image](https://github.com/user-attachments/assets/86e52700-f6c7-40d7-922c-03eedab3dc05)

Enter the IP address (or DNS name) of the server and click 'confirm'.
![image](https://github.com/user-attachments/assets/cbb80aec-0f4b-4732-9143-eff056d3538f)

Accept the server Certificate

![image](https://github.com/user-attachments/assets/913a50f6-5bbd-4736-80e1-68b543447153)

Next, enter what you would like the server name to be. This is ususally what players will see when connecting.
![image](https://github.com/user-attachments/assets/3b02f5ef-1fc7-4f59-a553-517241f5c678)

Then Set an Admin password
![image](https://github.com/user-attachments/assets/93a7f1ec-2cf4-444d-be86-831cd7ab7e84)

Once the Admin password has been set, head over to 'Server Settings' shown below
![image](https://github.com/user-attachments/assets/d1c0a3c3-33d1-4fb2-8366-35450beaf274)

Set the Player Password (the password other players will need to use in order to successfully connect)

![image](https://github.com/user-attachments/assets/3c3f54fa-4337-423e-8101-f7cd9c6659a7)

and adjust any other settings here. Hit 'APPLY' when done.
When ready, head towards the 'Create Game' tab, shown below
![image](https://github.com/user-attachments/assets/2d6feaff-4890-4d25-b590-d40debc46f19)

Select a starting map and give it a name.
![image](https://github.com/user-attachments/assets/b4a62b22-d854-45aa-accd-ace7a39fa21b)
![image](https://github.com/user-attachments/assets/1340724c-44bd-4fca-910c-9baf6ff33d6f)

It will take a while for the server to be ready to join
![image](https://github.com/user-attachments/assets/8e8afa4e-903b-43e6-8552-662a77433e36)

![image](https://github.com/user-attachments/assets/3b4e5d3b-0ba0-4611-a4a2-4ad9e8d9b09c)


## Joining a friends Server
Open Satisfactory
![image](https://github.com/user-attachments/assets/dfe37ad5-e2db-4d5e-840f-b60154439983)

Click on 'Server Manager' and then select 'Add Server'.

![image](https://github.com/user-attachments/assets/86e52700-f6c7-40d7-922c-03eedab3dc05)

Enter the IP address (or DNS name) of the server and click 'confirm'.
![image](https://github.com/user-attachments/assets/cbb80aec-0f4b-4732-9143-eff056d3538f)

Accept the server Certificate

![image](https://github.com/user-attachments/assets/913a50f6-5bbd-4736-80e1-68b543447153)

## Updating the server

---

## Removing Container and Volumes

For server wipes you may wish to remove the container config and the data at the same time.
The commands shown below are used to stop and remove the resources created by a Docker Compose application, including the volumes associated with the services defined in your docker-compose.yml file

First make sure you are in the location where the existing docker-compose.yml sits. You may need to adjust this to suite. For example, if your project sits in ```/home/username/documents/linuxgsm-sf```, you would put ```cd /home/username/documents/linuxgsm-sf```.
```bash
cd ~/linuxgsm-sf
```

Then stop the container and remove the volumes.

```bash
docker compose down --volumes
```
![DockerComposeDown](https://github.com/user-attachments/assets/90be66e8-6019-41e7-bfdf-5621e7c298f3)

![DockerComposeDown2](https://github.com/user-attachments/assets/15a7d340-bb5c-4a95-b3fc-df84209e9c5b)

![DockerComposeDown3](https://github.com/user-attachments/assets/6a41c24a-11c1-4c80-8b69-d09b20e0c725)

![DockerComposeDown4](https://github.com/user-attachments/assets/3c1d7bcd-a50b-442a-a103-c0b93993424d)

![DockerComposeDown5](https://github.com/user-attachments/assets/7d27f92d-8b49-424c-a3fa-b3f139ef8691)

You might want to clean up the images left over, especially if you want a clean start or are not confident that the existing images are up to 'scratch'.
to do this, run the following command;

```bash
docker image prune
```
