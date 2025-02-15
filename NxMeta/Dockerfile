# https://support.networkoptix.com/hc/en-us/articles/205313168-Nx-Witness-Operating-System-Support
# Latest supported is Focal
FROM ubuntu:focal


# Labels
ARG LABEL_NAME="NxMeta"
ARG LABEL_DESCRIPTION="Nx Meta VMS Docker"

# Download URL and version
# Current values are defined by the build pipeline
ARG DOWNLOAD_URL="https://updates.networkoptix.com/metavms/35269/metavms-server_update-5.0.0.35269-linux_x64.zip"
ARG DOWNLOAD_VERSION="5.0.0.35269"

# NxWitness (networkoptix) or DWSpectrum (digitalwatchdog) or NxMeta (networkoptix-metavms)
ARG RUNTIME_NAME="networkoptix-metavms"

# The RUN wget command will be cached unless we change the cache tag
# Use the download version for the cache tag
ARG CACHE_DATE=${DOWNLOAD_VERSION}

# Prevent EULA and confirmation prompts in installers
ARG DEBIAN_FRONTEND=noninteractive

# Media server user and directory name
ENV COMPANY_NAME=${RUNTIME_NAME}

# Labels
LABEL name=${LABEL_NAME} \
    description=${LABEL_DESCRIPTION} \
    version=${DOWNLOAD_VERSION} \
    download=${DOWNLOAD_URL} \
    maintainer="Pieter Viljoen <ptr727@users.noreply.github.com>"

# Install required tools and utilities
RUN apt-get update \
    && apt-get upgrade --yes \
    && apt-get install --no-install-recommends --yes \
        ca-certificates \
        mc \
        nano \
        unzip \
        wget

# Download the installer file
RUN mkdir -p /temp
COPY Download.sh /temp/Download.sh
# Set the working directory to /temp
WORKDIR /temp
RUN chmod +x Download.sh \
    && ./Download.sh

# Install the mediaserver and dependencies
RUN apt-get update \
    && apt-get install --no-install-recommends --yes \
        gdb \
        sudo \
        ./vms_server.deb \
# Cleanup        
    && apt-get clean \
    && apt-get autoremove --purge \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /temp

# Add the mediaserver ${COMPANY_NAME} user to the sudoers group
# Only allow sudo no password access to the root-tool
# Add v4.2 and v5.0 flavors of the root tool naming
RUN echo "${COMPANY_NAME} ALL = NOPASSWD: /opt/${COMPANY_NAME}/mediaserver/bin/root-tool, /opt/${COMPANY_NAME}/mediaserver/bin/root-tool-bin" > /etc/sudoers.d/${COMPANY_NAME}

# Copy the entrypoint.sh launch script
# entrypoint.sh will run the mediaserver and root-tool
COPY root/entrypoint.sh /opt/entrypoint.sh
RUN chmod +x /opt/entrypoint.sh

# Run the entrypoint as the mediaserver ${COMPANY_NAME} user
# Note that this user exists in the container and does not directly map to a user on the host
USER ${COMPANY_NAME}

# Runs entrypoint.sh on container start
ENTRYPOINT ["/opt/entrypoint.sh"]

# Expose port 7001
EXPOSE 7001

# Mount points
# /opt/${COMPANY_NAME}/mediaserver/etc -> /config
# /opt/${COMPANY_NAME}/mediaserver/var -> /media

