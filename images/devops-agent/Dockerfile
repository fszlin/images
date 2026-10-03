ARG OPENCODE_IMAGE=ghcr.io/fszlin/opencode:latest
FROM ${OPENCODE_IMAGE}

ARG KUBECTL_VERSION=v1.31.0

USER root
RUN apt-get update \
    && apt-get install -y --no-install-recommends jq unzip openssh-client \
    && rm -rf /var/lib/apt/lists/* \
    && ARCH="$(dpkg --print-architecture)" \
    && curl -fsSLo /usr/local/bin/kubectl "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/${ARCH}/kubectl" \
    && chmod +x /usr/local/bin/kubectl

USER node
