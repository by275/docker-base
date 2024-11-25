ARG UBUNTU_VER=22.04

FROM ubuntu:${UBUNTU_VER}
LABEL maintainer="by275"
LABEL org.opencontainers.image.source=https://github.com/by275/docker-base

ARG DEBIAN_FRONTEND="noninteractive"

ENV \
    LANGUAGE="en_US.UTF-8" \
    LANG="en_US.UTF-8" \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0

RUN \
    echo "**** install core packages ****" && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        apt-utils \
        ca-certificates \
        curl \
        locales \
        tzdata \
        && \
    echo "**** generate locale ****" && \
    locale-gen en_US.UTF-8 && \
    echo "**** create abc user and make empty dirs ****" && \
    useradd -u 911 -U -d /config -s /bin/false abc && \
    usermod -G users abc && \
    mkdir -p \
        /config \
        /defaults \
        && \
    echo "**** cleanup ****" && \
    rm -rf \
        /tmp/* \
        /var/tmp/* \
        /var/cache/* \
        /var/lib/apt/lists/*
