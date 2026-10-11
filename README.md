# Caddy Server Examples

English | [简体中文](/README_zh-hans.md)

A collection of practical Caddy 2 configuration examples, deployment patterns, and build recipes. Includes configurations for basic reverse proxying, Cloudflare CDN origin integration, external ACME / EAB certificate management, multi-domain SNI routing, and systemd service management.

---

## Directory Overview

| Directory / File | Description |
| :--- | :--- |
| [`Caddy-basic/`](/Caddy-basic/Caddyfile) | Basic configuration: listens on ports 80/443, reverse proxies to local applications or serves static files, with automated TLS certificates and HSTS. |
| [`Caddy-cf-origin/`](/Caddy-cf-origin/Caddyfile) | Cloudflare CDN origin configuration: uses Cloudflare Origin CA certificates and Authenticated Origin Pulls (mTLS), integrated with `caddy-cloudflare-ip` to preserve true client IPs. |
| [`Caddy-ext-TLS-cert/`](/Caddy-ext-TLS-cert/Caddyfile) | External CA binding: obtains TLS certificates via ACME EAB (External Account Binding) from third-party CAs (e.g., Google Trust Services). |
| [`Caddy-multi-SNI/`](/Caddy-multi-SNI/Caddyfile) | Multi-domain / multi-path routing: uses SNI matching to route requests across different domains and paths to multiple backend services. |
| [`systemd/`](/systemd/caddy.service) | Service management: dedicated `caddy:caddy` system user, structured runtime data directories, and systemd service unit. |
| [`.woodpecker.yml`](/.woodpecker.yml) | Multi-arch build and automated release workflow tailored for Codeberg / Woodpecker CI. |
| [`.github/workflows/caddy.yml`](/.github/workflows/caddy.yml) | GitHub Actions CI workflow. |

---

## Bundled Plugins

The CI pipeline compiles Caddy with the following modules using `xcaddy`:
- **[`caddy-cloudflare-ip`](https://github.com/WeidiDeng/caddy-cloudflare-ip)**: Automatically resolves and trusts Cloudflare reverse proxy IP ranges so Caddy logs and proxied requests carry genuine client IPs.
- **[`caddy-dns/cloudflare`](https://github.com/caddy-dns/cloudflare)**: Enables Cloudflare DNS API validation (DNS-01 challenge) for issuing wildcard and internal domain TLS certificates without exposing HTTP ports.

---

## Getting Started

### 1. Manual Build with xcaddy

To manually build Caddy with the required plugins on your machine:

```bash
# Install the latest xcaddy
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest

# Build Caddy with Cloudflare plugins
~/go/bin/xcaddy build \
  --with github.com/WeidiDeng/caddy-cloudflare-ip \
  --with github.com/caddy-dns/cloudflare \
  --output ./caddy
```

Cross-compiling for other architectures (e.g. Linux ARM64):

```bash
GOOS=linux GOARCH=arm64 ~/go/bin/xcaddy build \
  --with github.com/WeidiDeng/caddy-cloudflare-ip \
  --with github.com/caddy-dns/cloudflare \
  --output ./caddy-arm64
```

---

### 2. Deployment & systemd Service

For a detailed deployment setup, see [`systemd/README.md`](/systemd/README.md).

Quick setup summary:

```bash
# 1. Create system user and group
sudo groupadd --system caddy
sudo useradd --system --gid caddy --no-create-home --shell /usr/sbin/nologin caddy

# 2. Create data and logging directories
sudo mkdir -p /caddyserver/log
sudo chown -R caddy:caddy /caddyserver
sudo chmod 755 /caddyserver

# 3. Install binary
sudo mv caddy /usr/local/bin/
sudo chmod +x /usr/local/bin/caddy

# 4. Install and start the service
sudo cp systemd/caddy.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now caddy
```

---

## Continuous Integration (CI / CD)

This repository includes CI workflows for both GitHub Actions and Codeberg (Forgejo) / Woodpecker CI.

### Codeberg & Woodpecker CI

The pipeline is defined in [`.woodpecker.yml`](/.woodpecker.yml):

- **Multi-architecture build matrix**: Builds both `linux-amd64` and `linux-arm64` binaries in parallel.
- **Automated version resolution**: Automatically checks the latest upstream release tag from Caddy and builds using `xcaddy`.
- **Packaging and checksums**: Generates SHA256 checksums and creates `.tar.gz` release archives.
- **Automated releases**: Whenever a Git tag is pushed to Codeberg, `woodpeckerci/plugin-release` creates a release and uploads the archives.

#### Setting up Secrets in Woodpecker CI

To allow Woodpecker CI to publish releases to Codeberg:

1. In Codeberg, generate a Personal Access Token with repository write permissions under **Settings -> Applications -> Generate New Token**.
2. In the Woodpecker CI web UI, open this repository and navigate to **Repository Settings -> Secrets**.
3. Create a secret named `codeberg_token` with your generated token as the value.
4. Push a Git tag (e.g. `v2.11.6`) to trigger the release pipeline:
   ```bash
   git tag v2.11.6
   git push origin v2.11.6
   ```

---

## License

This project is licensed under the [GNU General Public License v3.0](/LICENSE).
