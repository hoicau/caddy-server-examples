适用于 Cloudflare CDN 后的源服务器。使用 Cloudflare Origin Certificate，需自行获取和配置 Origin Certificates，启用Authenticated Origin Pulls 并配置 SSL/TLS 模式为 Full (Strict)。

默认监听 `50000` 端口，需自行配置 Origin Rule 以重写回源端口号。可以获取到 Client IP。

使用 Cloudflare ZeroTrust Tunnel 以取得更高的安全性，即使这意味着牺牲相当一部分性能。
