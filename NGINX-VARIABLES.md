# Nginx Variables

Variables store request and server information for use in configuration.

## Request Variables

### Basic Request Info
```nginx
$request_uri          # Full URI with query string
$uri                  # URI without query string
$args                 # Query string
$query_string         # Same as $args
$request_method       # GET, POST, PUT, DELETE, etc.
$request_body         # Request body content
$request_length       # Request body length
$request_time         # Request processing time
```

### Headers
```nginx
$http_host            # Host header
$http_user_agent      # User-Agent header
$http_referer         # Referer header
$http_x_forwarded_for # X-Forwarded-For header
$http_accept          # Accept header
$http_accept_encoding # Accept-Encoding header
$http_accept_language # Accept-Language header
$http_cookie          # Cookie header
$http_authorization   # Authorization header
```

### Custom Headers
```nginx
$http_custom_header   # Any header (lowercase, hyphens to underscores)
$http_x_real_ip       # X-Real-IP header
$http_x_forwarded_proto # X-Forwarded-Proto header
```

## Response Variables

### Status and Content
```nginx
$status               # HTTP status code
$body_bytes_sent      # Bytes sent to client
$bytes_sent           # Same as $body_bytes_sent
$content_length       # Content-Length header
$content_type         # Content-Type header
```

## Connection Variables

### Client Info
```nginx
$remote_addr          # Client IP address
$remote_port          # Client port
$remote_user          # Username from basic auth
$binary_remote_addr   # Binary representation of client IP
```

### Server Info
```nginx
$server_addr          # Server IP address
$server_name          # Server name from server_name directive
$server_port          # Server port
$server_protocol      # HTTP protocol version
```

## Time Variables

```nginx
$time_iso8601         # ISO 8601 time format
$time_local           # Local time format
$msec                 # Current time in seconds with milliseconds
$date_gmt             # GMT date
$date_local           # Local date
```

## File Variables

```nginx
$document_root        # Root directory from root directive
$document_uri         # Same as $uri
$realpath_root        # Real path of document root
$request_filename     # Full path to requested file
```

## Scheme and Protocol

```nginx
$scheme               # http or https
$https                # "on" if HTTPS, empty otherwise
$request_scheme       # Same as $scheme
$ssl_protocol         # SSL protocol version (TLSv1.2, TLSv1.3)
$ssl_cipher           # SSL cipher suite
```

## System Variables

```nginx
$host                 # Actual host (from Host header or server_name)
$hostname             # Machine hostname
$nginx_version        # Nginx version
$pid                  # Worker process PID
```

## Connection Variables

```nginx
$connection           # Connection serial number
$connection_requests  # Number of requests in connection
$pipe                 # "p" if pipelined, "." otherwise
```

## Upstream Variables

```nginx
$upstream_addr        # Upstream server address
$upstream_status      # Upstream response status
$upstream_response_time # Upstream response time
$upstream_connect_time # Upstream connect time
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

---

[← Contexts](NGINX-CONTEXTS.md) | [Back to README](README.md) | [Directives →](NGINX-DIRECTIVES.md)
