ARG UBUNTU_VER=24.04
ARG GOLANG_VER=1.23

FROM ubuntu:${UBUNTU_VER} AS ubuntu
FROM golang:${GOLANG_VER}-bookworm AS golang

# 
# BUILD
# 
FROM golang AS go-builder

ARG GO_CRON_VER=v0.0.6
ARG WATCHER_VER=v1.0.7a

RUN \
    echo "**** build go-cron ${GO_CRON_VER} ****" && \
    go install github.com/by275/go-cron@${GO_CRON_VER}

RUN \
    echo "**** build watcher ${WATCHER_VER} ****" && \
    go install github.com/by275/watcher/cmd/watcher@${WATCHER_VER}


FROM ubuntu AS ubuntu-builder

ARG TARGETARCH
ARG DEBIAN_FRONTEND="noninteractive"

ARG OVERLAY_VER=v3.2.1.0
ARG OVERLAY_URL_BASE="https://github.com/just-containers/s6-overlay/releases/download/${OVERLAY_VER}"

RUN \
    apt-get update -qq && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        xz-utils

RUN \
    echo "**** add s6-overlay ${OVERLAY_VER} ****" && \
    mkdir -p /s6 && \
    if [ "$TARGETARCH" = "arm64" ]; then \
        OVERLAY_ARCH=aarch64 ; \
    elif [ "$TARGETARCH" = "arm" ]; then \
        OVERLAY_ARCH=armhf ; \
    elif [ "$TARGETARCH" = "amd64" ]; then \
        OVERLAY_ARCH="x86_64" ; \
    else echo "unknown architecture '${TARGETARCH}'" ; exit 1 ; fi && \
    curl -sL "${OVERLAY_URL_BASE}/s6-overlay-noarch.tar.xz" | tar Jxpf - -C /s6 && \
    curl -sL "${OVERLAY_URL_BASE}/s6-overlay-${OVERLAY_ARCH}.tar.xz" | tar Jxpf - -C /s6 && \
    curl -sL "${OVERLAY_URL_BASE}/s6-overlay-symlinks-noarch.tar.xz" | tar Jxpf - -C /s6 && \
    curl -sL "${OVERLAY_URL_BASE}/s6-overlay-symlinks-arch.tar.xz" | tar Jxpf - -C /s6

# 
# RELEASE
# 
FROM scratch
LABEL maintainer="by275"
LABEL org.opencontainers.image.source=https://github.com/by275/docker-base
COPY --from=go-builder /go/bin/ /go/bin/
COPY --from=ubuntu-builder /s6/ /s6/
