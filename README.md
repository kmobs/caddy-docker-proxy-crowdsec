# Caddy Docker Proxy with Crowdsec and Socket Proxy Example

## Introduction
Looking around, there didn't seem to be a good guide using caddy with crowdsec and appsec, particulary with Lucaslorentz's caddy-docker-proxy. I've been using this system with multiple containers and wanted them all integrated. 

## Setup
The setup consists of a custom caddy build that includes both [caddy-docker-proxy](https://github.com/lucaslorentz/caddy-docker-proxy) as well as [caddy-crowdsec-bouncer](https://github.com/hslatman/caddy-crowdsec-bouncer). 

In addition, it uses a readonly [socket-proxy](https://docs.linuxserver.io/images/docker-socket-proxy/).

Adding the block from the example file will add the container to your reverse proxy and protect it with crowdsec.

## Description

In the caddy container, labels are set where the API key, Appsec URL, and Crowdsec URL are set. 

These are placed in their own block using the naming convention from lucaslorentz

```
caddy_1.crowdsec.api_url: http://${API_URL}:24444
caddy_1.crowdsec.api_key: ${API_KEY}
caddy_1.crowdsec.appsec_url: http://${API_URL}:7422
```

I'm using a different port for the `api_url` because I had a conflict on the default 8080.


A logging snippet is also created to allow one to create access logs for crowdsec to read. By using a snippet with args, the name of the file can be specified per container.

```
caddy_2: (logging)
caddy_2.log.output: file /var/log/caddy/{args[0]}.access.log
caddy_2.log.output.roll_size: 10MiB
caddy_2.log.output.roll_keep_for: 168h
```

For example, to create logs for an nginx container, I may include `caddy.import: logging nginx`.

The routing block includes the reverse proxy as well as specifies that crowdsec and appsec will be invoked. This example would proxy port 80 from the specified container to subdomain.example.com.

```
caddy: subdomain.example.com
caddy.route.reverse_proxy: "{{upstreams 80}}"
caddy.route.crowdsec:
caddy.route.appsec:
``` 

In addition to this, the acquis file must be set to import all the caddy logs in the directory in the caddy format. It also sets the appsec port

```
filenames:
 - /var/log/caddy/*.access.log
labels:
  type: caddy
---
listen_addr: 0.0.0.0:7422
appsec_config: crowdsecurity/appsec-default
name: myAppSecComponent
source: appsec
labels:
  type: appsec
```


Hopefully this helps folks setup crowdsec and maybe make their network a little more secure. 
