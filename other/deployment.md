# deployment

- [deployment](#deployment)
  - [introduction](#introduction)
  - [architecture overview](#architecture-overview)
  - [automated deployment with dstrn deploy](#automated-deployment-with-dstrn-deploy)
    - [generating configuration files](#generating-configuration-files)
    - [dry run preview](#dry-run-preview)
    - [direct installation](#direct-installation)
  - [single instance deployment with caddy](#single-instance-deployment-with-caddy)
    - [installing caddy](#installing-caddy)
    - [configuring caddyfile](#configuring-caddyfile)
  - [systemd service setup](#systemd-service-setup)
    - [creating the service](#creating-the-service)
    - [enabling and starting](#enabling-and-starting)
  - [environment configuration](#environment-configuration)
  - [performance optimization](#performance-optimization)
    - [zero loss tls offloading](#zero-loss-tls-offloading)
    - [websocket proxying](#websocket-proxying)

<a name="introduction"></a>

## introduction

dframework is engineered for throughput and deterministic execution. in production environments, to maximize request handling speed and maintain a minimal binary footprint, tls termination is offloaded to a dedicated edge proxy or cloud load balancer. this architecture keeps V8 event loop threads dedicated to application logic and routing while isolating cryptographic overhead and certificate management at the edge.

<a name="architecture-overview"></a>

## architecture overview

a standard single instance deployment uses a lightweight edge proxy to terminate https on port 443 and proxy plain http requests over local loopback (`127.0.0.1:825`) to dframework.

```
[ client ] --( https / tls 1.3:443 )--> [ caddy edge proxy ]
                                                 |
                                     ( http loopback:825 )
                                                 v
                                     [ dframework server ]
```

this setup guarantees zero security compromise, automated zero maintenance ssl certificates, and maximum request throughput.

<a name="automated-deployment-with-dstrn-deploy"></a>

## automated deployment with dstrn deploy

dframework includes a dedicated deployment helper via the `dstrn deploy` cli command. it automates production setup by generating or installing caddy and systemd service configurations.

<a name="generating-configuration-files"></a>

### generating configuration files

run `dstrn deploy` in your application root:

```bash
dstrn deploy --domain=api.example.com
```

this will:
1. verify or generate a missing production `APP_KEY` in `.env`.
2. create a `deploy/Caddyfile` configured for your domain and application port.
3. create a `deploy/dframework-<app>.service` systemd unit file.
4. print the exact `sudo` shell commands to copy and activate the services.

<a name="dry-run-preview"></a>

### dry run preview

preview generated configuration files without writing them to disk:

```bash
dstrn deploy --domain=api.example.com --dry-run
```

<a name="direct-installation"></a>

### direct installation

when running on your Linux production server with root privileges (sudo), `dstrn deploy` can automatically write configuration files to system paths and start the services:

```bash
sudo dstrn deploy --domain=api.example.com --install
```

<a name="single-instance-deployment-with-caddy"></a>

## single instance deployment with caddy

caddy is the recommended reverse proxy for single instance deployments on aws ec2 or traditional servers. it provides zero touch let's encrypt ssl certificate management, automated http to https redirection, and HTTP/2 and HTTP/3 support by default.

<a name="installing-caddy"></a>

### installing caddy

install caddy on debian or ubuntu:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

<a name="configuring-caddyfile"></a>

### configuring caddyfile

edit the caddy configuration at `/etc/caddy/Caddyfile`:

```caddy
api.example.com {
    reverse_proxy 127.0.0.1:825
}
```

reload caddy to apply changes:

```bash
sudo systemctl reload caddy
```

caddy automatically provisions let's encrypt certificates for `api.example.com`, enables tls 1.3, sets up ocsp stapling, and proxies all incoming traffic to your running dframework instance.

<a name="systemd-service-setup"></a>

## systemd service setup

to ensure your application runs continuously, recovers from server reboots, and automatically restarts on failure, configure a systemd service.

<a name="creating-the-service"></a>

### creating the service

create `/etc/systemd/system/dframework.service`:

```ini
[Unit]
Description=dframework application server
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/var/www/my-app
ExecStart=/usr/bin/node dstrn serve
Restart=always
RestartSec=3
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
```

<a name="enabling-and-starting"></a>

### enabling and starting

reload systemd and enable the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dframework
sudo systemctl start dframework
```

check service status:

```bash
sudo systemctl status dframework
```

<a name="environment-configuration"></a>

## environment configuration

ensure your production `.env` file is present in your application root with appropriate production settings:

```ini
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.example.com
APP_PORT=825
APP_KEY=your_generated_64_character_hex_key
```

generate a production key if needed:

```bash
dstrn key:generate
```

<a name="performance-optimization"></a>

## performance optimization

<a name="zero-loss-tls-offloading"></a>

### zero loss tls offloading

by offloading tls encryption and decryption to caddy, the cpu hardware acceleration occurs in caddy's worker threads on dedicated kernel loops.

dframework receives plain HTTP over `127.0.0.1` TCP loopback, allowing the transport to operate at full native speed with no V8 thread blockage caused by in process handshakes.

<a name="websocket-proxying"></a>

### websocket proxying

dframework includes native websocket support registered under `/ws`. caddy automatically handles websocket upgrade requests with zero additional configuration required.

incoming `ws://` or `wss://` connections routed through caddy to `/ws` will seamlessly upgrade to high performance websocket connections.