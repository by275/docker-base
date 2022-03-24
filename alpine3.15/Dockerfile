ARG ALPINE_VER=3.15
ARG GOLANG_VER=1.16

FROM alpine:${ALPINE_VER} AS alpine
FROM golang:${GOLANG_VER}-alpine${ALPINE_VER} AS golang

RUN \
    echo "**** install build packages ****" && \
    apk add --update --no-cache curl

# 
# BUILD
# 
FROM golang AS go-builder

ARG GO_CRON_VER=0.0.4
ARG GO_CRON_SHA256=6c8ac52637150e9c7ee88f43e29e158e96470a3aaa3fcf47fd33771a8a76d959
ARG WATCHER_VER=1.0.7

RUN \
    echo "**** build go-cron v${GO_CRON_VER} ****" && \
    curl -sL -o go-cron.tar.gz https://github.com/djmaze/go-cron/archive/v${GO_CRON_VER}.tar.gz && \
    echo "${GO_CRON_SHA256}  go-cron.tar.gz" | sha256sum -c - && \
    tar xzf go-cron.tar.gz && \
    cd go-cron-${GO_CRON_VER} && \
    go install

RUN \
    echo "**** build watcher v${WATCHER_VER} ****" && \
    go install github.com/radovskyb/watcher/cmd/watcher@v${WATCHER_VER}

# 
# RELEASE
# 
FROM alpine
LABEL maintainer="by275"
LABEL org.opencontainers.image.source https://github.com/by275/docker-base
COPY --from=go-builder /go/bin/ /go/bin/
