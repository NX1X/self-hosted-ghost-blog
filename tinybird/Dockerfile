FROM python:3.14-slim@sha256:cea0e6040540fb2b965b6e7fb5ffa00871e632eef63719f0ea54bca189ce14a6

# Install system dependencies, create user, and set up scripts as root.
# apt versions aren't pinned (DL3008): Debian slim moves them frequently and
# pinning breaks rebuilds; the base image is digest-pinned instead.
# hadolint ignore=DL3008
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    jq \
    && rm -rf /var/lib/apt/lists/* \
    && groupadd -r tinybird \
    && useradd -r -g tinybird -d /home/tinybird -s /bin/bash tinybird \
    && mkdir -p /home/tinybird \
    && chown -R tinybird:tinybird /home/tinybird

# Copy scripts as root and set permissions
COPY handleLogin.sh /usr/local/bin/tinybird-login
COPY getTokens.sh /usr/local/bin/get-tokens
COPY tb-wrapper /usr/local/bin/tb-wrapper
RUN chmod +x /usr/local/bin/tinybird-login /usr/local/bin/get-tokens /usr/local/bin/tb-wrapper

# Switch to non-root user
USER tinybird
WORKDIR /home/tinybird

# Install the Tinybird CLI as the non-root user.
# Download the installer to a file first: a bare `curl ... | sh` silently runs
# nothing (exit 0) if the download fails or is truncated, whereas this fails the
# build on any HTTP error (-f) or empty body (test -s).
# NOTE: the upstream installer is not checksum-pinned. To fully control it,
# vendor a pinned copy and verify it before running, e.g.:
#   echo "<sha256>  tb-install.sh" | sha256sum -c -
RUN curl -fsSL https://tinybird.co -o /tmp/tb-install.sh \
    && test -s /tmp/tb-install.sh \
    && sh /tmp/tb-install.sh \
    && rm -f /tmp/tb-install.sh

ENV PATH="/home/tinybird/.local/bin:$PATH"

# Sanity check that the Tinybird CLI is installed and on PATH.
HEALTHCHECK CMD tb --version || exit 1
