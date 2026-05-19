FROM lightpanda/browser as lightpanda
FROM nousresearch/hermes-agent as hermes
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
COPY --from=lightpanda /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=lightpanda /bin/lightpanda /usr/local/bin/lightpanda
COPY --from=hermes /opt/hermes /opt/hermes
COPY supervisord.conf /etc/supervisor/supervisord.conf

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]

