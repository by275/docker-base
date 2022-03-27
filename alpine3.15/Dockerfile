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

# 
# RELEASE
# 
FROM alpine
LABEL maintainer="by275"
LABEL org.opencontainers.image.source https://github.com/by275/docker-base
COPY --from=go-builder /go/bin/ /go/bin/
