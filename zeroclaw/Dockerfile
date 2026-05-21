FROM rust:alpine AS builder
WORKDIR /app
ARG FEATURES="agent-runtime,gateway,acp-bridge,tui-onboarding,observability-prometheus,schema-export"
RUN apk add --no-cache git nodejs npm
RUN git clone --branch codex/issue-6201-wecom-ws --depth 1 \
    https://github.com/EyrieCommander/zeroclaw.git . && rm -rf .git
RUN cd web && npm ci --ignore-scripts
RUN cargo build --release --locked --features "$FEATURES" \
    && cp target/release/zeroclaw /zeroclaw && strip /zeroclaw
RUN cargo run --release -p xtask --bin web -- gen-api
RUN cd web && npm run build

FROM alpine:3.21
RUN apk add --no-cache bash ca-certificates curl git
COPY --from=builder /zeroclaw /usr/local/bin/zeroclaw
COPY --from=builder /app/web/dist /usr/share/zeroclaw/web/dist
RUN mkdir -p /zeroclaw-data/.zeroclaw /zeroclaw-data/workspace && \
    printf '%s\n' \
        'workspace_dir = "/zeroclaw-data/workspace"' \
        'default_provider = "openai"' \
        'default_model = "gpt-4o"' \
        '' \
        '[gateway]' \
        'port = 42617' \
        'host = "[::]"' \
        'allow_public_bind = true' \
        'web_dist_dir = "/usr/share/zeroclaw/web/dist"' \
        '' \
        '[risk_profiles.default]' \
        'level = "supervised"' \
        'auto_approve = ["file_read", "file_write", "file_edit", "memory_recall", "memory_store", "web_search_tool", "web_fetch", "calculator", "glob_search", "content_search", "image_info", "weather", "git_operations"]' \
        > /zeroclaw-data/.zeroclaw/config.toml && \
    chown -R 65534:65534 /zeroclaw-data
ENV LANG=C.UTF-8
ENV ZEROCLAW_WORKSPACE=/zeroclaw-data/workspace
ENV HOME=/zeroclaw-data
ENV ZEROCLAW_GATEWAY_PORT=42617
WORKDIR /zeroclaw-data
USER 65534:65534
EXPOSE 42617
ENTRYPOINT ["zeroclaw"]
CMD ["daemon"]
