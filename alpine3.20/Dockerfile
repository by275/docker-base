ARG ALPINE_VER=3.20

FROM alpine:${ALPINE_VER}
LABEL maintainer="by275"
LABEL org.opencontainers.image.source=https://github.com/by275/docker-base

ENV \
    PS1="$(whoami)@$(hostname):$(pwd)\\$ " \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0

RUN \
    echo "**** install core packages ****" && \
    apk add --no-cache \
        bash \
        ca-certificates \
        coreutils \
        curl \
        procps \
        shadow \
        tzdata \
        && \
    echo "**** create abc user and make empty dirs ****" && \
    groupmod -g 1000 users && \
    useradd -u 911 -U -d /config -s /bin/false abc && \
    usermod -G users abc && \
    mkdir -p \
        /config \
        /defaults \
        && \
    echo "**** cleanup ****" && \
    rm -rf \
        /tmp/*
