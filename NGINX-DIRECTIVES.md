# Nginx Directives

Essential directives organized by purpose and context.

## Server Configuration

### Basic Server Directives
```nginx
listen 80;                    # Listen on HTTP port 80 for incoming connections (server)
listen 443 ssl;              # Listen on HTTPS port 443 with SSL encryption enabled (server)
listen 127.0.0.1:8080;       # Listen on specific IP address and port for local connections (server)
listen [::]:80;              # IPv6 support for port 80, accepts connections from IPv6 clients (server)

server_name example.com;     # Match requests for single domain name (server)
server_name example.com www.example.com;  # Match requests for multiple domain names (server)
server_name *.example.com;   # Match all subdomains using wildcard pattern (server)
server_name _;               # Catch-all server block for unmatched requests (server)

root /var/www/html;         # Set document root directory where files are served from (server, location)
index index.html index.htm;  # Define default files to serve when directory is requested (server, location)
```

### Server Status and Logging
```nginx
access_log /var/log/nginx/access.log;     # Log all HTTP requests to specified file (http, server, location)
access_log off;                          # Disable access logging completely (http, server, location)
error_log /var/log/nginx/error.log warn; # Log errors with warning level and above (main, http, server, location)

return 404;                              # Return HTTP 404 Not Found status code immediately (server, location)
return 301 https://$server_name$request_uri;  # Redirect to HTTPS with 301 permanent redirect (server, location)

# Custom error pages for better user experience
error_page 404 /404.html;                # Show custom 404 error page when file not found (server, location)
error_page 500 502 503 504 /50x.html;   # Show custom error page for server errors (server, location)
error_page 404 =200 /index.html;        # Return 200 status with index.html content for 404s (server, location)
```

### Server Flags
```nginx
listen 80 default_server;                # Set as default server for port 80 when no other server matches (server)
listen 443 ssl default_server;          # Set as default SSL server for port 443 when no other server matches (server)
```

## Location Matching

### Location Types
```nginx
# Exact match (highest priority) - matches only exactly "/exact" path
location = /exact {
    # Only matches exactly "/exact" path, highest priority (location)
}

# Prefix match for API routes - matches any path starting with "/api/"
location /api/ {
    # Matches "/api/users", "/api/posts", "/api/v1/data" etc. (location)
}

# Regex match (case sensitive) - matches files ending with .php
location ~ \.php$ {
    # Matches files ending with .php using case-sensitive regex (location)
}

# Regex match (case insensitive) for static files - matches static file extensions
location ~* \.(css|js|png|jpg)$ {
    # Matches static files with these extensions using case-insensitive regex (location)
}

# Longest prefix match for static content - higher priority than regex
location ^~ /static/ {
    # Matches "/static/css/style.css" with higher priority than regex matches (location)
}

# Named location for fallback handling - used internally by try_files
location @fallback {
    # Used with try_files directive for internal redirects (location)
}
```

### File Handling
```nginx
try_files $uri $uri/ /index.html;        # Try requested file, then directory, then fallback to index.html (location)
try_files $uri $uri/ @fallback;          # Try requested file, then directory, then use named location fallback (location)

autoindex on;                             # Enable directory listing when no index file found (location)
autoindex_exact_size off;                # Show human readable file sizes instead of bytes (location)
autoindex_localtime on;                  # Use local time format for file timestamps (location)

# Path aliases for different document roots
alias /path/to/files/;                   # Alias path to different directory (replaces location path) (location)
root /var/www/html;                      # Set document root directory for file serving (server, location)

# Internal locations for security
location @internal {
    internal;                            # Only allow internal redirects, not direct access (location)
}
```

## Proxy Configuration

### Basic Proxy
```nginx
proxy_pass http://backend;                # Forward requests to backend upstream server group (location)
proxy_pass http://192.168.1.10:8080;     # Forward requests to specific server IP and port (location)
proxy_pass http://upstream;              # Forward requests to named upstream group (location)
```

### Proxy Headers
```nginx
proxy_set_header Host $host;             # Preserve original host header from client request (location)
proxy_set_header X-Real-IP $remote_addr; # Pass real client IP address to backend (location)
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # Forward client IP chain to backend (location)
proxy_set_header X-Forwarded-Proto $scheme; # Pass original protocol (http/https) to backend (location)
proxy_set_header X-Forwarded-Host $server_name; # Pass server name to backend (location)
```

### Proxy Timeouts
```nginx
proxy_connect_timeout 60s;               # Timeout for establishing connection to backend server (location)
proxy_send_timeout 60s;                 # Timeout for sending request data to backend server (location)
proxy_read_timeout 60s;                 # Timeout for reading response data from backend server (location)
```

### Proxy Buffering
```nginx
proxy_buffering on;                     # Enable response buffering from backend server (location)
proxy_buffer_size 4k;                   # Size of buffer for reading response headers (location)
proxy_buffers 8 4k;                     # Number and size of buffers for reading response body (location)
proxy_busy_buffers_size 8k;             # Size of buffers that can be sent to client while reading (location)
```

### Proxy Redirects and Cookies
```nginx
proxy_redirect off;                     # Disable automatic redirect rewriting from backend (location)
proxy_redirect http://localhost:8080/ /; # Rewrite redirects from localhost:8080 to root path (location)
proxy_cookie_domain localhost example.com; # Rewrite cookie domain from localhost to example.com (location)
proxy_cookie_path /one/ /;              # Rewrite cookie path from /one/ to / (location)
```

### FastCGI and uWSGI
```nginx
# FastCGI (PHP) - pass requests to FastCGI server
fastcgi_pass 127.0.0.1:9000;           # Forward requests to FastCGI server (location)
fastcgi_index index.php;                # Set default FastCGI index file (location)
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; # Set script filename parameter (location)

# uWSGI (Python) - pass requests to uWSGI server
uwsgi_pass 127.0.0.1:8000;              # Forward requests to uWSGI server (location)
uwsgi_param SCRIPT_NAME $fastcgi_script_name; # Set script name parameter (location)
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
ssl_certificate /path/to/cert.crt;       # Path to SSL certificate file (server)
ssl_certificate_key /path/to/key.key;    # Path to SSL private key file (server)
ssl_protocols TLSv1.2 TLSv1.3;          # Allowed SSL/TLS protocol versions (server)
ssl_ciphers HIGH:!aNULL:!MD5;           # Allowed SSL cipher suites (server)
```

### SSL Optimization
```nginx
ssl_session_cache shared:SSL:10m;       # Enable SSL session cache with 10MB shared memory (server)
ssl_session_timeout 10m;                 # SSL session timeout duration (server)
ssl_stapling on;                         # Enable OCSP stapling for certificate validation (server)
ssl_stapling_verify on;                  # Enable OCSP stapling verification (server)
```

## HTTP/2 and HTTP/3 Configuration

### HTTP/2 Directives
```nginx
# HTTP/2 server - enable HTTP/2 protocol
listen 443 ssl http2;                    # Enable HTTP/2 on SSL port 443 (server)

# HTTP/2 optimizations - tune HTTP/2 performance
http2_max_field_size 4k;                 # Maximum size of HTTP/2 header field (server)
http2_max_header_size 16k;               # Maximum size of HTTP/2 header block (server)
http2_body_preread_size 64k;             # Size of HTTP/2 body preread buffer (server)
http2_idle_timeout 3m;                   # HTTP/2 connection idle timeout (server)
http2_recv_timeout 4m;                   # HTTP/2 connection receive timeout (server)
```

### HTTP/3 Directives (Nginx Plus)
```nginx
# HTTP/3 server - enable HTTP/3 protocol (Nginx Plus only)
listen 443 ssl http3;                   # Enable HTTP/3 on SSL port 443 (server)

# HTTP/3 optimizations - tune HTTP/3 performance
quic_retry on;                           # Enable QUIC retry mechanism for connection reliability (server)
quic_gso on;                             # Enable Generic Segmentation Offload for better performance (server)
quic_host_key /path/to/host.key;        # Path to host key file for QUIC connections (server)
```

## Advanced Caching Directives

### Proxy Cache
```nginx
# Cache path configuration - define cache storage location and settings
proxy_cache_path /var/cache/nginx 
    levels=1:2 
    keys_zone=my_cache:10m 
    max_size=1g 
    inactive=60m 
    use_temp_path=off;                   # Configure cache storage (http)

# Cache directives - control caching behavior
proxy_cache my_cache;                    # Enable caching using specified cache zone (location)
proxy_cache_valid 200 302 10m;          # Cache successful responses for 10 minutes (location)
proxy_cache_valid 404 1m;               # Cache 404 errors for 1 minute (location)
proxy_cache_use_stale error timeout updating; # Use stale cache when backend is unavailable (location)
proxy_cache_bypass $no_cache;           # Bypass cache when condition is true (location)
proxy_no_cache $no_cache;               # Don't cache when condition is true (location)
proxy_cache_key $scheme$proxy_host$request_uri; # Define cache key for storing responses (location)
```

### FastCGI Cache
```nginx
# FastCGI cache path - configure cache storage for FastCGI responses
fastcgi_cache_path /var/cache/nginx/fastcgi 
    levels=1:2 
    keys_zone=php_cache:10m 
    max_size=1g 
    inactive=60m;                        # Configure FastCGI cache storage (http)

# FastCGI cache directives - control FastCGI caching behavior
fastcgi_cache php_cache;                # Enable FastCGI caching using specified cache zone (location)
fastcgi_cache_valid 200 60m;             # Cache successful responses for 60 minutes (location)
fastcgi_cache_valid 404 1m;              # Cache 404 errors for 1 minute (location)
fastcgi_cache_bypass $no_cache;        # Bypass cache when condition is true (location)
fastcgi_no_cache $no_cache;             # Don't cache when condition is true (location)
fastcgi_cache_key $scheme$request_method$host$request_uri; # Define cache key for FastCGI responses (location)
```

### Cache Purging
```nginx
# Cache purge location - allow manual cache invalidation
location ~ /purge(/.*) {
    proxy_cache_purge my_cache $scheme$host$1; # Purge cache entry matching the pattern (location)
    access_log off;                            # Disable logging for purge requests (location)
}
```

## Advanced Rate Limiting

### DDoS Protection
```nginx
# Multiple rate limit zones
limit_req_zone $binary_remote_addr zone=general:10m rate=1r/s;
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
limit_req_zone $binary_remote_addr zone=upload:10m rate=1r/s;

# Connection limiting
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
limit_conn_zone $server_name zone=conn_limit_per_server:10m;

# Apply rate limiting
server {
    listen 80;
    server_name example.com;
    
    # General rate limiting
    limit_req zone=general burst=5 nodelay;
    limit_conn conn_limit_per_ip 10;
    limit_conn conn_limit_per_server 100;
    
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
    
    # Upload endpoint
    location /upload {
        limit_req zone=upload burst=2 nodelay;
        client_max_body_size 100M;
        proxy_pass http://backend;
    }
}
```

### Advanced Rate Limiting Patterns
```nginx
# Rate limiting by user agent
map $http_user_agent $is_bot {
    default 0;
    ~*bot 1;
    ~*crawler 1;
    ~*spider 1;
}

# Rate limiting by country
geo $country {
    default US;
    192.168.1.0/24 CA;
    10.0.0.0/8 US;
}

# Different rates for different countries
limit_req_zone $binary_remote_addr zone=us_rate:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=ca_rate:10m rate=5r/s;

location / {
    if ($is_bot) {
        limit_req zone=general burst=1 nodelay;
    }
    if ($country = "US") {
        limit_req zone=us_rate burst=20 nodelay;
    }
    if ($country = "CA") {
        limit_req zone=ca_rate burst=10 nodelay;
    }
    proxy_pass http://backend;
}
```

## Security Hardening Directives

### Security Headers
```nginx
# Security headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

### SSL/TLS Hardening
```nginx
# Modern SSL configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /path/to/trusted_ca_cert.pem;
ssl_dhparam /path/to/dhparam.pem;
```

### Access Control and Blocking
```nginx
# Block common attacks
location ~* /(wp-admin|wp-login|xmlrpc|admin|administrator) {
    return 403;
}

# Block suspicious requests
if ($request_uri ~* "(union|select|insert|delete|drop|create|alter|exec)") {
    return 403;
}

# Block bad user agents
if ($http_user_agent ~* "(bot|crawler|spider|scraper|curl|wget)") {
    return 403;
}

# Block empty user agents
if ($http_user_agent = "") {
    return 403;
}

# Block requests with suspicious headers
if ($http_x_forwarded_for ~* "(127\.0\.0\.1|localhost)") {
    return 403;
}
```

## Monitoring and Metrics Directives

### Status and Monitoring
```nginx
# Nginx status
location /nginx_status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    allow 192.168.1.0/24;
    deny all;
}

# Health check endpoint
location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
}

# Metrics endpoint
location /metrics {
    access_log off;
    return 200 "nginx_up 1\n";
    add_header Content-Type text/plain;
}
```

### Advanced Logging
```nginx
# Custom log formats
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

log_format json_combined escape=json
    '{"time":"$time_iso8601","remote_addr":"$remote_addr",'
    '"request":"$request","status":"$status","body_bytes_sent":"$body_bytes_sent",'
    '"http_referer":"$http_referer","http_user_agent":"$http_user_agent"}';

log_format upstream '$remote_addr - $upstream_addr [$time_local] '
                   '"$request" $upstream_status $upstream_response_time';

# Apply logging
access_log /var/log/nginx/access.log main;
access_log /var/log/nginx/access.json json_combined;
```

## Debugging Directives

### Error Logging
```nginx
# Error log levels
error_log /var/log/nginx/error.log warn;
error_log /var/log/nginx/debug.log debug;

# Debug specific connections
debug_connection 192.168.1.0/24;
debug_connection 127.0.0.1;

# Debug specific locations
location /debug {
    error_log /var/log/nginx/debug.log debug;
    return 200 "Debug mode enabled\n";
}
```

### Performance Debugging
```nginx
# Add timing headers
add_header X-Response-Time $request_time always;
add_header X-Upstream-Response-Time $upstream_response_time always;
add_header X-Cache-Status $upstream_cache_status always;

# Debug upstream
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Health check
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
}
```

## Performance Tuning Directives

### Worker Optimization
```nginx
# Main context - configure worker processes
worker_processes auto;                   # Set number of worker processes to CPU cores (main)
worker_rlimit_nofile 65535;             # Set maximum number of open files per worker (main)
worker_cpu_affinity auto;               # Bind workers to CPU cores automatically (main)

# Events context - configure event handling
events {
    worker_connections 1024;             # Maximum connections per worker process (events)
    use epoll;                           # Use epoll event method for Linux (events)
    multi_accept on;                     # Accept multiple connections at once (events)
    accept_mutex off;                    # Disable accept mutex for better performance (events)
}
```

### HTTP Optimization
```nginx
# HTTP performance - optimize HTTP connection handling
sendfile on;                             # Use efficient file serving method (http, server, location)
tcp_nopush on;                           # Optimize TCP packet sending (http, server, location)
tcp_nodelay on;                          # Disable Nagle's algorithm for better performance (http, server, location)
keepalive_timeout 65;                    # Keep-alive connection timeout (http, server, location)
keepalive_requests 100;                  # Maximum requests per keep-alive connection (http, server, location)

# Buffer optimization - configure client request buffering
client_body_buffer_size 128k;            # Buffer size for client request body (http, server, location)
client_max_body_size 10m;                # Maximum size of client request body (http, server, location)
client_header_buffer_size 1k;            # Buffer size for client request headers (http, server, location)
large_client_header_buffers 4 4k;         # Buffers for large client request headers (http, server, location)

# Proxy optimization - configure proxy response buffering
proxy_buffering on;                      # Enable proxy response buffering (http, server, location)
proxy_buffer_size 4k;                    # Buffer size for proxy response headers (http, server, location)
proxy_buffers 8 4k;                      # Number and size of proxy response buffers (http, server, location)
proxy_busy_buffers_size 8k;              # Size of buffers that can be sent to client (http, server, location)
```

### Connection Pooling
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Connection pooling - maintain persistent connections to backend
    keepalive 32;                          # Keep 32 idle connections to backend servers (upstream)
    keepalive_requests 100;               # Maximum requests per keep-alive connection (upstream)
    keepalive_timeout 60s;                # Keep-alive connection timeout (upstream)
    
    # Health checks - monitor backend server health
    server 192.168.1.10:8080 max_conns=100; # Limit connections to this server (upstream)
    server 192.168.1.11:8080 max_conns=100; # Limit connections to this server (upstream)
}
```

## Headers and CORS

### Security Headers
```nginx
add_header X-Frame-Options "SAMEORIGIN" always; # Prevent clickjacking by allowing framing only on same origin (http, server, location)
add_header X-Content-Type-Options "nosniff" always; # Prevent MIME type sniffing attacks (http, server, location)
add_header X-XSS-Protection "1; mode=block" always; # Enable XSS protection in browsers (http, server, location)
add_header Strict-Transport-Security "max-age=31536000" always; # Force HTTPS for 1 year (http, server, location)
```

### CORS Headers
```nginx
add_header Access-Control-Allow-Origin "*" always; # Allow cross-origin requests from any domain (http, server, location)
add_header Access-Control-Allow-Methods "GET, POST, OPTIONS" always; # Allow specific HTTP methods for CORS (http, server, location)
add_header Access-Control-Allow-Headers "Content-Type, Authorization" always; # Allow specific headers for CORS (http, server, location)
```

## URL Rewriting

### Basic Rewrites
```nginx
rewrite ^/old/(.*)$ /new/$1 permanent;   # Permanent redirect (301) to new URL (server, location)
rewrite ^/old/(.*)$ /new/$1 redirect;    # Temporary redirect (302) to new URL (server, location)
rewrite ^/old/(.*)$ /new/$1 last;        # Internal redirect, restart location matching (server, location)
rewrite ^/old/(.*)$ /new/$1 break;       # Stop processing and use current location (server, location)
```

### Common Rewrite Patterns
```nginx
# Remove trailing slash - redirect URLs ending with / to version without /
rewrite ^/(.*)/$ /$1 permanent;          # Remove trailing slash with permanent redirect (server, location)

# Add trailing slash - redirect URLs without / to version with /
rewrite ^/(.*[^/])$ /$1/ permanent;     # Add trailing slash with permanent redirect (server, location)

# Remove www - redirect www subdomain to main domain
if ($host = 'www.example.com') {
    return 301 https://example.com$request_uri; # Redirect www to non-www domain (server, location)
}

# Force HTTPS - redirect HTTP to HTTPS
if ($scheme != "https") {
    return 301 https://$server_name$request_uri; # Redirect HTTP to HTTPS (server, location)
}
```

## Caching and Compression

### Static File Caching
```nginx
location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;                           # Cache static files for 1 year (location)
    add_header Cache-Control "public, immutable"; # Add cache control headers (location)
    access_log off;                       # Don't log static file requests (location)
}
```

### Gzip Compression
```nginx
gzip on;                                 # Enable gzip compression for responses (http, server, location)
gzip_vary on;                            # Add Vary header to indicate compression (http, server, location)
gzip_min_length 1024;                    # Minimum file size to compress (http, server, location)
gzip_comp_level 6;                      # Compression level (1-9, 6 is good balance) (http, server, location)
gzip_types text/plain text/css application/json application/javascript; # File types to compress (http, server, location)
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
