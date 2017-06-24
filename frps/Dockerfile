FROM alpine
MAINTAINER Yangxuan <yangxuan8282@gmail.com>

ENV FRP_VERSION=0.11.0

COPY frps run_frps /usr/bin/

EXPOSE 80 7000 7500

ENV BIND_PORT="7000" \
    KCP_BIND_PORT="7000" \
    VHOST_HTTP_PORT="80" \
    PRIVILEGE_TOKEN="youcanusefrpwithDocker" \
    FRP_DASHBOARD_PORT="7500" \
    FRP_DASHBOARD_USER="admin" \
    FRP_DASHBOARD_PWD="youcanusefrpwithDocker"

ENTRYPOINT ["run_frps"]
CMD ["-c", "/etc/frp/frps.ini"]
