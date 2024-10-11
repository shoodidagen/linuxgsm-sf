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
