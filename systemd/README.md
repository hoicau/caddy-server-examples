# Caddy systemd Service Management

Manage the Caddy web server service using `systemctl`.

### 1. Create Dedicated User and Group

```bash
# Create caddy:caddy system user and group
sudo groupadd --system caddy
sudo useradd --system \
  --gid caddy \
  --no-create-home \
  --shell /usr/sbin/nologin \
  caddy
```

### 2. Prepare Data Directory

```bash
# Create data/log directory and set ownership
sudo mkdir -p /caddyserver/log
sudo chown -R caddy:caddy /caddyserver
sudo chmod 755 /caddyserver
```

### 3. Install systemd Service Unit

Place the provided [`caddy.service`](file:///home/victorchan/Documents/git/caddy-server-examples/systemd/caddy.service) file into `/etc/systemd/system/`:

```bash
sudo cp caddy.service /etc/systemd/system/caddy.service
```

Or write the unit file directly:

```bash
sudo tee /etc/systemd/system/caddy.service > /dev/null <<EOF
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
Type=notify
User=caddy
Group=caddy
ExecStart=/usr/local/bin/caddy run --environ --config /caddyserver/Caddyfile
ExecReload=/usr/local/bin/caddy reload --config /caddyserver/Caddyfile --force
TimeoutStopSec=5s
LimitNOFILE=1048576
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
EOF
```

### 4. Enable and Start the Service

```bash
# Reload systemd daemon
sudo systemctl daemon-reload

# Enable and start Caddy immediately
sudo systemctl enable --now caddy

# Check service status
sudo systemctl status caddy
```
