# deployment

- [deployment](#deployment)
  - [introduction](#introduction)
  - [deployment strategies](#deployment-strategies)
    - [direct bare metal (default)](#direct-bare-metal-default)
    - [reverse proxy with caddy](#reverse-proxy-with-caddy)
    - [edge deployment with cloudflare](#edge-deployment-with-cloudflare)
  - [automated deployment](#automated-deployment)
    - [direct bare metal setup](#direct-bare-metal-setup)
    - [reverse proxy setup](#reverse-proxy-setup)
    - [edge setup](#edge-setup)
    - [dry run preview](#dry-run-preview)
    - [system installation](#system-installation)
  - [manual deployment setup](#manual-deployment-setup)
    - [manual direct bare metal](#manual-direct-bare-metal)
    - [manual caddy reverse proxy](#manual-caddy-reverse-proxy)
  - [tls and certificate management](#tls-and-certificate-management)
    - [automatic certificate discovery](#automatic-certificate-discovery)
    - [certificate management commands](#certificate-management-commands)
      - [obtain certificates](#obtain-certificates)
      - [renew certificates](#renew-certificates)
      - [inspect certificate status](#inspect-certificate-status)
    - [multi domain and sni routing](#multi-domain-and-sni-routing)
    - [http to https redirection](#http-to-https-redirection)
  - [websocket secure connections](#websocket-secure-connections)
    - [automatic wss promotion](#automatic-wss-promotion)
    - [native websocket upgrades](#native-websocket-upgrades)
  - [systemd service setup](#systemd-service-setup)
    - [application service unit](#application-service-unit)
    - [certificate renewal timer unit](#certificate-renewal-timer-unit)
  - [environment configuration](#environment-configuration)
  - [performance considerations](#performance-considerations)

<a name="introduction"></a>

## introduction

dframework is engineered for raw throughput and low latency execution. for production environments, dframework offers three deployment architectures:

1. direct bare metal with native tls (default): single process architecture where the native transport addon handles openssl tls 1.3 termination, http to https redirects, and routing.
2. reverse proxy (`--proxy`): traditional deployment behind caddy or nginx when sharing ports with other services or integrating with an existing proxy stack.
3. edge deployment (`--edge`): running plain http behind cloudflare or another cdn where tls termination occurs at the edge.

<a name="deployment-strategies"></a>

## deployment strategies

<a name="direct-bare-metal-default"></a>

### direct bare metal (default)

in direct bare metal mode, dframework binds port 443 with native openssl tls and simultaneously runs a lightweight listener on port 80 to redirect http traffic to https.

```
[ client ] --( https:443 )--> [ dframework ssl app (native tls) ]
[ client ] --( http:80 )----> [ dframework redirect app (301) ]
```

benefits:

- zero reverse proxy overhead (no extra tcp loopback hop or context switching).
- native multi domain sni support via `storage/certs/{domain}/`.
- single node process running under systemd with non root privileges.

<a name="reverse-proxy-with-caddy"></a>

### reverse proxy with caddy

in reverse proxy mode, a dedicated web server such as caddy receives public https traffic on port 443 and proxies plain http requests over local loopback (`127.0.0.1:825`) to dframework.

```
[ client ] --( https:443 )--> [ caddy ] --( loopback:825 )--> [ dframework ]
```

when to choose a reverse proxy:

- hosting multiple independent applications on the same server sharing port 80 and 443.
- integrating with existing infrastructure that requires custom proxy middleware or external waf rules.

<a name="edge-deployment-with-cloudflare"></a>

### edge deployment with cloudflare

in edge mode, cloudflare terminates public tls connections and forwards requests to your origin server over http or https.

```
[ client ] --( https )--> [ cloudflare edge ] --( http/https )--> [ dframework ]
```

benefits:

- ddos protection and global cdn caching at the edge.
- origin server can run on an internal port with firewall rules restricting access to cloudflare ip addresses.

<a name="automated-deployment-with-dstrn-deploy"></a>

## automated deployment

the `dstrn deploy` command automates configuration generation, certificate provisioning, and service installation.

<a name="direct-bare-metal-setup"></a>

### direct bare metal setup

to generate bare metal production configuration for your domain:

```bash
dstrn deploy --domain=api.example.com --email=admin@example.com
```

for multi domain setups, provide comma separated domains:

```bash
dstrn deploy --domain=api.example.com,admin.example.com --email=admin@example.com
```

this command performs the following actions:

1. verifies or generates a production `APP_KEY` in `.env`.
2. checks for existing certificates in `storage/certs/{domain}/` and obtains missing certificates.
3. creates `deploy/dframework-<app>.service` with `AmbientCapabilities=CAP_NET_BIND_SERVICE` so the application can bind ports 80 and 443 without root privileges.
4. creates `deploy/dframework-<app>-renew.timer` and `deploy/dframework-<app>-renew.service` to automate weekly certificate renewal.

<a name="reverse-proxy-setup"></a>

### reverse proxy setup

to generate caddy and systemd service configurations:

```bash
dstrn deploy --proxy --domain=api.example.com
```

this generates:

1. `deploy/Caddyfile` with reverse proxy rules pointing to your application port.
2. `deploy/dframework-<app>.service` configured for standard unprivileged execution.

<a name="edge-setup"></a>

### edge setup

to generate systemd service files with cloudflare guidance:

```bash
dstrn deploy --edge --domain=api.example.com
```

<a name="dry-run-preview"></a>

### dry run preview

preview all generated service and configuration files in the terminal without writing to disk:

```bash
dstrn deploy --domain=api.example.com --email=admin@example.com --dry-run
```

<a name="system-installation"></a>

### system installation

when running on your server with root privileges (sudo), `dstrn deploy` can write directly to `/etc/systemd/system/` and activate the services:

```bash
sudo dstrn deploy --domain=api.example.com --email=admin@example.com --install
```

<a name="manual-deployment-setup"></a>

## manual deployment setup

if you prefer manual server provisioning or are using automation tools, follow these steps.

<a name="manual-direct-bare-metal"></a>

### manual direct bare metal

place your ssl certificate and private key in the application storage directory:

```
storage/certs/api.example.com/
  cert.pem
  key.pem
```

configure production environment variables in `.env`:

```ini
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.example.com
APP_KEY=your_64_character_hex_key
```

create `/etc/systemd/system/dframework.service`:

```ini
[Unit]
Description=dframework application server
After=network.target

[Service]
Type=simple
User=user
WorkingDirectory=/var/www/app
ExecStart=/usr/bin/node dstrn serve
Restart=always
RestartSec=3
Environment=NODE_ENV=production
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dframework
sudo systemctl start dframework
```

<a name="manual-caddy-reverse-proxy"></a>

### manual caddy reverse proxy

install caddy on debian or ubuntu:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

configure `/etc/caddy/Caddyfile`:

```caddy
api.example.com {
    reverse_proxy 127.0.0.1:825
}
```

reload caddy:

```bash
sudo systemctl reload caddy
```

caddy will automatically obtain let's encrypt certificates for `api.example.com`, manage renewals, and proxy requests over loopback to dframework.

<a name="tls-and-certificate-management"></a>

## tls and certificate management

<a name="automatic-certificate-discovery"></a>

### automatic certificate discovery

dframework automatically detects certificates in `storage/certs/` upon startup:

```
storage/certs/
  api.example.com/
    cert.pem (or fullchain.pem, certificate.crt, api.example.com.crt)
    key.pem  (or privkey.pem, private.key, api.example.com.key)
    ca.pem   (optional intermediate chain)
  admin.example.com/
    cert.pem
    key.pem
```

when valid certificates are found and `APP_URL` starts with `https://`, dframework automatically boots with native openssl tls on port 443 and starts the http to https redirect listener on port 80.

<a name="certificate-management-commands"></a>

### certificate management commands

the `dstrn cert` command family provides management over the certificate lifecycle.

#### obtain certificates

obtain a new let's encrypt certificate for one or more domains:

```bash
dstrn cert:obtain --domain=api.example.com --email=admin@example.com
```

for testing with let's encrypt staging ca (avoids rate limits):

```bash
dstrn cert:obtain --domain=api.example.com --email=admin@example.com --staging
```

#### renew certificates

renew all certificates that expire within 30 days:

```bash
dstrn cert:renew --domain=api.example.com --email=admin@example.com
```

#### inspect certificate status

check expiration dates, issuer information, and validity for all installed certificates:

```bash
dstrn cert:status
```

<a name="multi-domain-and-sni-routing"></a>

### multi domain and sni routing

the transport supports server name indication (sni). when multiple domain directories exist in `storage/certs/`, each domain receives its matching certificate during the tls handshake.

pair this with domain route groups in your application routes:

```javascript
Route.domain('api.example.com', (api) => {
  api.get('/status', async () => json({ service: 'api' }));
});

Route.domain('admin.example.com', (admin => {
  admin.get('/status', async () => json({ service: 'admin' }));
});
```

<a name="http-to-https-redirection"></a>

### http to https redirection

when tls is enabled, dframework automatically starts a native non ssl listener on port 80. any request to `http://domain/path` receives an immediate `301 Moved Permanently` response pointing to `https://domain/path`.

to disable the redirect listener, set `tls.redirect: false` in `config/app.js` or `TLS_REDIRECT=false` in `.env`.

<a name="websocket-secure-connections"></a>

## websocket secure connections

<a name="automatic-wss-promotion"></a>

### automatic wss promotion

the client automatically promotes websocket connections to `wss://` whenever the page is loaded over `https://`.

because the initial page request is redirected from http to https, the browser always loads over `https:` and establishes a secure `wss://` connection.

<a name="native-websocket-upgrades"></a>

### native websocket upgrades

on the server side, websocket connections over port 443 are handled directly inside the ssl event loop. the http upgrade handshake occurs over the existing encrypted tls connection, ensuring zero plaintext exposure and instant websocket availability.

<a name="systemd-service-setup"></a>

## systemd service setup

<a name="application-service-unit"></a>

### application service unit

the production systemd unit file (`/etc/systemd/system/dframework.service`):

```ini
[Unit]
Description=dframework application server
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/var/www/app
ExecStart=/usr/bin/node dstrn serve
Restart=always
RestartSec=3
Environment=NODE_ENV=production
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

the `AmbientCapabilities=CAP_NET_BIND_SERVICE` setting allows the service to bind privileged low ports (80 and 443) while running as a standard non root user.

<a name="certificate-renewal-timer-unit"></a>

### certificate renewal timer unit

automated certificate renewal uses a weekly systemd timer (`/etc/systemd/system/dframework-renew.timer`):

```ini
[Unit]
Description=certificate renewal timer for dframework

[Timer]
OnCalendar=weekly
RandomizedDelaySec=3600
Persistent=true

[Install]
WantedBy=timers.target
```

with the corresponding oneshot renewal service (`/etc/systemd/system/dframework-renew.service`):

```ini
[Unit]
Description=certificate renewal for dframework

[Service]
Type=oneshot
WorkingDirectory=/var/www/app
ExecStart=/usr/bin/node dstrn cert:renew --domain=api.example.com --email=admin@example.com
ExecStartPost=/bin/systemctl restart dframework
```

enable the renewal timer with:

```bash
sudo systemctl daemon-reload
sudo systemctl enable dframework-renew.timer
sudo systemctl start dframework-renew.timer
```

<a name="environment-configuration"></a>

## environment configuration

standard production `.env` configuration:

```ini
APP_NAME=dframework
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.example.com
APP_KEY=your_generated_64_character_hex_key
ACME_EMAIL=admin@example.com
```

optional explicit tls configuration (overrides auto discovery convention):

```ini
TLS_CERT=storage/certs/api.example.com/cert.pem
TLS_KEY=storage/certs/api.example.com/key.pem
TLS_CA=storage/certs/api.example.com/ca.pem
TLS_REDIRECT=true
```

<a name="performance-considerations"></a>

## performance considerations

traditional architectures place a reverse proxy like caddy or nginx in front of application runtimes. in benchmark measurements, a reverse proxy layer can potentially add over 60% of overhead for simple routes due to double socket buffering, process context switching, and loopback tcp serialization.

direct bare metal deployment eliminates intermediate proxy layers, delivering maximum throughput directly to the application event loop while maintaining automated let's encrypt certificate management and multi domain support.
