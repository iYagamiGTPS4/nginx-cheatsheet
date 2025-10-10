# Nginx Contexts

Contexts define the scope and purpose of configuration directives.

## Main Context

Global settings for the entire Nginx process.

```nginx
# Main context - global settings for entire nginx process
user nginx;                              # Run worker processes as nginx user
worker_processes auto;                   # Auto-detect optimal number of worker processes
error_log /var/log/nginx/error.log;     # Log errors to specified file
pid /run/nginx.pid;                     # Store master process PID in this file
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
    worker_connections 1024;           # Maximum connections per worker process
    use epoll;                         # Use epoll event method (Linux)
    multi_accept on;                   # Accept multiple connections at once
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
    # Global HTTP settings for all virtual hosts
    include /etc/nginx/mime.types;        # Include MIME type definitions
    default_type application/octet-stream; # Default MIME type for unknown files
    
    # Logging configuration
    access_log /var/log/nginx/access.log; # Log all HTTP requests
    
    # Gzip compression for bandwidth optimization
    gzip on;                              # Enable gzip compression
    
    # Server blocks define virtual hosts
    server { }                            # Virtual host configuration goes here
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
    listen 80;                           # Listen on port 80 for HTTP requests
    server_name example.com;             # Match requests for this domain
    root /var/www/html;                 # Document root directory for files
    index index.html;                   # Default file to serve for directory requests
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
    try_files $uri $uri/ /index.html;   # Try file, then directory, then fallback
}

location /api/ {
    proxy_pass http://backend;          # Forward API requests to backend server
}

location ~* \.(css|js|png)$ {
    expires 1y;                        # Cache static files for 1 year
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

## Geo Context

Geographic routing and location-based configuration.

```nginx
# Define geographic zones
geo $country {
    default US;
    192.168.1.0/24 CA;
    10.0.0.0/8 US;
    172.16.0.0/12 EU;
}

# Usage in server block
server {
    listen 80;
    server_name example.com;
    
    # Redirect based on country
    if ($country = "EU") {
        return 301 https://eu.example.com$request_uri;
    }
    
    # Different content for different regions
    location / {
        if ($country = "CA") {
            rewrite ^ /canada$uri;
        }
        try_files $uri $uri/ /index.html;
    }
}
```

## Split Clients Context

A/B testing and feature flags.

```nginx
# A/B testing
split_clients "${remote_addr}${http_user_agent}" $variant {
    50% "A";
    50% "B";
}

# Usage
server {
    listen 80;
    server_name example.com;
    
    location / {
        if ($variant = "A") {
            rewrite ^ /version-a$uri;
        }
        if ($variant = "B") {
            rewrite ^ /version-b$uri;
        }
        try_files $uri $uri/ /index.html;
    }
}

# Feature flags
split_clients "${remote_addr}" $new_feature {
    10% "enabled";
    90% "disabled";
}
```

## Cache Contexts

Caching configuration and management.

### Proxy Cache
```nginx
# Proxy cache configuration
proxy_cache_path /var/cache/nginx/proxy 
    levels=1:2 
    keys_zone=my_cache:10m 
    max_size=1g 
    inactive=60m 
    use_temp_path=off;

# Usage in server block
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
```

### FastCGI Cache
```nginx
# FastCGI cache configuration
fastcgi_cache_path /var/cache/nginx/fastcgi 
    levels=1:2 
    keys_zone=php_cache:10m 
    max_size=1g 
    inactive=60m;

# Usage
server {
    listen 80;
    server_name example.com;
    
    location ~ \.php$ {
        fastcgi_cache php_cache;
        fastcgi_cache_valid 200 60m;
        fastcgi_cache_valid 404 1m;
        fastcgi_cache_bypass $no_cache;
        fastcgi_no_cache $no_cache;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

## Stream Context

TCP/UDP load balancing and proxying.

```nginx
# TCP load balancing
stream {
    upstream backend_tcp {
        server 192.168.1.10:3306;
        server 192.168.1.11:3306;
        server 192.168.1.12:3306;
    }
    
    server {
        listen 3306;
        proxy_pass backend_tcp;
        proxy_timeout 1s;
        proxy_responses 1;
    }
}

# UDP load balancing
stream {
    upstream backend_udp {
        server 192.168.1.10:53;
        server 192.168.1.11:53;
    }
    
    server {
        listen 53 udp;
        proxy_pass backend_udp;
        proxy_timeout 1s;
        proxy_responses 1;
    }
}
```

## Performance Context Patterns

### High-Performance Configuration
```nginx
# Main context optimizations
worker_processes auto;
worker_rlimit_nofile 65535;
worker_cpu_affinity auto;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
    accept_mutex off;
}

http {
    # Performance optimizations
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 100;
    
    # Buffer optimizations
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;
}
```

### Connection Pooling
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Connection pooling
    keepalive 32;
    keepalive_requests 100;
    keepalive_timeout 60s;
    
    # Health checks
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
}
```

## Real-World Multi-Context Patterns

### Microservices Architecture
```nginx
# Main context
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;

events {
    worker_connections 1024;
}

http {
    # Rate limiting zones
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
    
    # Connection limiting
    limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
    
    # Upstream services
    upstream auth_service {
        server auth:8080;
        keepalive 16;
    }
    
    upstream user_service {
        server user:8080;
        keepalive 16;
    }
    
    upstream order_service {
        server order:8080;
        keepalive 16;
    }
    
    # Cache configuration
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=api_cache:10m max_size=1g;
    
    # Main server
    server {
        listen 80;
        server_name api.example.com;
        
        # Rate limiting
        limit_req zone=api burst=20 nodelay;
        limit_conn conn_limit_per_ip 10;
        
        # Authentication service
        location /auth/ {
            limit_req zone=login burst=5 nodelay;
            proxy_pass http://auth_service/;
            proxy_cache api_cache;
            proxy_cache_valid 200 5m;
        }
        
        # User service
        location /users/ {
            proxy_pass http://user_service/;
            proxy_cache api_cache;
            proxy_cache_valid 200 10m;
        }
        
        # Order service
        location /orders/ {
            proxy_pass http://order_service/;
            proxy_cache api_cache;
            proxy_cache_valid 200 2m;
        }
    }
}
```

### CDN Configuration
```nginx
# Geographic routing
geo $country {
    default US;
    192.168.1.0/24 CA;
    10.0.0.0/8 US;
}

# A/B testing
split_clients "${remote_addr}${http_user_agent}" $cdn_variant {
    50% "cloudflare";
    50% "aws";
}

# Cache zones
proxy_cache_path /var/cache/nginx/cdn levels=1:2 keys_zone=cdn_cache:100m max_size=10g;

server {
    listen 80;
    server_name cdn.example.com;
    
    # Geographic content
    location / {
        if ($country = "CA") {
            rewrite ^ /canada$uri;
        }
        try_files $uri $uri/ @cdn;
    }
    
    # CDN fallback
    location @cdn {
        if ($cdn_variant = "cloudflare") {
            proxy_pass http://cloudflare_backend;
        }
        if ($cdn_variant = "aws") {
            proxy_pass http://aws_backend;
        }
        proxy_cache cdn_cache;
        proxy_cache_valid 200 1h;
    }
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
