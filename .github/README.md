# caddy-cf

Custom Caddy build which adds cloudflare

## Why

The default caddy images do not include the cloudflare DNS challenge method of gaining a TLS certificate using certbot.

## How

A github action runs `docker build` using the `Dockerfile`, this will then push that image to a github image repository that I can use to pull the latest image to any infrastructure that requires it.

Previously I would build the image on each host which wastes a lot of time especially if the VM / LXC / Host does not have the resources to compile quickly.

With this I create the image once using github actions.

## compose.yml
``` yaml
services:
  caddy:
    image: ghcr.io/ahmzaa/caddy-cf:latest
    restart: unless-stopped
    container_name: caddy-cf
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./conf:/etc/caddy
      - ./site:/srv
      - ./data:/data
      - ./config:/config
    networks:
      net:
        ipv4_address: 172.21.0.2

networks:
  net:
    ipam:
      config:
        - subnet: 172.21.0.0/16
```

## Cadyfile
```
(cloudflare) {
  tls {
    dns cloudflare TOKEN
  }
}

# Reverse Proxy Definitions
test.example.com {
  reverse_proxy http://your-server:1234
  import cloudflare
}
```
