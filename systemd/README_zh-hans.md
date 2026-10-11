使用 `systemctl` 控制 caddy 服务。

```
# 新建 caddy:caddy 用户（组）
groupadd --system caddy
useradd --system \
--gid caddy \
--no-create-home \
--shell /usr/sbin/nologin \
caddy

# 新建 data 文件夹并赋权
mkdir /caddyserver
chown caddy:caddy /caddyserver
chmod 755 /caddyserver

# 写入 systemctl
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

# 重载 systemctl 并启用服务
systemctl daemon-reload
systemctl enable --now caddy
```
