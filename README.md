
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

### environment file

The environment file is just a best practice, this could also be put straight into the docker-compose.yml but this is a best practice.

The linuxgsm-sf.env file should contain the following single line. This can be added to in future.

```bash
TZ=Europe/London
```

## Building the image

We use the Dockerfile to create a local custom image just for our use. 

## Running the container for the first time

---

## Updating the server

---

## 
