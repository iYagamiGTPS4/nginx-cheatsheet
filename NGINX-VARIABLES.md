# Nginx Variables

Variables store request and server information for use in configuration.

## Request Variables

### Basic Request Info
```nginx
$request_uri          # Full URI with query string (e.g., /api/users?id=123)
$uri                  # URI without query string (e.g., /api/users)
$args                 # Query string parameters (e.g., id=123&name=john)
$query_string         # Same as $args (alternative name)
$request_method       # HTTP method (GET, POST, PUT, DELETE, etc.)
$request_body         # Request body content (POST data)
$request_length       # Request body length in bytes
$request_time         # Request processing time in seconds
```

### Headers
```nginx
$http_host            # Host header from client request
$http_user_agent      # User-Agent header (browser/client info)
$http_referer         # Referer header (previous page URL)
$http_x_forwarded_for # X-Forwarded-For header (client IP chain)
$http_accept          # Accept header (content types client accepts)
$http_accept_encoding # Accept-Encoding header (compression support)
$http_accept_language # Accept-Language header (preferred languages)
$http_cookie          # Cookie header (client cookies)
$http_authorization   # Authorization header (authentication data)
```

### Custom Headers
```nginx
$http_custom_header   # Any custom header (lowercase, hyphens to underscores)
$http_x_real_ip       # X-Real-IP header (real client IP from proxy)
$http_x_forwarded_proto # X-Forwarded-Proto header (original protocol)
```

## Response Variables

### Status and Content
```nginx
$status               # HTTP status code (200, 404, 500, etc.)
$body_bytes_sent      # Bytes sent to client (response body size)
$bytes_sent           # Same as $body_bytes_sent (alternative name)
$content_length       # Content-Length header value
$content_type         # Content-Type header value
```

## Connection Variables

### Client Info
```nginx
$remote_addr          # Client IP address (e.g., 192.168.1.100)
$remote_port          # Client port number
$remote_user          # Username from basic authentication
$binary_remote_addr   # Binary representation of client IP (for rate limiting)
```

### Server Info
```nginx
$server_addr          # Server IP address (e.g., 192.168.1.10)
$server_name          # Server name from server_name directive
$server_port          # Server port number (80, 443, etc.)
$server_protocol      # HTTP protocol version (HTTP/1.1, HTTP/2.0)
```

## Time Variables

```nginx
$time_iso8601         # ISO 8601 time format (2023-12-01T10:30:45+00:00)
$time_local           # Local time format (01/Dec/2023:10:30:45 +0000)
$msec                 # Current time in seconds with milliseconds (1701432645.123)
$date_gmt             # GMT date string
$date_local           # Local date string
```

## File Variables

```nginx
$document_root        # Root directory from root directive (e.g., /var/www/html)
$document_uri         # Same as $uri (URI without query string)
$realpath_root        # Real path of document root (resolved symlinks)
$request_filename     # Full path to requested file (e.g., /var/www/html/index.html)
```

## Scheme and Protocol

```nginx
$scheme               # Protocol scheme (http or https)
$https                # "on" if HTTPS connection, empty otherwise
$request_scheme       # Same as $scheme (alternative name)
$ssl_protocol         # SSL protocol version (TLSv1.2, TLSv1.3)
$ssl_cipher           # SSL cipher suite used for encryption
```

## System Variables

```nginx
$host                 # Actual host (from Host header or server_name)
$hostname             # Machine hostname (e.g., web-server-01)
$nginx_version        # Nginx version (e.g., 1.24.0)
$pid                  # Worker process PID (process ID)
```

## Connection Variables

```nginx
$connection           # Connection serial number (unique per connection)
$connection_requests  # Number of requests in current connection
$pipe                 # "p" if pipelined request, "." otherwise
```

## Upstream Variables

```nginx
$upstream_addr        # Upstream server address (e.g., 192.168.1.10:8080)
$upstream_status      # Upstream response status (200, 404, 502, etc.)
$upstream_response_time # Upstream response time in seconds
$upstream_connect_time # Upstream connect time in seconds
$upstream_header_time # Upstream header time in seconds
$upstream_bytes_received # Bytes received from upstream server
$upstream_bytes_sent # Bytes sent to upstream server
$upstream_cache_status # Cache status (HIT, MISS, BYPASS, EXPIRED, STALE)
```

## HTTP/2 Variables

```nginx
$http2                # HTTP/2 protocol version (e.g., 2.0)
$http2_stream_id      # HTTP/2 stream ID (unique per stream)
$http2_stream_priority # HTTP/2 stream priority (0-255)
$http2_stream_dependency # HTTP/2 stream dependency (parent stream ID)
```

## Cache Variables

```nginx
$upstream_cache_status # Cache status (HIT, MISS, BYPASS, EXPIRED, STALE, UPDATING)
$cache_status         # Same as $upstream_cache_status (alternative name)
$cache_valid          # Cache validity status (1 if valid, 0 if expired)
$cache_age            # Cache age in seconds since creation
$cache_modified       # Cache modification time (Unix timestamp)
```

## Rate Limiting Variables

```nginx
$limit_req_status     # Rate limit status (PASS, DELAY, REJECT)
$limit_conn_status    # Connection limit status (PASS, REJECT)
$limit_rate           # Current rate limit (requests per second)
$limit_conn           # Current connection limit (max connections)
```

## Real-time Monitoring Variables

```nginx
$connections_active   # Active connections (currently processing)
$connections_reading  # Connections in reading state (receiving request)
$connections_writing  # Connections in writing state (sending response)
$connections_waiting  # Connections in waiting state (idle)
$connections_accepted # Total accepted connections (since startup)
$connections_handled # Total handled connections (since startup)
$requests_total      # Total requests (since startup)
```

## Performance Variables

```nginx
$request_time         # Request processing time in seconds
$upstream_response_time # Upstream response time in seconds
$upstream_connect_time # Upstream connect time in seconds
$upstream_header_time # Upstream header time in seconds
$pipe                 # "p" if pipelined request, "." otherwise
$connection_requests  # Number of requests in current connection
$connection           # Connection serial number (unique per connection)
```

## Security Variables

```nginx
$ssl_protocol         # SSL protocol version (TLSv1.2, TLSv1.3)
$ssl_cipher           # SSL cipher suite used for encryption
$ssl_session_id       # SSL session ID (unique per session)
$ssl_client_verify    # SSL client verification status (SUCCESS, FAILED, NONE)
$ssl_client_s_dn      # SSL client subject DN (certificate subject)
$ssl_client_i_dn      # SSL client issuer DN (certificate issuer)
$ssl_client_fingerprint # SSL client certificate fingerprint (SHA-1 hash)
```

## Custom Variables

### Map Context Variables
```nginx
# Create custom variables
map $http_user_agent $mobile {
    default 0;
    ~*mobile 1;
    ~*android 1;
    ~*iphone 1;
}

map $http_user_agent $is_bot {
    default 0;
    ~*bot 1;
    ~*crawler 1;
    ~*spider 1;
}

# Usage
if ($mobile) {
    rewrite ^ /mobile$uri;
}

if ($is_bot) {
    return 403;
}
```

### Geo Context Variables
```nginx
# Geographic variables
geo $country {
    default US;
    192.168.1.0/24 CA;
    10.0.0.0/8 US;
    172.16.0.0/12 EU;
}

# Usage
if ($country = "EU") {
    return 301 https://eu.example.com$request_uri;
}
```

### Split Clients Variables
```nginx
# A/B testing variables
split_clients "${remote_addr}${http_user_agent}" $variant {
    50% "A";
    50% "B";
}

# Usage
if ($variant = "A") {
    rewrite ^ /version-a$uri;
}
```

## Common Usage Examples

### Logging
```nginx
# Custom log format
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent"';

# Usage
access_log /var/log/nginx/access.log main;
```

### Conditional Logic
```nginx
# Mobile detection
if ($http_user_agent ~* "mobile") {
    rewrite ^ /mobile$uri;
}

# HTTPS redirect
if ($scheme != "https") {
    return 301 https://$server_name$request_uri;
}
```

### Proxy Headers
```nginx
location /api/ {
    proxy_pass http://backend;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### Rate Limiting
```nginx
# Rate limit by IP
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

# Rate limit by user
limit_req_zone $remote_user zone=user:10m rate=5r/m;
```

### Upstream Monitoring
```nginx
# Log upstream response times
log_format upstream '$remote_addr - $upstream_addr [$time_local] '
                   '"$request" $upstream_status $upstream_response_time';

# Use in location
location /api/ {
    proxy_pass http://backend;
    access_log /var/log/nginx/upstream.log upstream;
}
```

### Cache Monitoring
```nginx
# Log cache status
log_format cache '$remote_addr - [$time_local] "$request" '
                '$status $upstream_cache_status $upstream_response_time';

# Use in location
location /api/ {
    proxy_cache my_cache;
    proxy_pass http://backend;
    access_log /var/log/nginx/cache.log cache;
}
```

### Rate Limiting Monitoring
```nginx
# Log rate limit status
log_format rate_limit '$remote_addr - [$time_local] "$request" '
                     '$status $limit_req_status $limit_conn_status';

# Use in location
location /api/ {
    limit_req zone=api burst=20 nodelay;
    limit_conn conn_limit_per_ip 10;
    access_log /var/log/nginx/rate_limit.log rate_limit;
    proxy_pass http://backend;
}
```

### Performance Monitoring
```nginx
# Log performance metrics
log_format performance '$remote_addr - [$time_local] "$request" '
                     '$status $request_time $upstream_response_time '
                     '$upstream_connect_time $upstream_header_time';

# Use in location
location /api/ {
    proxy_pass http://backend;
    access_log /var/log/nginx/performance.log performance;
}
```

### Security Monitoring
```nginx
# Log security events
log_format security '$remote_addr - [$time_local] "$request" '
                   '$status $http_user_agent $ssl_protocol $ssl_cipher';

# Use in location
location / {
    access_log /var/log/nginx/security.log security;
    try_files $uri $uri/ /index.html;
}
```

### SSL Information
```nginx
# Log SSL details
log_format ssl '$remote_addr - [$time_local] "$request" '
              '$status $ssl_protocol $ssl_cipher';

# Conditional SSL redirect
if ($ssl_protocol = "") {
    return 301 https://$server_name$request_uri;
}
```

### Access Control
```nginx
# Allow specific IPs
if ($remote_addr ~ "^(192\.168\.1\.|10\.0\.0\.)") {
    # Allow
}

# Block bots
if ($http_user_agent ~* "(bot|crawler|spider)") {
    return 403;
}
```

### URL Rewriting
```nginx
# Remove trailing slash
if ($uri ~ "^/(.*)/$") {
    rewrite ^/(.*)/$ /$1 permanent;
}

# API versioning
if ($uri ~ "^/api/v1/(.*)$") {
    rewrite ^/api/v1/(.*)$ /api/v2/$1 last;
}
```

## Variable Categories

### Request Information
| Variable | Description | Example |
|----------|-------------|---------|
| `$request_uri` | Full URI with query | `/api/users?id=123` |
| `$uri` | URI without query | `/api/users` |
| `$args` | Query string | `id=123` |
| `$request_method` | HTTP method | `GET`, `POST` |
| `$request_time` | Processing time | `0.123` |

### Client Information
| Variable | Description | Example |
|----------|-------------|---------|
| `$remote_addr` | Client IP | `192.168.1.100` |
| `$remote_user` | Auth username | `john` |
| `$http_user_agent` | User agent | `Mozilla/5.0...` |
| `$http_referer` | Referer header | `https://google.com` |

### Server Information
| Variable | Description | Example |
|----------|-------------|---------|
| `$server_name` | Server name | `example.com` |
| `$server_addr` | Server IP | `192.168.1.10` |
| `$server_port` | Server port | `80`, `443` |
| `$scheme` | Protocol | `http`, `https` |

### Response Information
| Variable | Description | Example |
|----------|-------------|---------|
| `$status` | HTTP status | `200`, `404`, `500` |
| `$body_bytes_sent` | Bytes sent | `1024` |
| `$content_type` | Content type | `text/html` |

## Advanced Variable Usage

### Map Context
```nginx
# Create custom variables
map $http_user_agent $mobile {
    default 0;
    ~*mobile 1;
    ~*android 1;
    ~*iphone 1;
}

# Usage
if ($mobile) {
    rewrite ^ /mobile$uri;
}
```

### Geo Context
```nginx
# Geographic variables
geo $country {
    default US;
    192.168.1.0/24 CA;
    10.0.0.0/8 US;
}

# Usage
if ($country = "US") {
    # US-specific logic
}
```

### Split Clients
```nginx
# A/B testing
split_clients "${remote_addr}${http_user_agent}" $variant {
    50% "A";
    50% "B";
}

# Usage
if ($variant = "A") {
    rewrite ^ /version-a$uri;
}
```

## Advanced Variable Usage Patterns

### Dynamic Content Routing
```nginx
# Route based on user agent
map $http_user_agent $backend {
    default backend_web;
    ~*mobile backend_mobile;
    ~*bot backend_api;
}

# Route based on request method
map $request_method $method_backend {
    GET backend_read;
    POST backend_write;
    PUT backend_write;
    DELETE backend_write;
}

# Usage
location / {
    proxy_pass http://$backend;
}
```

### Conditional Headers
```nginx
# Add headers based on conditions
map $http_user_agent $add_mobile_header {
    default "";
    ~*mobile "X-Mobile: 1";
}

# Usage
location / {
    add_header $add_mobile_header;
    try_files $uri $uri/ /index.html;
}
```

### Dynamic Rate Limiting
```nginx
# Different rates for different user agents
map $http_user_agent $rate_limit {
    default "10r/s";
    ~*bot "1r/s";
    ~*mobile "5r/s";
}

# Usage
location / {
    limit_req zone=dynamic rate=$rate_limit burst=20 nodelay;
    proxy_pass http://backend;
}
```

### Geographic Content
```nginx
# Different content for different countries
geo $country {
    default US;
    192.168.1.0/24 CA;
    10.0.0.0/8 US;
    172.16.0.0/12 EU;
}

# Usage
location / {
    if ($country = "EU") {
        return 301 https://eu.example.com$request_uri;
    }
    try_files $uri $uri/ /index.html;
}
```

### A/B Testing
```nginx
# A/B testing with split clients
split_clients "${remote_addr}${http_user_agent}" $variant {
    50% "A";
    50% "B";
}

# Usage
location / {
    if ($variant = "A") {
        rewrite ^ /version-a$uri;
    }
    if ($variant = "B") {
        rewrite ^ /version-b$uri;
    }
    try_files $uri $uri/ /index.html;
}
```

### Cache Control
```nginx
# Cache control based on request
map $request_uri $cache_control {
    default "public, max-age=3600";
    ~*\.(css|js|png|jpg|jpeg|gif|ico|svg)$ "public, max-age=31536000";
    ~*\.(html|htm)$ "public, max-age=0";
}

# Usage
location / {
    add_header Cache-Control $cache_control;
    try_files $uri $uri/ /index.html;
}
```

### Security Headers
```nginx
# Security headers based on request
map $request_uri $security_headers {
    default "X-Frame-Options: SAMEORIGIN";
    ~*\.(css|js|png|jpg|jpeg|gif|ico|svg)$ "X-Frame-Options: DENY";
}

# Usage
location / {
    add_header $security_headers;
    try_files $uri $uri/ /index.html;
}
```

## Variable Scope

Variables are available in different contexts:

- **Main context**: Limited variables
- **HTTP context**: Most variables available
- **Server context**: All HTTP variables
- **Location context**: All variables
- **If context**: All variables

## Performance Notes

- Variables are evaluated at runtime
- Use `$binary_remote_addr` instead of `$remote_addr` for rate limiting
- Cache frequently used variables with `map`
- Avoid complex regex in variable evaluation
- Use `$request_time` for performance monitoring
- Use `$upstream_response_time` for upstream monitoring

## Debugging Variables

### Common Debug Variables
```nginx
# Debug request processing
add_header X-Debug-Request-Time $request_time;
add_header X-Debug-Upstream-Time $upstream_response_time;
add_header X-Debug-Cache-Status $upstream_cache_status;
add_header X-Debug-Rate-Limit $limit_req_status;

# Debug connection info
add_header X-Debug-Connection $connection;
add_header X-Debug-Connection-Requests $connection_requests;
add_header X-Debug-Pipe $pipe;
```

### Error Debugging
```nginx
# Debug error conditions
if ($status = 404) {
    add_header X-Debug-Error "File not found: $request_uri";
}
if ($status = 500) {
    add_header X-Debug-Error "Internal server error";
}
if ($upstream_status = 502) {
    add_header X-Debug-Error "Bad gateway: $upstream_addr";
}
```

---

[← Contexts](NGINX-CONTEXTS.md) | [Back to README](README.md) | [Directives →](NGINX-DIRECTIVES.md)
