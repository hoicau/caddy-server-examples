Listens on ports `80` and `443` and reverse proxies requests to a local application. Automatically obtains TLS certificates and redirects HTTP traffic to HTTPS by default.

### Static File Server

If you want to use Caddy as a static file server:

```caddyfile
# Find:
reverse_proxy 127.0.0.1:12345 # Reverse proxy to local application

# Replace with:
root * /home/user/www # Path to your website files
file_server
```

### Reverse Proxy to an External Website

If you need to reverse proxy requests to another website/host:

```caddyfile
# Find:
reverse_proxy 127.0.0.1:12345 # Reverse proxy to local application

# Replace with:
reverse_proxy https://target.example.com {
    header_up Host {upstream_hostport}
}
```