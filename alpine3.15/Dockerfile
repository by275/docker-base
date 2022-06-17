ARG ALPINE_VER=3.15
ARG GOLANG_VER=1.17

FROM alpine:${ALPINE_VER} AS alpine
FROM golang:${GOLANG_VER}-alpine${ALPINE_VER} AS golang

# 
# BUILD
# 
FROM golang AS go-builder

ARG GO_CRON_VER=v0.0.5
ARG WATCHER_VER=v1.0.7

RUN \
    echo "**** build go-cron ${GO_CRON_VER} ****" && \
    go install github.com/by275/go-cron@${GO_CRON_VER}

RUN \
    echo "**** build watcher ${WATCHER_VER} ****" && \
    go install github.com/radovskyb/watcher/cmd/watcher@${WATCHER_VER}


FROM alpine AS alpine-builder

ARG TARGETARCH

RUN \
    apk add --no-cache \
        curl \
        xz

RUN \
    OVERLAY_VER=2.2.0.3 && \
    echo "**** add s6 overlay v${OVERLAY_VER} ****" && \
    mkdir -p /s6 && \
    if [ "$TARGETARCH" = "arm64" ]; then \
        OVERLAY_ARCH=aarch64 ; \
    elif [ "$TARGETARCH" = "arm" ]; then \
        OVERLAY_ARCH=armhf ; \
    elif [ "$TARGETARCH" = "amd64" ]; then \
        OVERLAY_ARCH="$TARGETARCH" ; \
    else echo "unknown architecture '${TARGETARCH}'" ; exit 1 ; fi && \
    curl -o /tmp/s6-overlay.tar.gz \
        -L "https://github.com/just-containers/s6-overlay/releases/download/v${OVERLAY_VER}/s6-overlay-${OVERLAY_ARCH}.tar.gz" && \
    tar xzf /tmp/s6-overlay.tar.gz -C /s6/ --exclude='./bin' && \
    tar xzf /tmp/s6-overlay.tar.gz -C /s6/usr ./bin

RUN \
    OVERLAY_VER=3.1.0.1 && \
    echo "**** add s6 overlay v${OVERLAY_VER} ****" && \
    mkdir -p /s6v3 && \
    if [ "$TARGETARCH" = "arm64" ]; then \
        OVERLAY_ARCH=aarch64 ; \
    elif [ "$TARGETARCH" = "arm" ]; then \
        OVERLAY_ARCH=armhf ; \
    elif [ "$TARGETARCH" = "amd64" ]; then \
        OVERLAY_ARCH="x86_64" ; \
    else echo "unknown architecture '${TARGETARCH}'" ; exit 1 ; fi && \
    curl -o /tmp/s6-overlay-noarch.tar.xz \
        -L "https://github.com/just-containers/s6-overlay/releases/download/v${OVERLAY_VER}/s6-overlay-noarch.tar.xz" && \
    tar -C /s6v3 -Jxpf /tmp/s6-overlay-noarch.tar.xz && \
    curl -o /tmp/s6-overlay-arch.tar.xz \
        -L "https://github.com/just-containers/s6-overlay/releases/download/v${OVERLAY_VER}/s6-overlay-${OVERLAY_ARCH}.tar.xz" && \
    tar -C /s6v3 -Jxpf /tmp/s6-overlay-arch.tar.xz && \    
    curl -o /tmp/s6-overlay-symlinks-noarch.tar.xz \
        -L "https://github.com/just-containers/s6-overlay/releases/download/v${OVERLAY_VER}/s6-overlay-symlinks-noarch.tar.xz" && \
    tar -C /s6v3 -Jxpf /tmp/s6-overlay-symlinks-noarch.tar.xz && \    
    curl -o /tmp/s6-overlay-symlinks-arch.tar.xz \
        -L "https://github.com/just-containers/s6-overlay/releases/download/v${OVERLAY_VER}/s6-overlay-symlinks-arch.tar.xz" && \
    tar -C /s6v3 -Jxpf /tmp/s6-overlay-symlinks-arch.tar.xz

# 
# RELEASE
# 
FROM alpine
LABEL maintainer="by275"
LABEL org.opencontainers.image.source https://github.com/by275/docker-base
COPY --from=go-builder /go/bin/ /go/bin/
COPY --from=alpine-builder /s6/ /s6/
COPY --from=alpine-builder /s6v3/ /s6v3/
