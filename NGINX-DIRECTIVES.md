# Nginx Directives

Essential directives organized by purpose and context.

## Server Configuration

### Basic Server Directives
```nginx
listen 80;                    # Listen on port 80
listen 443 ssl;              # Listen on port 443 with SSL
listen 127.0.0.1:8080;       # Listen on specific IP and port
listen [::]:80;              # IPv6 support

server_name example.com;     # Single domain
server_name example.com www.example.com;  # Multiple domains
server_name *.example.com;   # Wildcard subdomain
server_name _;               # Catch-all server

root /var/www/html;         # Document root directory
index index.html index.htm;  # Default files to serve
```

### Server Status and Logging
```nginx
access_log /var/log/nginx/access.log;     # Access log
access_log off;                          # Disable access log
error_log /var/log/nginx/error.log warn; # Error log with level

return 404;                              # Return status code
return 301 https://$server_name$request_uri;  # Redirect

# Custom error pages
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;
error_page 404 =200 /index.html;        # Return 200 instead of 404
```

### Server Flags
```nginx
listen 80 default_server;                # Default server for port
listen 443 ssl default_server;          # Default SSL server
```

## Location Matching

### Location Types
```nginx
# Exact match (highest priority)
location = /exact {
    # Only matches exactly "/exact"
}

# Prefix match
location /api/ {
    # Matches "/api/users", "/api/posts"
}

# Regex match (case sensitive)
location ~ \.php$ {
    # Matches files ending with .php
}

# Regex match (case insensitive)
location ~* \.(css|js|png|jpg)$ {
    # Static files
}

# Longest prefix match
location ^~ /static/ {
    # Matches "/static/css/style.css"
}

# Named location
location @fallback {
    # Used with try_files
}
```

### File Handling
```nginx
try_files $uri $uri/ /index.html;        # File fallback chain
try_files $uri $uri/ @fallback;          # Named location fallback

autoindex on;                             # Directory listing
autoindex_exact_size off;                # Human readable file sizes
autoindex_localtime on;                  # Local time format

# Path aliases
alias /path/to/files/;                   # Alias (different from root)
root /var/www/html;                      # Document root

# Internal locations
location @internal {
    internal;                            # Only internal redirects
}
```

## Proxy Configuration

### Basic Proxy
```nginx
proxy_pass http://backend;                # Proxy to backend
proxy_pass http://192.168.1.10:8080;     # Direct proxy
proxy_pass http://upstream;              # Upstream proxy
```

### Proxy Headers
```nginx
proxy_set_header Host $host;             # Preserve host header
proxy_set_header X-Real-IP $remote_addr; # Real client IP
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host $server_name;
```

### Proxy Timeouts
```nginx
proxy_connect_timeout 60s;               # Connection timeout
proxy_send_timeout 60s;                 # Send timeout
proxy_read_timeout 60s;                 # Read timeout
```

### Proxy Buffering
```nginx
proxy_buffering on;                     # Enable buffering
proxy_buffer_size 4k;                   # Buffer size
proxy_buffers 8 4k;                     # Number and size of buffers
proxy_busy_buffers_size 8k;             # Busy buffer size
```

### Proxy Redirects and Cookies
```nginx
proxy_redirect off;                     # Disable redirect rewriting
proxy_redirect http://localhost:8080/ /; # Rewrite redirects
proxy_cookie_domain localhost example.com; # Rewrite cookie domain
proxy_cookie_path /one/ /;              # Rewrite cookie path
```

### FastCGI and uWSGI
```nginx
# FastCGI (PHP)
fastcgi_pass 127.0.0.1:9000;           # FastCGI server
fastcgi_index index.php;                # FastCGI index file
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

# uWSGI (Python)
uwsgi_pass 127.0.0.1:8000;              # uWSGI server
uwsgi_param SCRIPT_NAME $fastcgi_script_name;
```

## Upstream Configuration

### Basic Upstream
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### Upstream Methods
```nginx
upstream backend {
    # Round-robin (default)
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Weighted round-robin
    server 192.168.1.10:8080 weight=3;
    server 192.168.1.11:8080 weight=1;
    
    # Health checks
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    
    # Backup server
    server 192.168.1.12:8080 backup;
}
```

### Load Balancing Methods
```nginx
upstream backend {
    # ip_hash;          # Sticky sessions by IP
    # least_conn;        # Least connections
    # hash $request_uri; # Consistent hashing
    
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

## SSL/TLS Configuration

### Basic SSL
```nginx
ssl_certificate /path/to/cert.crt;       # SSL certificate
ssl_certificate_key /path/to/key.key;    # SSL private key
ssl_protocols TLSv1.2 TLSv1.3;          # SSL protocols
ssl_ciphers HIGH:!aNULL:!MD5;           # SSL ciphers
```

### SSL Optimization
```nginx
ssl_session_cache shared:SSL:10m;       # SSL session cache
ssl_session_timeout 10m;                 # Session timeout
ssl_stapling on;                         # OCSP stapling
ssl_stapling_verify on;                  # OCSP verification
```

## Headers and CORS

### Security Headers
```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000" always;
```

### CORS Headers
```nginx
add_header Access-Control-Allow-Origin "*" always;
add_header Access-Control-Allow-Methods "GET, POST, OPTIONS" always;
add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;
```

## URL Rewriting

### Basic Rewrites
```nginx
rewrite ^/old/(.*)$ /new/$1 permanent;   # Permanent redirect
rewrite ^/old/(.*)$ /new/$1 redirect;    # Temporary redirect
rewrite ^/old/(.*)$ /new/$1 last;        # Internal redirect
rewrite ^/old/(.*)$ /new/$1 break;       # Stop processing
```

### Common Rewrite Patterns
```nginx
# Remove trailing slash
rewrite ^/(.*)/$ /$1 permanent;

# Add trailing slash
rewrite ^/(.*[^/])$ /$1/ permanent;

# Remove www
if ($host = 'www.example.com') {
    return 301 https://example.com$request_uri;
}

# Force HTTPS
if ($scheme != "https") {
    return 301 https://$server_name$request_uri;
}
```

## Caching and Compression

### Static File Caching
```nginx
location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;                           # Cache for 1 year
    add_header Cache-Control "public, immutable";
    access_log off;                       # Don't log static files
}
```

### Gzip Compression
```nginx
gzip on;                                 # Enable gzip
gzip_vary on;                            # Add Vary header
gzip_min_length 1024;                    # Minimum file size
gzip_comp_level 6;                      # Compression level
gzip_types text/plain text/css application/json application/javascript;
```

## Security Directives

### Rate Limiting
```nginx
# Define rate limit zones
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

# Apply rate limiting
location /api/ {
    limit_req zone=api burst=20 nodelay;
}

location /login {
    limit_req zone=login burst=5 nodelay;
}
```

### Access Control
```nginx
# IP whitelist
location /admin {
    allow 192.168.1.0/24;
    allow 10.0.0.0/8;
    deny all;
}

# Basic authentication
location /secure {
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
}

# Referer validation
valid_referers none blocked server_names *.example.com;
if ($invalid_referer) {
    return 403;
}

# Access control logic
location /api/ {
    satisfy any;                          # Any condition (allow OR auth)
    allow 192.168.1.0/24;
    auth_basic "API Access";
}
```

### Security Blocks
```nginx
# Block common attacks
location ~* /(wp-admin|wp-login|xmlrpc) {
    return 403;
}

# Block suspicious requests
if ($request_uri ~* "(union|select|insert|delete)") {
    return 403;
}
```

## Performance Directives

### Worker Configuration
```nginx
worker_processes auto;                   # Number of worker processes
worker_connections 1024;                  # Connections per worker
worker_rlimit_nofile 65535;              # File descriptor limit
```

### HTTP Optimization
```nginx
sendfile on;                             # Efficient file serving
tcp_nopush on;                           # Optimize TCP packets
tcp_nodelay on;                          # Disable Nagle's algorithm
keepalive_timeout 65;                    # Keep-alive timeout
keepalive_requests 100;                  # Requests per connection
```

### Buffer Sizes
```nginx
client_body_buffer_size 128k;            # Client body buffer
client_max_body_size 10m;                # Max request body size
client_header_buffer_size 1k;            # Client header buffer
large_client_header_buffers 4 4k;         # Large header buffers
```

### Timeouts
```nginx
client_body_timeout 12s;                 # Client body timeout
client_header_timeout 12s;               # Client header timeout
send_timeout 10s;                        # Response send timeout
```

### MIME Types
```nginx
types {                                  # Define MIME types
    text/html html htm shtml;
    text/css css;
    application/javascript js;
    image/png png;
    image/jpeg jpg jpeg;
}
default_type application/octet-stream;    # Default MIME type
```

## Logging Directives

### Log Formats
```nginx
# Custom log format
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

# JSON log format
log_format json_combined escape=json
    '{"time":"$time_iso8601","remote_addr":"$remote_addr",'
    '"request":"$request","status":"$status","body_bytes_sent":"$body_bytes_sent"}';
```

### Log Configuration
```nginx
access_log /var/log/nginx/access.log main;  # Access log
access_log off;                            # Disable access log
error_log /var/log/nginx/error.log warn;   # Error log
```

## Common Directive Patterns

### Static Website
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### Reverse Proxy
```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Load Balancer
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

### SSL Termination
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;
    
    location / {
        proxy_pass http://backend;
    }
}
```

## Directive Categories

### Essential Directives
| Directive | Context | Purpose |
|-----------|---------|---------|
| `listen` | server | Port and IP binding |
| `server_name` | server | Domain matching |
| `root` | server, location | Document root |
| `index` | server, location | Default files |
| `location` | server | URL matching |
| `proxy_pass` | location | Reverse proxy |
| `try_files` | location | File fallback |

### Security Directives
| Directive | Context | Purpose |
|-----------|---------|---------|
| `allow` | location | IP whitelist |
| `deny` | location | IP blacklist |
| `auth_basic` | location | Basic authentication |
| `limit_req` | location | Rate limiting |
| `add_header` | server, location | Security headers |

### Performance Directives
| Directive | Context | Purpose |
|-----------|---------|---------|
| `worker_processes` | main | Worker count |
| `worker_connections` | events | Connections per worker |
| `gzip` | http | Compression |
| `expires` | location | Caching |
| `sendfile` | http | Efficient file serving |

---

[← Variables](NGINX-VARIABLES.md) | [Back to README](README.md)
