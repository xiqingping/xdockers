FROM debian:stable-slim as builder

ARG MINISIG=0.12
ARG ZIG_MINISIG=RWSGOq2NVecA2UPNdBUZykf1CCb147pkmdtYxgb3Ti+JO/wCYvhbAb/U
ARG V8=14.0.365.4
ARG ZIG_V8=v0.3.9
ARG TARGETPLATFORM

RUN echo "deb http://mirrors.ustc.edu.cn/debian trixie main contrib non-free non-free-firmware" > /etc/apt/sources.list
RUN echo "deb http://mirrors.ustc.edu.cn/debian trixie-updates main contrib non-free non-free-firmware" >> /etc/apt/sources.list

RUN apt-get update -yq && \
    apt-get install -yq xz-utils ca-certificates \
        pkg-config libglib2.0-dev \
        clang make curl git golang-go

# Get Rust
RUN curl https://sh.rustup.rs -sSf | sh -s -- --profile minimal -y
ENV PATH="/root/.cargo/bin:${PATH}"

# install minisig
RUN curl --fail -L -O https://github.com/jedisct1/minisign/releases/download/${MINISIG}/minisign-${MINISIG}-linux.tar.gz && \
    tar xvzf minisign-${MINISIG}-linux.tar.gz -C /

# clone lightpanda
RUN git clone https://github.com/lightpanda-io/browser.git
WORKDIR /browser

# install zig
RUN ZIG=$(grep '\.minimum_zig_version = "' "build.zig.zon" | cut -d'"' -f2) && \
    case $TARGETPLATFORM in \
      "linux/arm64") ARCH="aarch64" ;; \
      *) ARCH="x86_64" ;; \
    esac && \
    curl --fail -L -O https://ziglang.org/download/${ZIG}/zig-${ARCH}-linux-${ZIG}.tar.xz && \
    curl --fail -L -O https://ziglang.org/download/${ZIG}/zig-${ARCH}-linux-${ZIG}.tar.xz.minisig && \
    /minisign-linux/${ARCH}/minisign -Vm zig-${ARCH}-linux-${ZIG}.tar.xz -P ${ZIG_MINISIG} && \
    tar xvf zig-${ARCH}-linux-${ZIG}.tar.xz && \
    mv zig-${ARCH}-linux-${ZIG} /usr/local/lib && \
    ln -s /usr/local/lib/zig-${ARCH}-linux-${ZIG}/zig /usr/local/bin/zig

# download and install v8
RUN case $TARGETPLATFORM in \
    "linux/arm64") ARCH="aarch64" ;; \
    *) ARCH="x86_64" ;; \
    esac && \
    curl --fail -L -o libc_v8.a https://github.com/lightpanda-io/zig-v8-fork/releases/download/${ZIG_V8}/libc_v8_${V8}_linux_${ARCH}.a && \
    mkdir -p v8/ && \
    mv libc_v8.a v8/libc_v8.a

# build v8 snapshot
RUN zig build -Doptimize=ReleaseFast \
    -Dprebuilt_v8_path=v8/libc_v8.a \
    snapshot_creator -- src/snapshot.bin

# build release
RUN zig build -Doptimize=ReleaseFast \
    -Dsnapshot_path=../../snapshot.bin \
    -Dprebuilt_v8_path=v8/libc_v8.a

WORKDIR /
RUN git clone https://github.com/sipeed/picoclaw /picoclaw

WORKDIR /picoclaw
RUN make build
RUN make build-launcher

FROM debian:stable-slim
RUN apt-get update && apt-get install -y supervisor
COPY *.conf /etc/supervisor/conf.d/

VOLUME /root/.picoclaw

# copy ca certificates
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=builder /browser/zig-out/bin/lightpanda /usr/local/bin/lightpanda
COPY --from=builder /picoclaw/build/picoclaw /usr/local/bin/picoclaw
COPY --from=builder /picoclaw/build/picoclaw-launcher /usr/local/bin/picoclaw-launcher

CMD ["/usr/bin/supervisord", "-n"]
