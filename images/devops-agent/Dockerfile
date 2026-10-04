FROM ubuntu:24.04

# for build azure agent download uri
ENV TARGETARCH="linux-x64"

ENV DEBIAN_FRONTEND=noninteractive
ENV ANDROID_HOME=/opt/android-sdk
ENV PATH="${PATH}:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator"

# Basic tools
RUN apt-get update && apt-get install -y \
    wget unzip curl git sudo kmod \
    libvirt-daemon-system libvirt-clients qemu-kvm \
    libglu1-mesa jq libicu74 zip \
    gpg apt-transport-https \
    openjdk-17-jdk \
    podman fuse-overlayfs podman-docker

# Install Azure CLI
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Install Android SDK command-line tools
RUN mkdir -p ${ANDROID_HOME}/cmdline-tools && \
    wget -q https://dl.google.com/android/repository/commandlinetools-linux-14742923_latest.zip -O cmdline-tools.zip && \
    unzip -q cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools/ && \
    mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest && \
    rm cmdline-tools.zip

# Accept licenses
RUN yes | sdkmanager --licenses

# Install required Android packages
RUN sdkmanager \
    "platform-tools" \
    "emulator" \
    "platforms;android-34" \
    "system-images;android-34;google_apis;x86_64"

# Create AVD
RUN echo "no" | avdmanager create avd -n ci_avd \
    -k "system-images;android-34;google_apis;x86_64" \
    --device "pixel"

# Install Maestro CLI
RUN curl -Ls "https://get.maestro.mobile.dev" | bash
ENV PATH="${PATH}:/root/.maestro/bin"

# Podman config
RUN mkdir -p /etc/containers /var/lib/containers/storage /run/podman && \
    echo '[storage]\ndriver = "vfs"\nrunroot = "/run/podman"\ngraphroot = "/var/lib/containers/storage"' > /etc/containers/storage.conf && \
    echo '[containers]\nisolation = "chroot"' > /etc/containers/containers.conf && \
    touch /etc/containers/nodocker

# Download the Microsoft repository GPG keys
RUN wget -q https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb \
    && dpkg -i packages-microsoft-prod.deb \
    && rm packages-microsoft-prod.deb

# Install PowerShell
RUN apt-get update && apt-get install -y powershell

WORKDIR /azp/
COPY ./start.sh ./
RUN chmod +x ./start.sh

ENV AGENT_ALLOW_RUNASROOT="true"

ENTRYPOINT ["./start.sh"]
