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

## Reference Files

- **[Contexts](NGINX-CONTEXTS.md)** - Main, Events, HTTP, Server, Location
- **[Variables](NGINX-VARIABLES.md)** - Common variables and their usage
- **[Directives](NGINX-DIRECTIVES.md)** - Essential directives organized by purpose

## Quick Troubleshooting

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
```

---

**Need more details?** Check the reference files above for contexts, variables, and directives.
