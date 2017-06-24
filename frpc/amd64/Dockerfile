FROM nimmis/alpine:3.3
MAINTAINER Yangxuan <yangxuan8282@gmail.com>

ENV FRP_VERSION=0.12.0

RUN apk --no-cache --update add curl \
 && mkdir -p /tmp/frp \
 && curl -sL -o /tmp/frp/frp.tar.gz https://github.com/fatedier/frp/releases/download/v${FRP_VERSION}/frp_${FRP_VERSION}_linux_amd64.tar.gz \
 && tar -zxf /tmp/frp/frp.tar.gz -C /tmp/frp \
 && mv /tmp/frp/frp_${FRP_VERSION}_linux_amd64/frpc /usr/bin/ \
 && rm -rf /tmp/frp

ENV DOCKER_GEN_VERSION=0.7.3

RUN curl -sL -o docker-gen-linux-amd64-$DOCKER_GEN_VERSION.tar.gz https://github.com/jwilder/docker-gen/releases/download/$DOCKER_GEN_VERSION/docker-gen-linux-amd64-$DOCKER_GEN_VERSION.tar.gz \
 && tar -C /usr/local/bin -xvzf docker-gen-linux-amd64-$DOCKER_GEN_VERSION.tar.gz \
 && rm /docker-gen-linux-amd64-$DOCKER_GEN_VERSION.tar.gz

ADD etc /etc

ENV FRP_SERVER_IP=127.0.0.1 \
    FRP_SERVER_PORT=7000 \
    FRP_KCP_PORT=7000 \
    FRP_PROTOCOL=tcp \
    PRIVILEGE_TOKEN=youcanusefrpwithDocker \
    FRP_TYPE=http \
    IF_USE_ENCRYPTION=true \
    IF_USE_COMPRESSION=true \
    FRP_DASHBOARD_PORT=7500 \
    FRP_DASHBOARD_DOMAIN=frp.example.com \
    SSH_LOCAL_IP=127.0.0.1 \
    SSH_LOCAL_PORT=22 \
    SSH_REMOTE_PORT=10022 \
    FRP_UDP_LOCAL_IP=8.8.8.8 \
    FRP_UDP_LOCAL_PORT=53 \
    FRP_UDP_REMOTE_PORT=5353

ENV DOCKER_HOST unix:///tmp/docker.sock

