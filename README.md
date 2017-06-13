frpd - [frp](https://github.com/fatedier/frp) in Docker
===

frpd is pretty similar to [jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy), but support more protocol [ssh/tcp/udp] 

It also use docker-gen generates reverse proxy configs for frp and restart frp when containers are started and stopped.

### Docker Compose

there are two containers: frps (server) and frpc (client)

create network first (on server and client):

```
docker network create proxy-network
```

then run frps with this `docker-compose.yml` template:

> the default token and frp dashboard passwd is `youcanusefrpwithDocker`, you need pass your own token in `environment`, for more environment variables please check Dockerfile  

```
version: "2"

services:
  frps:
    image: yangxuan8282/frpds
    ports:
      - "80:80"
      - "7000:7000"
      - "7500:7500"

    environment:
      - PRIVILEGE_TOKEN=TOKEN_HERE

networks:
  default:
    external:
      name: proxy-network
``` 

next run frpc:

> replace value of `FRP_SERVER_IP` && `PRIVILEGE_TOKEN`

```
version: "2"

services:
  frpdc:
    image: yangxuan8282/frpdc
    container_name: frpdc
    restart: always
    volumes:
      - "/var/run/docker.sock:/tmp/docker.sock:ro"

    environment:
      - FRP_SERVER_IP=YOUR_FRP_SERVER_IP_HERE
      - PRIVILEGE_TOKEN=TOKEN_HERE

networks:
  default:
    external:
      name: proxy-network
```

finally start your apps, take `ghost` for example:

> replace the value of `VIRTUAL_HOST` with your domain name

```
version: "2"

services:
  app:
    image: ghost:alpine
    restart: "always"
    expose:
      - 2368
    volumes:
      - "./ghost:/var/lib/ghost"
    environment:
      - VIRTUAL_HOST=blog.mydomain.com
      - VIRTUAL_PORT=2368
      - VIRTUAL_NETWORK=proxy-network

networks:
  default:
    external:
      name: proxy-network
```

for more templates, please check this [repo](https://github.com/yangxuan8282/docker-recipes), just remove `LETSENCRYPT_HOST` && `LETSENCRYPT_EMAIL` environment variables

### About

```
user --------------------------> frps ------> frpc --------> apps


      | <------------ remote server with public ip -------------> |

or   

      | <-- remote server with public ip --> | <-- PC in home --> |

or

      | <------------ remote server with public ip -------------> |

                                             | <-- PC in home --> |

```

> `frps` && `frpc` can be run on same machine, but also work if on different one, and you can run `frps` on multiple computer at the same time.

### UDP && SSH

frps:

> by default ssh will forwarding to server `10022` port

```
version: "2"

services:
  frps:
    image: local/frpds
    ports:
      - "80:80"
      - "7000:7000"
      - "7500:7500"
      - "5353:5353"
      - "5353:5353/udp"
      - "10022:10022"

    environment:
      - PRIVILEGE_TOKEN=it'sasecert

networks:
  default:
    external:
      name: proxy-network
```

then you need to edit `frpc.tmpl`

download `frpc.tmpl`, as the description say, uncomment the related block

```
mkdir -p ~/work/run/frpc/frpc &&
wget -O ~/work/run/frpc/frpc.tmpl https://raw.githubusercontent.com/yangxuan8282/frpd/master/frpc/amd64/etc/frp/frpc.tmpl
```

then start `frpc` mount your edited `frpc.tml`, and pass related environment variables in `docker-compose.yml`

```
echo "version: "2"

services:
  frpdc:
    image: yangxuan8282/frpdc
    container_name: frpdc
    restart: always
    volumes:
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
      - "./frpc.tml:/etc/frp/frpc.tml"
    environment:
      - FRP_SERVER_IP=YOUR_FRP_SERVER_IP_HERE
      - PRIVILEGE_TOKEN=TOKEN_HERE
      - FRP_DASHBOARD_PORT=7500
      - FRP_DASHBOARD_DOMAIN=frp.example.com 
      - SSH_LOCAL_IP=HOST_IP
      - SSH_LOCAL_PORT=22
      - SSH_REMOTE_PORT=10022
      - FRP_UDP_LOCAL_IP=8.8.8.8
      - FRP_UDP_LOCAL_PORT=53
      - FRP_UDP_REMOTE_PORT=5353
 
networks:
  default:
    external:
      name: proxy-network" | tee ~/work/run/frpc/docker-compose.yml
```

then you can ssh into client with:

```
ssh -p 10022 user@FRP_SERVER_IP
```

and you can test udp forwarding(the template in `frpc.tmpl` is forward DNS query request):

```
dig @FRP_SERVER_IP -p 5353 www.google.com
```

### frp dashboard

frp dashboard is already enabled in `frps` container, the default port is `7500`, user is `admin`, passwd is `youcanusefrpwithDocker` 

what we need is config `frpc`

uncomment this section in `frpc.tmpl`

```
...

# uncomment this block to visit frp dashboard, and you need pass ENV when start container
# [frp_dashboard]
# type = http
# local_ip = {{ .Env.FRP_SERVER_IP }}
# local_port = {{ .Env.FRP_DASHBOARD_PORT }}
# use_encryption = {{ .Env.IF_USE_ENCRYPTION }}
# use_compression  = {{ .Env.IF_USE_COMPRESSION }}
# custom_domains = {{ .Env.FRP_DASHBOARD_DOMAIN }}

...
```

and pass environment variables when we start container:

```
version: "2"

services:
  frpdc:
    image: yangxuan8282/frpdc
    container_name: frpdc
    restart: always
    volumes:
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
      - "./frpc.tmpl:/etc/frp/frpc.tmpl" 
    environment:
      - FRP_SERVER_IP=YOUR_FRP_SERVER_IP_HERE
      - PRIVILEGE_TOKEN=TOKEN_HERE
      - FRP_DASHBOARD_PORT=7500 
      - FRP_DASHBOARD_DOMAIN=frp.example.com

networks:
  default:
    external:
      name: proxy-network
```
