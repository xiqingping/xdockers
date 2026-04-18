FROM lightpanda/browser as builder

USER root
RUN apt-get update -yq && \
    apt-get install -yq xz-utils ca-certificates \
        make curl git golang-go \
        nodejs npm

# Get pnpm
RUN npm install -g pnpm

RUN git clone https://github.com/sipeed/picoclaw /picoclaw
WORKDIR /picoclaw
RUN make build
RUN make build-launcher

FROM debian:stable-slim

USER root

RUN mkdir -p /data \
    && apt-get update -yq \
    && apt-get install -yq supervisor htop curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN curl -LsSf https://astral.sh/uv/install.sh | /bin/bash

WORKDIR /data
VOLUME /data

# copy ca certificates
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /bin/lightpanda /usr/local/bin/lightpanda
COPY --from=builder /picoclaw/build/picoclaw /usr/local/bin/picoclaw
COPY --from=builder /picoclaw/build/picoclaw-launcher /usr/local/bin/picoclaw-launcher
COPY supervisord.conf /etc/supervisor/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]

