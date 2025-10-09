# Nginx Contexts

Contexts define the scope and purpose of configuration directives.

## Main Context

Global settings for the entire Nginx process.

```nginx
# Main context (outside any other context)
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
```

### Common Main Directives
- `user` - User and group for worker processes
- `worker_processes` - Number of worker processes
- `error_log` - Error log file and level
- `pid` - PID file location

## Events Context

Controls how Nginx handles connections.

```nginx
events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}
```

### Common Events Directives
- `worker_connections` - Max connections per worker
- `use` - Event processing method (epoll, kqueue, select)
- `multi_accept` - Accept multiple connections at once

## HTTP Context

Contains all web server configuration.

```nginx
http {
    # Global HTTP settings
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging
    access_log /var/log/nginx/access.log;
    
    # Gzip compression
    gzip on;
    
    # Server blocks go here
    server { }
}
```

### Common HTTP Directives
- `include` - Include other config files
- `default_type` - Default MIME type
- `access_log` - Access log configuration
- `gzip` - Enable compression

## Server Context

Defines a virtual host.

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.html;
}
```

### Common Server Directives
- `listen` - Port and IP to listen on
- `server_name` - Domain names
- `root` - Document root directory
- `index` - Default files to serve

## Location Context

Matches URLs and defines how to handle them.

```nginx
location / {
    try_files $uri $uri/ /index.html;
}

location /api/ {
    proxy_pass http://backend;
}

location ~* \.(css|js|png)$ {
    expires 1y;
}
```

### Location Matching Types

| Type | Syntax | Example | Matches |
|------|--------|---------|---------|
| Exact | `= /path` | `location = /api` | Only `/api` |
| Prefix | `/path` | `location /api/` | `/api/users`, `/api/posts` |
| Regex | `~ pattern` | `location ~ \.php$` | Files ending with `.php` |
| Case-insensitive | `~* pattern` | `location ~* \.(jpg\|png)$` | `.jpg`, `.JPG`, `.png` |
| Longest prefix | `^~ /path` | `location ^~ /static/` | `/static/css/style.css` |

### Location Priority
1. `=` (exact match)
2. `^~` (longest prefix)
3. `~` and `~*` (regex)
4. Longest prefix match

## Upstream Context

Defines groups of backend servers.

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### Upstream Methods
- **Round-robin** (default) - Distribute requests evenly
- **ip_hash** - Sticky sessions by client IP
- **least_conn** - Send to server with fewest connections
- **hash** - Consistent hashing

## Map Context

Creates variables based on other variables.

```nginx
map $http_user_agent $mobile {
    default 0;
    ~*mobile 1;
}

# Usage
if ($mobile) {
    rewrite ^ /mobile$uri;
}
```

## If Context

Conditional processing within location blocks.

```nginx
location / {
    if ($request_method = POST) {
        return 405;
    }
    
    if ($http_user_agent ~* "bot") {
        return 403;
    }
}
```

## Limit Contexts

Rate limiting and access control.

```nginx
# Rate limiting
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

location /api/ {
    limit_req zone=api burst=20;
}

# Connection limiting
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;

location /download/ {
    limit_conn conn_limit_per_ip 1;
}
```

## Context Hierarchy

```
Main
├── Events
└── HTTP
    ├── Upstream
    ├── Map
    ├── Limit zones
    └── Server
        ├── Location
        │   ├── If
        │   ├── Limit req
        │   └── Limit conn
        └── Upstream (in proxy_pass)
```

## Common Context Patterns

### Basic Website
```nginx
http {
    server {
        listen 80;
        server_name example.com;
        root /var/www/html;
        index index.html;
    }
}
```

### Reverse Proxy
```nginx
http {
    upstream backend {
        server 192.168.1.10:8080;
    }
    
    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
}
```

### Multiple Sites
```nginx
http {
    server {
        listen 80;
        server_name site1.com;
        root /var/www/site1;
    }
    
    server {
        listen 80;
        server_name site2.com;
        root /var/www/site2;
    }
}
```

---

[← Back to README](README.md) | [Variables →](NGINX-VARIABLES.md)
