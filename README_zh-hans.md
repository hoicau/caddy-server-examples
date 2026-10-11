# Caddy Server Examples

[English](/README_zh-hans.md) | 简体中文

一组实用的 Caddy 2 配置示例与编译部署指南。包含基础反向代理、Cloudflare CDN 源站集成、外部 ACME / EAB 证书申请、多域名 SNI 分流以及 systemd 托管服务配置。

---

## 目录结构说明

| 目录 / 文件 | 用途 |
| :--- | :--- |
| [`Caddy-basic/`](/Caddy-basic/Caddyfile) | 基础配置：监听 80/443 端口，支持反向代理到本地应用或静态文件服务，自动申请 HTTPS 证书与 HSTS。 |
| [`Caddy-cf-origin/`](/Caddy-cf-origin/Caddyfile) | Cloudflare CDN 回源配置：支持 Cloudflare Origin CA 证书与 Authenticated Origin Pulls (mTLS)，配合 `caddy-cloudflare-ip` 还原真实客户端 IP。 |
| [`Caddy-ext-TLS-cert/`](/Caddy-ext-TLS-cert/Caddyfile) | 外部 CA 凭据配置：通过 EAB (External Account Binding) 向第三方 ACME CA（如 Google Trust Services）申请证书。 |
| [`Caddy-multi-SNI/`](/Caddy-multi-SNI/Caddyfile) | 多域名 / 多路径分流配置：使用 SNI 匹配多域名并将请求分流至不同后端应用，支持路径规则路由。 |
| [`systemd/`](/systemd/caddy.service) | 系统服务集成：包含专用的 `caddy:caddy` 运行用户、数据目录规划以及 `caddy.service` 文件。 |
| [`.woodpecker.yml`](/.woodpecker.yml) | Codeberg / Woodpecker CI 的多架构自动构建和 Release 发布流水线配置。 |
| [`.github/workflows/caddy.yml`](/.github/workflows/caddy.yml) | 原生 GitHub Actions 构建流水线配置。 |

---

## 预构建插件说明

CI 编译的 Caddy 二进制内置了以下扩展插件：
- **[`caddy-cloudflare-ip`](https://github.com/WeidiDeng/caddy-cloudflare-ip)**：自动获取与信任 Cloudflare 代理节点 IP，确保日志与反向代理头获取到真实的访客客户端 IP。
- **[`caddy-dns/cloudflare`](https://github.com/caddy-dns/cloudflare)**：支持 Cloudflare DNS API 验证（DNS-01 质询），便于在内网或未开放 80/443 端口的环境下自动申请并续签泛域名证书。

---

## 快速使用

### 1. 手动使用 xcaddy 自定义构建

如果需要在本地构建包含所需插件的 Caddy，可以安装官方构建工具 [`xcaddy`](https://github.com/caddyserver/xcaddy)：

```bash
# 安装最新版 xcaddy
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest

# 构建包含 Cloudflare 插件的 Caddy
~/go/bin/xcaddy build \
  --with github.com/WeidiDeng/caddy-cloudflare-ip \
  --with github.com/caddy-dns/cloudflare \
  --output ./caddy
```

如需交叉编译到其他架构（如 ARM64）：

```bash
GOOS=linux GOARCH=arm64 ~/go/bin/xcaddy build \
  --with github.com/WeidiDeng/caddy-cloudflare-ip \
  --with github.com/caddy-dns/cloudflare \
  --output ./caddy-arm64
```

---

### 2. 部署与 systemd 服务管理

详细部署流程请参阅 [`systemd/README.md`](/systemd/README.md)。

核心步骤速览：

```bash
# 1. 创建专用系统用户
sudo groupadd --system caddy
sudo useradd --system --gid caddy --no-create-home --shell /usr/sbin/nologin caddy

# 2. 创建持久化存储目录
sudo mkdir -p /caddyserver/log
sudo chown -R caddy:caddy /caddyserver
sudo chmod 755 /caddyserver

# 3. 安装可执行文件
sudo mv caddy /usr/local/bin/
sudo chmod +x /usr/local/bin/caddy

# 4. 配置 systemd 服务
sudo cp systemd/caddy.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now caddy
```

---

## 持续集成 (CI / CD)

本项目同时兼容 GitHub 与 Codeberg (Forgejo) / Woodpecker CI 自动化构建流程。

### Codeberg & Woodpecker CI 配置

流水线文件位于 [`.woodpecker.yml`](/.woodpecker.yml)：

- **多架构矩阵编译**：并行构建 `linux-amd64` 与 `linux-arm64` 架构。
- **自动获取上游最新版本**：自动抓取 Caddy 官方最新 release 标签并驱动 `xcaddy` 构建。
- **自动打包与生成校验码**：输出 `sha256` 校验文件并归档为 `.tar.gz` 压缩包。
- **自动发布 Release**：当在 Codeberg 推送 Git 标签（Tag）时，调用 `woodpeckerci/plugin-release` 自动创建发行版并上传构建产物。

#### 配置 Woodpecker Secret

在使用 Woodpecker CI 自动发布 Release 前，请在 Woodpecker CI 仓库控制台中添加 Secret：

1. 在 Codeberg 用户设置中生成具有仓库写入权限的 **Access Token**（设置 -> 应用 -> 生成新令牌，勾选 `repository` 权限）。
2. 在 Woodpecker Web 界面打开本仓库，进入 **Repository Settings -> Secrets**。
3. 添加名为 `codeberg_token`（或 `CODEBERG_TOKEN`）的 Secret，值为上一步生成的令牌。
4. 推送 Git Tag（如 `v2.11.6`）即可自动触发编译发布：
   ```bash
   git tag v2.11.6
   git push origin v2.11.6
   ```

---

## 许可证

本项目基于 [GPL-3.0 许可证](/LICENSE) 开源。
