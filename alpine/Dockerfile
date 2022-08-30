ARG ALPINE_VER=3.16
ARG GOLANG_VER=1.18

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

ARG OVERLAY_VER=v3.1.2.1
ARG OVERLAY_URL_BASE="https://github.com/just-containers/s6-overlay/releases/download/${OVERLAY_VER}"

RUN \
    apk add --no-cache \
        curl \
        xz

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
LABEL org.opencontainers.image.source https://github.com/by275/docker-base
COPY --from=go-builder /go/bin/ /go/bin/
COPY --from=alpine-builder /s6/ /s6/
