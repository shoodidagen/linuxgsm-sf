
# Linux GSM, Satisfactory Docker Container (linuxgsm-sf)

I DON'T TAKE RESPONSIBILITY FOR ANY OF THE FOLLOWING CODE - SEE https://github.com/GameServerManagers/docker-gameserver FOR THE SUPPORTED VERSION OF THIS.

## About
LinuxGSM is a command-line tool for quick, simple deployment and management of Linux dedicated game servers. 

Satisfactory can be run on LinuxGSM. This is also easy via Docker containers.

## Install instructions

You need the following files in the following structure on your Linux Ubuntu OS.

/linuxgsm-sf/
│── Dockerfile
│── docker-compose.yml
└── linuxgsm-sf.env

### Dockerfile

The following was taken from https://github.com/GameServerManagers/docker-gameserver/blob/main/dockerfiles/Dockerfile.sf
Github should be checked for any changes/updates.

```bash
#
# LinuxGSM Satisfactory Dockerfile
#
# https://github.com/GameServerManagers/docker-gameserver
#
FROM gameservermanagers/linuxgsm:ubuntu-22.04
LABEL maintainer="LinuxGSM <me@danielgibbs.co.uk>"
ARG SHORTNAME=sf
ENV GAMESERVER=sfserver
WORKDIR /app
## Auto install game server requirements
RUN depshortname=$(curl --connect-timeout 10 -s https://raw.githubusercontent.com/GameServerManagers/LinuxGSM/master/lgsm/data/ubuntu-22.04.csv |awk -v shortname="sf" -F, '$1==shortname {$1=""; print $0}') \
  && if [ -n "${depshortname}" ]; then \
  echo "**** Install ${depshortname} ****" \
  && apt-get update \
  && apt-get install -y ${depshortname} \
  && apt-get -y autoremove \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*; \
  fi
HEALTHCHECK --interval=1m --timeout=1m --start-period=2m --retries=1 CMD /app/entrypoint-healthcheck.sh || exit 1
RUN date > /build-time.txt
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


## Running the container for the first time

---

## Updating the server

---

## 
