ARG UBUNTU_VER=22.04
ARG GOLANG_VER=1.17

FROM ubuntu:${UBUNTU_VER} AS ubuntu
FROM golang:${GOLANG_VER}-bullseye AS golang

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


FROM ubuntu AS ubuntu-builder

ARG TARGETARCH
ARG DEBIAN_FRONTEND="noninteractive"
ARG OVERLAY_VER=2.2.0.3

RUN \
    apt-get update -qq && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl

RUN \
    echo "**** add s6 overlay ****" && \
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

RUN echo "**** install docker-cli ****" && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        gnupg \
        lsb-release && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt-get update -qq && \
    apt-get install -y --no-install-recommends --download-only \
        docker-ce-cli && \
    dpkg -x /var/cache/apt/archives/docker-ce-cli*.deb /docker-cli

# 
# RELEASE
# 
FROM ubuntu
LABEL maintainer="by275"
LABEL org.opencontainers.image.source https://github.com/by275/docker-base
COPY --from=go-builder /go/bin/ /go/bin/
COPY --from=ubuntu-builder /s6/ /s6/
COPY --from=ubuntu-builder /docker-cli/ /docker-cli/