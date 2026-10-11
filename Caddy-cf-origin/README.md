Suitable for origin servers behind Cloudflare CDN. Uses Cloudflare Origin Certificates. You need to obtain and configure the Origin Certificate, enable **Authenticated Origin Pulls (AOP)**, and set Cloudflare's SSL/TLS encryption mode to **Full (Strict)**.

- **Port configuration**: Listens on port `50000` by default. You need to configure Cloudflare Origin Rules to rewrite the origin port to `50000`.
- **Client IP**: Preserves the original visitor client IP via the `caddy-cloudflare-ip` plugin (`trusted_proxies cloudflare`).
- **Alternative**: You may also consider Cloudflare Zero Trust Tunnels (cloudflared) for higher origin isolation and security, albeit with potential throughput/latency trade-offs.
