# Nginx Quick Reference

Quick cheatsheet for common Nginx configurations and commands.

## Essential Commands

```bash
# Configuration
nginx -t                    # Test configuration for errors
nginx -s reload             # Reload configuration without stopping
nginx -s stop               # Stop nginx gracefully
nginx -s quit               # Quit nginx (same as stop)

# Service management
systemctl start nginx       # Start nginx service
systemctl stop nginx        # Stop nginx service
systemctl restart nginx     # Restart nginx service
systemctl status nginx      # Check nginx status
systemctl enable nginx      # Enable nginx on boot

# Logs
tail -f /var/log/nginx/access.log   # Watch access log
tail -f /var/log/nginx/error.log    # Watch error log
```

## Common Configurations

### Basic Static Site
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.html;
}
```

### Reverse Proxy
```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### SSL/HTTPS
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;
    
    root /var/www/html;
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
```

### Load Balancing
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
    }
}
```

### Rate Limiting
```nginx
# Define rate limit zone
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

server {
    listen 80;
    server_name example.com;
    
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
    }
}
```

### Caching
```nginx
# Proxy cache
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m;

server {
    listen 80;
    server_name example.com;
    
    location /api/ {
        proxy_cache my_cache;
        proxy_cache_valid 200 10m;
        proxy_pass http://backend;
    }
}
```

## Performance Directives

```nginx
# Worker optimization
worker_processes auto;                 # Auto-detect worker count
worker_connections 1024;               # Max connections per worker

# HTTP optimizations
sendfile on;                           # Efficient file serving
tcp_nopush on;                         # Optimize TCP packets
tcp_nodelay on;                        # Disable Nagle's algorithm
keepalive_timeout 65;                  # Keep connections alive

# Gzip compression
gzip on;                               # Enable compression
gzip_types text/plain text/css application/json;
```

## Security Headers

```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000" always;
```

## Common Variables

```nginx
$remote_addr          # Client IP address
$request_uri          # Full URI with query string
$uri                  # URI without query string
$request_method       # HTTP method (GET, POST, etc.)
$status               # HTTP status code
$server_name          # Server name
$host                 # Host header
$http_user_agent      # User-Agent header
```

## Troubleshooting

```bash
# Test configuration
nginx -t

# Check nginx processes
ps aux | grep nginx

# Check open ports
ss -tuln | grep :80
ss -tuln | grep :443

# Monitor connections
watch -n 1 'ss -tuln | grep :80'

# Check error logs
grep "error" /var/log/nginx/error.log | tail -20

# Test SSL
openssl s_client -connect example.com:443
```

## Quick Patterns

### SPA (Single Page App)
```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### Static Files with Cache
```nginx
location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

### Block Common Attacks
```nginx
location ~* /(wp-admin|wp-login|xmlrpc) {
    return 403;
}
```

### Health Check
```nginx
location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
}
```

---

**Need more details?** Check the main documentation files:
- [README.md](README.md) - Complete examples and use cases
- [NGINX-CONTEXTS.md](NGINX-CONTEXTS.md) - Context explanations
- [NGINX-DIRECTIVES.md](NGINX-DIRECTIVES.md) - All directives
- [NGINX-VARIABLES.md](NGINX-VARIABLES.md) - All variables
