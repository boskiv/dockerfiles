FROM hairyhenderson/gomplate:v3.3.1-slim AS gomplate
FROM docker:18.09.3 AS docker
FROM docker/compose:1.23.2 AS compose
FROM vault:0.11.6 AS vault
FROM hashicorp/terraform:0.11.13 AS terraform
FROM hashicorp/packer:1.3.5 AS packer
FROM prom/prometheus:v2.8.0 AS prometheus
FROM mikefarah/yq:2.2.0 AS yq

FROM alpine:3.8 AS packages

RUN apk add --no-cache \
    zip \
    unzip \
    bats \
    bash \
    grep \
    sed \
    make \
    jq \
    curl \
    git \
    python \
    py2-pip \
    openssh-client \
    openssl \
    libgcc

ENV GLIBC=2.28-r0

RUN wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub

COPY awscli-requirements.txt requirements.txt
RUN pip install -r requirements.txt
RUN rm -rf requirements.txt ~/.cache

RUN wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/$GLIBC/glibc-$GLIBC.apk \
  && apk add --no-cache glibc-$GLIBC.apk \
  && rm glibc-$GLIBC.apk \
  && ln -s /lib/libz.so.1 /usr/glibc-compat/lib \
  && ln -s /lib/libc.musl-x86_64.so.1 /usr/glibc-compat/lib \
  && ln -s /usr/lib/libgcc_s.so.1 /usr/glibc-compat/lib

FROM golang:1.12-alpine AS gopackages

RUN apk add --no-cache git
RUN go get -u github.com/golang/dep/cmd/dep
RUN go get -u github.com/jstemmer/go-junit-report
RUN go get -u github.com/alecthomas/gometalinter \
  && gometalinter --install

FROM golang:1.12-alpine AS build

COPY --from=gomplate /gomplate /bin/gomplate
COPY --from=docker /usr/local/bin/docker /bin/docker
COPY --from=vault /bin/vault /bin/vault
COPY --from=terraform /bin/terraform /bin/terraform
COPY --from=packer /bin/packer /bin/packer
COPY --from=prometheus /bin/promtool /bin/promtool

COPY --from=compose /usr/local/bin/docker-compose /bin/docker-compose

COPY --from=packages / /
COPY --from=gopackages /go /go
