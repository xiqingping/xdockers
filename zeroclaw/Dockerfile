FROM rust:slim-trixie AS builder
WORKDIR /app
ARG FEATURES="agent-runtime,gateway,acp-bridge,tui-onboarding,observability-prometheus,schema-export"

RUN apt-get update && apt-get install -y --no-install-recommends \
    git ca-certificates curl pkg-config && rm -rf /var/lib/apt/lists/*

RUN curl -fsSL https://deb.nodesource.com/setup_22.x | bash - \
    && apt-get install -y --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/*

RUN git clone --branch master --depth 1 \
    https://github.com/zeroclaw-labs/zeroclaw.git . && rm -rf .git \
    && cd web && npm ci --ignore-scripts \
    && cd /app && cargo build --release --locked --features "$FEATURES" \
    && cp target/release/zeroclaw /zeroclaw && strip /zeroclaw \
    && cargo run --release -p xtask --bin web -- gen-api \
    && cd web && npm run build

FROM python:3.14-slim-trixie
COPY --from=ghcr.io/astral-sh/uv:0.6.3 /uv /uvx /bin/
RUN apt-get update && apt-get install -y --no-install-recommends \
    bash ca-certificates curl git && rm -rf /var/lib/apt/lists/*

COPY --from=builder /zeroclaw /usr/local/bin/zeroclaw
COPY --from=builder /app/web/dist /usr/share/zeroclaw/web/dist

COPY --from=docker.io/lightpanda/browser /bin/lightpanda /usr/local/bin/lightpanda

RUN mkdir -p /zeroclaw-data/.zeroclaw /zeroclaw-data/workspace \
    && uv pip install --system supervisor \
    && printf '%s\n' \
    '[supervisord]' \
    'pidfile = /tmp/zeroclaw_supervisord.pid' \
    'logfile = /tmp/zeroclaw_supervisord.log' \
    'nodaemon=true' \
    'logfile_maxbytes=1024000' \
    'logfile_backups=3' \
    '' \
    '[include]' \
    'files = /zeroclaw-data/supervisor_*.conf' \
    > /etc/supervisord.conf

ENV LANG=C.UTF-8
ENV ZEROCLAW_WORKSPACE=/zeroclaw-data/workspace
ENV HOME=/zeroclaw-data
ENV ZEROCLAW_GATEWAY_PORT=42617
WORKDIR /zeroclaw-data
EXPOSE 42617
ENTRYPOINT ["supervisord"]
CMD ["-c", "/etc/supervisord.conf"]
