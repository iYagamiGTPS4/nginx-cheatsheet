# Nginx Cheatsheet

Simple and practical Nginx reference guide.

## Quick Start

### Essential Commands
```bash
nginx -t                    # Test config
nginx -s reload             # Reload config
nginx -s stop               # Stop nginx
systemctl start nginx       # Start nginx
systemctl status nginx      # Check status
```

### Basic Config Structure
```nginx
events { }

http {
    server {
        listen 80;
        server_name example.com;
        root /var/www/html;
        index index.html;
    }
}
```

## Common Use Cases

### 1. Static Website
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.html;
}
```

### 2. Reverse Proxy
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

### 3. Load Balancing
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

### 4. SSL/HTTPS
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

### 5. SPA (Single Page App)
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

## More Examples

### 6. Custom Error Pages
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    
    location = /50x.html {
        root /var/www/html;
    }
}
```

### 7. PHP/FastCGI
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php;
    
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 8. WebSocket Proxy
```nginx
server {
    listen 80;
    server_name ws.example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

### 9. File Upload Handling
```nginx
server {
    listen 80;
    server_name upload.example.com;
    
    client_max_body_size 100M;
    client_body_timeout 60s;
    
    location /upload {
        proxy_pass http://backend;
        proxy_request_buffering off;
    }
}
```

### 10. HTTP/2 Configuration
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;
    
    # HTTP/2 optimizations
    http2_max_field_size 4k;
    http2_max_header_size 16k;
    http2_body_preread_size 64k;
}
```

### 11. Docker Deployment
```nginx
# nginx.conf for Docker
events {
    worker_connections 1024;
}

http {
    upstream app {
        server app:3000;
    }
    
    server {
        listen 80;
        server_name localhost;
        
        location / {
            proxy_pass http://app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

### 12. Caching Strategy
```nginx
# Proxy cache
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m;

server {
    listen 80;
    server_name example.com;
    
    location /api/ {
        proxy_cache my_cache;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        proxy_cache_use_stale error timeout updating;
        proxy_pass http://backend;
    }
}

# FastCGI cache
fastcgi_cache_path /var/cache/nginx/fastcgi levels=1:2 keys_zone=php_cache:10m;

server {
    location ~ \.php$ {
        fastcgi_cache php_cache;
        fastcgi_cache_valid 200 60m;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

### 13. Microservices API Gateway
```nginx
upstream auth_service {
    server auth:8080;
}

upstream user_service {
    server user:8080;
}

upstream order_service {
    server order:8080;
}

server {
    listen 80;
    server_name api.example.com;
    
    # Authentication service
    location /auth/ {
        proxy_pass http://auth_service/;
        proxy_set_header X-Service auth;
    }
    
    # User service
    location /users/ {
        proxy_pass http://user_service/;
        proxy_set_header X-Service user;
    }
    
    # Order service
    location /orders/ {
        proxy_pass http://order_service/;
        proxy_set_header X-Service order;
    }
}
```

### 14. DDoS Protection
```nginx
# Rate limiting zones
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=general:10m rate=1r/s;

# Connection limiting
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;

server {
    listen 80;
    server_name example.com;
    
    # General rate limiting
    limit_req zone=general burst=5 nodelay;
    limit_conn conn_limit_per_ip 10;
    
    # API endpoints
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
    }
    
    # Login endpoint
    location /login {
        limit_req zone=login burst=3 nodelay;
        proxy_pass http://backend;
    }
}
```

### 15. Health Checks and Monitoring
```nginx
# Health check endpoint
location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
}

# Status page
location /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
}

# Metrics endpoint
location /metrics {
    access_log off;
    return 200 "nginx_up 1\n";
    add_header Content-Type text/plain;
}
```

## Reference Files

- **[Contexts](NGINX-CONTEXTS.md)** - Main, Events, HTTP, Server, Location
- **[Variables](NGINX-VARIABLES.md)** - Common variables and their usage
- **[Directives](NGINX-DIRECTIVES.md)** - Essential directives organized by purpose

## Performance Optimization

### Quick Performance Tips
```nginx
# Worker optimization
worker_processes auto;
worker_connections 1024;
worker_rlimit_nofile 65535;

# HTTP optimizations
sendfile on;
tcp_nopush on;
tcp_nodelay on;
keepalive_timeout 65;
keepalive_requests 100;

# Gzip compression
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_types text/plain text/css application/json application/javascript;

# Static file caching
location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

### Connection Pooling
```nginx
upstream backend {
    server 192.168.1.10:8080 max_conns=100;
    server 192.168.1.11:8080 max_conns=100;
    keepalive 32;
    keepalive_requests 100;
    keepalive_timeout 60s;
}
```

## Security Best Practices

### Security Headers
```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'" always;
```

### SSL/TLS Hardening
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;
    
    # SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_stapling on;
    ssl_stapling_verify on;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
}
```

### Access Control
```nginx
# Block common attacks
location ~* /(wp-admin|wp-login|xmlrpc) {
    return 403;
}

# Block suspicious requests
if ($request_uri ~* "(union|select|insert|delete|drop|create)") {
    return 403;
}

# Block bad user agents
if ($http_user_agent ~* "(bot|crawler|spider|scraper)") {
    return 403;
}

# IP whitelist for admin
location /admin {
    allow 192.168.1.0/24;
    allow 10.0.0.0/8;
    deny all;
}
```

## Troubleshooting

### Common Issues and Solutions

#### 1. 502 Bad Gateway
```bash
# Check if backend is running
curl http://localhost:8080/health

# Check nginx error logs
tail -f /var/log/nginx/error.log

# Test configuration
nginx -t
```

#### 2. 504 Gateway Timeout
```nginx
# Increase timeouts
proxy_connect_timeout 60s;
proxy_send_timeout 60s;
proxy_read_timeout 60s;
```

#### 3. SSL Certificate Issues
```bash
# Check certificate validity
openssl x509 -in /path/to/cert.crt -text -noout

# Test SSL configuration
openssl s_client -connect example.com:443 -servername example.com
```

#### 4. Performance Issues
```bash
# Check worker processes
ps aux | grep nginx

# Monitor connections
ss -tuln | grep :80
ss -tuln | grep :443

# Check memory usage
free -h
```

### Debugging Commands
```bash
# Test config
nginx -t

# Check status
systemctl status nginx

# View logs
tail -f /var/log/nginx/error.log
tail -f /var/log/nginx/access.log

# Reload config
nginx -s reload

# Graceful restart
nginx -s quit && nginx

# Check open files
lsof -p $(pgrep nginx)

# Monitor real-time connections
watch -n 1 'ss -tuln | grep :80'
```

### Log Analysis
```bash
# Top IPs
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10

# Top pages
awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10

# Error analysis
grep "error" /var/log/nginx/error.log | tail -20

# Response time analysis
awk '{print $NF}' /var/log/nginx/access.log | sort -n | tail -10
```

---

**Need more details?** Check the reference files above for contexts, variables, and directives.
