FROM node:22-slim

ARG OPENCODE_VERSION=latest

RUN apt-get update \
    && apt-get install -y --no-install-recommends git ca-certificates curl ripgrep \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g "opencode-ai@${OPENCODE_VERSION}" \
    && npm cache clean --force

USER node
WORKDIR /workspace
ENTRYPOINT ["opencode"]
