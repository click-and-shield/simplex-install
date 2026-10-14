# SimpleX SMP Server: Directory Layout

This Docker Compose deployment separates Caddy configuration, SMP server configuration and identity, and persistent server state.

The following directories are shared between the host and the containers through bind mounts:

| Host directory | Container path | Container |
|---|---|---|
| `caddy_conf` | `/etc/caddy` | `oneshot` and `caddy` |
| `smp_configs` | `/etc/opt/simplex` | `smp-server` |
| `smp_state` | `/var/opt/simplex` | `smp-server` |

Files written by a container under these paths are therefore also available directly on the host.

## `caddy_conf/`: Caddy Configuration

This directory contains the `Caddyfile`, which the `oneshot` container creates if it does not already exist.

In this deployment, the configuration instructs Caddy to:

- Redirect HTTP requests to HTTPS.
- Manage a TLS certificate for the configured domain using a 4096-bit RSA key.
- Configure an HTTPS listener on container port `8443`.

Only container port `80` is published to the host by the `caddy` service.

Certificates obtained by Caddy are stored separately in the Docker-managed `caddy_data` volume.

## `smp_configs/`: SMP Server Configuration and Cryptographic Identity

| File | Purpose |
|---|---|
| `smp-server.ini` | Server configuration, including transport ports, persistence, statistics, and Web service settings. The enabled features depend on its contents. |
| `ca.crt` | Certificate of the SMP server's own certificate authority. |
| `ca.key` | Private key of that certificate authority, used to sign SMP server certificates. |
| `ca.srl` | OpenSSL serial-number file used when signing certificates. |
| `server.crt` | SMP server TLS certificate, signed by its certificate authority. |
| `server.key` | Private key associated with `server.crt`. |
| `server.csr` | Certificate signing request used to generate `server.crt`. |
| `openssl_ca.conf` | OpenSSL settings used to generate the certificate authority certificate. |
| `openssl_server.conf` | OpenSSL settings used to generate the server certificate. |
| `fingerprint` | Fingerprint of the certificate authority certificate, included in the `smp://…` server address. |

### SMP Certificates and Web Certificates

The SMP certificates in this directory are separate from the Web certificate obtained by Caddy.

SimpleX clients verify the SMP server's identity using the certificate fingerprint included in its server address.

The `ca.key` file allows a replacement SMP server certificate to be signed while preserving the same server identity. The [SimpleX server documentation](https://simplex.chat/docs/server.html#private-keys) recommends keeping this key in secure storage outside the server.

## `smp_state/`: Persistent State and Web Files

| Entry | Purpose |
|---|---|
| `smp-server-store.log` | Queue persistence journal, used to restore SMP queue state after a restart. This is separate from the console output displayed by `docker compose logs`. |
| `smp-server-stats.daily.log` | Daily activity statistics, such as queue creation, queue deletion, and message traffic. |
| `backups/` | Directory intended for backups of server state. |
| `www/` | Static files used by the SMP server's Web service. |

### `backups/`

The directory contains three subdirectories:

| Subdirectory | Intended category |
|---|---|
| `messages/` | Message backups. |
| `queues/` | Queue backups. |
| `stats/` | Statistics backups. |

All three subdirectories are empty in the directory listing shown above.

Their presence alone does not establish whether scheduled backups are enabled, how often they run, or which mechanism populates them.

### `www/`

This directory contains the Web pages and associated resources provided with the server.

| Entry | Purpose |
|---|---|
| `index.html` | Server home and information page. |
| `contact/`, `invitation/` | Web pages associated with contact and invitation links. |
| `a/`, `c/`, `g/`, `i/`, `r/` | Pages associated with different SimpleX link routes. Their exact behavior depends on the installed version and file contents. |
| `media/` | Shared resources, including images, logos, fonts, stylesheets, JavaScript, QR-code functionality, and application download links. |
| `well-known/apple-app-site-association` | Domain association information for the iOS application. |
| `well-known/assetlinks.json` | Domain association information for the Android application. |

The SMP server can serve these files over HTTPS using the settings in the `[WEB]` section of `smp-server.ini`.

In this Docker Compose configuration, host port `443` is published by the `smp-server` service.

## Docker-Managed Volumes

Two additional persistent volumes are declared in `docker-compose.yml`. They do not appear in the project directory listing because Docker manages their storage separately.

| Volume | Purpose |
|---|---|
| `caddy_data` | Caddy's persistent data, including ACME account information, certificates, and private keys. |
| `caddy_config` | Caddy's internal configuration data, mounted at `/config`. |

The `smp-server` container receives the domain-specific certificate subdirectory from `caddy_data` at `/certificates`. This gives the SMP server access to the Web certificate and private key managed by Caddy.

## References

- [SimpleX SMP server Docker deployment files](https://github.com/simplex-chat/simplexmq/tree/stable/scripts/docker)
- [Hosting your own SMP server](https://simplex.chat/docs/server.html)
