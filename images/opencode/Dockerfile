FROM debian:latest

# Automatically set by Docker BuildKit (amd64 or arm64)
ARG TARGETARCH

USER root

# Setup GitHub CLI, Podman, passt, slirp4netns, aardvark-dns, and nftables, then install system utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    sudo \
    git \
    curl \
    ca-certificates \
    gnupg \
    python3 \
    python3-pip \
    libatomic1 \
    libicu-dev \
    default-jdk-headless \
    unzip \
    libsecret-1-0 \
    lsof \
    procps \
    net-tools \
    ripgrep \
    jq \
    fd-find \
    tree \
    yq \
    httpie \
    podman \
    podman-compose \
    passt \
    slirp4netns \
    aardvark-dns \
    fuse-overlayfs \
    uidmap \
    nftables \
    xclip \
    xsel \
    libgl1 \
    libglu1-mesa \
    libegl1 \
    libnss3 \
    libpulse0 \
    # Setup GitHub CLI keyring and repository with dynamic architecture lookup
    && mkdir -p -m 755 /etc/apt/keyrings \
    && curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
    && chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=${TARGETARCH} signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    # Update and install gh
    && apt-get update \
    && apt-get install -y --no-install-recommends gh \
    && rm -rf /var/lib/apt/lists/* \
    && ln -s /usr/bin/fdfind /usr/bin/fd

# Set environment variable to enable full ICU globalization support
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=0

# Update .NET version argument to 10.0
ARG DOTNET_VERSION=10.0

# Install .NET SDK targeting build architecture dynamically (amd64 vs arm64)
RUN curl -sSL https://dot.net/v1/dotnet-install.sh -o dotnet-install.sh \
    && chmod +x dotnet-install.sh \
    && ./dotnet-install.sh --install-dir /usr/share/dotnet --channel ${DOTNET_VERSION} --architecture ${TARGETARCH} \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet \
    && rm dotnet-install.sh

# Install PowerShell in a shared directory accessible to dev
ENV DOTNET_ROOT=/usr/share/dotnet
RUN dotnet tool install --tool-path /usr/share/powershell PowerShell \
    && chmod -R a+rX /usr/share/powershell \
    && ln -s /usr/share/powershell/pwsh /usr/bin/pwsh \
    && rm -rf /root/.nuget /root/.templateengine /root/.dotnet /tmp/*

# Install Node.js 22 & npm (NodeSource detects architecture automatically)
RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

# Install pnpm, Bun, OpenCode V2 CLI, and Playwright CLI globally
# (UPDATED: changed 'opencode-ai' to '@opencode/cli' for V2)
RUN npm install -g pnpm bun @opencode/cli playwright \
    && npm cache clean --force

# Install system dependencies for Playwright browsers
RUN npx playwright install-deps chromium \
    && rm -rf /var/lib/apt/lists/*

# Pre-download Chromium browser binary into the image (Playwright selects native arm64/amd64)
ENV PLAYWRIGHT_BROWSERS_PATH=/opt/ms-playwright
RUN npx playwright install chromium \
    && chmod -R a+rX /opt/ms-playwright

# Create repos directory, user 1000 with sudo privileges, and Android SDK directory structure
ARG USER_ID=1000
ARG GROUP_ID=1000
RUN mkdir -p /repos /home/dev/android-sdk && \
    groupadd -g ${GROUP_ID} dev && \
    useradd -l -u ${USER_ID} -g dev -m -s /bin/bash dev && \
    echo "dev ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers && \
    chown -R dev:dev /repos /home/dev/android-sdk

# Configure subuid and subgid for rootless podman inside the container for user dev (1000)
RUN echo "dev:100000:65536" > /etc/subuid && \
    echo "dev:100000:65536" > /etc/subgid

# Set Corepack home, Android SDK, Git global config, and Podman execution environment variables
ENV COREPACK_HOME=/home/dev/.cache/corepack
ENV ANDROID_HOME=/home/dev/android-sdk
ENV PATH="${PATH}:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools"
ENV GIT_CONFIG_GLOBAL=/home/dev/.config/git/.gitconfig
ENV BUILDAH_ISOLATION=chroot
ENV CONTAINERS_CGROUP_MANAGER=cgroupfs

# Download, extract, and install Android Command-line Tools (Java-based, cross-platform)
RUN curl -sSL https://dl.google.com/android/repository/commandlinetools-linux-15859902_latest.zip -o /tmp/cmdline-tools.zip \
    && mkdir -p ${ANDROID_HOME}/cmdline-tools \
    && unzip -q /tmp/cmdline-tools.zip -d /tmp/android-cmdline \
    && rm /tmp/cmdline-tools.zip \
    && mv /tmp/android-cmdline/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest \
    && rm -rf /tmp/android-cmdline \
    && chown -R dev:dev ${ANDROID_HOME}

# Pre-create OpenCode v2 directory paths, container configs, and storage directories
RUN mkdir -p /home/dev/.config/opencode \
             /home/dev/.local/share/opencode/logs \
             /home/dev/.local/share/opencode/snapshots \
             /home/dev/.local/share/opencode/media \
             /home/dev/.config/git \
             /home/dev/.config/gh \
             /home/dev/.local/share/containers \
             /home/dev/.config/containers \
    # Pre-configure Podman engine defaults: cgroupfs manager with disabled container cgroups
    && printf '[engine]\ncgroup_manager = "cgroupfs"\nevents_logger = "file"\n[containers]\nnetns = "pasta"\ncgroups = "disabled"\n' > /home/dev/.config/containers/containers.conf \
    && printf '[storage]\ndriver = "overlay"\n[storage.options.overlay]\nmount_program = "/usr/bin/fuse-overlayfs"\n' > /home/dev/.config/containers/storage.conf \
    && chown -R dev:dev /home/dev

# Set up docker executable symlink and shell alias to route docker commands to podman
RUN ln -s /usr/bin/podman /usr/local/bin/docker && \
    echo "alias docker=podman" >> /home/dev/.bashrc

# Switch to user 1000 so all file creations map to host 1000
USER dev

# Accept Android SDK licenses and install platform tools and build tools
RUN yes | sdkmanager --sdk_root=${ANDROID_HOME} --licenses \
    && sdkmanager --sdk_root=${ANDROID_HOME} "platform-tools" "platforms;android-36" "build-tools;36.0.0" "ndk;27.1.12297006"

# Install Android emulator and system image, create AVD "dc"
# (Google publishes no Linux arm64 emulator, so this is skipped on arm64)
RUN if [ "${TARGETARCH}" = "amd64" ]; then \
        sdkmanager --sdk_root=${ANDROID_HOME} "emulator" "system-images;android-36;google_apis;x86_64" \
        && echo "no" | avdmanager create avd -n dc -k "system-images;android-36;google_apis;x86_64" --device "pixel"; \
    fi
ENV PATH="${PATH}:${ANDROID_HOME}/emulator"

# Create base git config files pointing to the custom credentials file
RUN printf "[credential]\n\thelper = store --file=/home/dev/.config/git/.git-credentials\n" > /home/dev/.config/git/.gitconfig && \
    touch /home/dev/.config/git/.git-credentials

WORKDIR /repos

# Expose the server port and set default startup command for OpenCode v2 daemon
EXPOSE 4096
CMD ["opencode", "serve", "--hostname", "0.0.0.0", "--port", "4096"]
