# Advanced Security

Enterprise-level security configurations for protecting high-value applications.

## Web Application Firewall (WAF)

### ModSecurity Integration
```nginx
# Load ModSecurity module
load_module modules/ngx_http_modsecurity_module.so;

server {
    listen 80;
    server_name example.com;
    
    # Enable ModSecurity
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsecurity.conf;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Custom Security Rules
```nginx
# Custom security rules
server {
    location / {
        # Block SQL injection attempts
        if ($request_uri ~* "(union|select|insert|delete|drop|create|alter|exec)") {
            return 403;
        }
        
        # Block XSS attempts
        if ($request_uri ~* "(<script|javascript:|onload=|onerror=)") {
            return 403;
        }
        
        # Block path traversal
        if ($request_uri ~* "\.\./") {
            return 403;
        }
        
        proxy_pass http://backend;
    }
}
```

## Advanced Rate Limiting

### Multi-Tier Rate Limiting
```nginx
# Different rate limits for different endpoints
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=upload:10m rate=1r/s;
limit_req_zone $binary_remote_addr zone=general:10m rate=1r/s;

# Connection limiting
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
limit_conn_zone $server_name zone=conn_limit_per_server:10m;

server {
    listen 80;
    server_name example.com;
    
    # General rate limiting
    limit_req zone=general burst=5 nodelay;
    limit_conn conn_limit_per_ip 10;
    limit_conn conn_limit_per_server 100;
    
    # Login endpoint - strict rate limiting
    location /login {
        limit_req zone=login burst=3 nodelay;
        proxy_pass http://backend;
    }
    
    # API endpoints - moderate rate limiting
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://backend;
    }
    
    # Upload endpoint - very strict rate limiting
    location /upload {
        limit_req zone=upload burst=2 nodelay;
        client_max_body_size 100M;
        proxy_pass http://backend;
    }
}
```

### Geographic Rate Limiting
```nginx
# Rate limiting by country
geo $country {
    default US;
    192.168.1.0/24 US;
    10.0.0.0/8 US;
    172.16.0.0/12 EU;
}

# Different rates for different countries
limit_req_zone $binary_remote_addr zone=us_rate:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=eu_rate:10m rate=5r/s;

server {
    location / {
        if ($country = "US") {
            limit_req zone=us_rate burst=20 nodelay;
        }
        if ($country = "EU") {
            limit_req zone=eu_rate burst=10 nodelay;
        }
        proxy_pass http://backend;
    }
}
```

## Bot Detection and Blocking

### User Agent Analysis
```nginx
# Bot detection map
map $http_user_agent $is_bot {
    default 0;
    ~*bot 1;
    ~*crawler 1;
    ~*spider 1;
    ~*scraper 1;
    ~*curl 1;
    ~*wget 1;
    ~*python 1;
    ~*java 1;
    ~*php 1;
}

# Block bots
server {
    location / {
        if ($is_bot) {
            return 403 "Access denied";
        }
        proxy_pass http://backend;
    }
}
```

### Behavioral Analysis
```nginx
# Track request patterns
map $request_uri $suspicious_pattern {
    default 0;
    ~*"/admin" 1;
    ~*"/wp-admin" 1;
    ~*"/xmlrpc" 1;
    ~*"/.env" 1;
    ~*"/config" 1;
}

# Block suspicious patterns
server {
    location / {
        if ($suspicious_pattern) {
            return 403 "Access denied";
        }
        proxy_pass http://backend;
    }
}
```

## DDoS Mitigation

### Connection Limiting
```nginx
# Limit connections per IP
limit_conn_zone $binary_remote_addr zone=conn_limit_per_ip:10m;
limit_conn_zone $server_name zone=conn_limit_per_server:10m;

# Limit requests per IP
limit_req_zone $binary_remote_addr zone=req_limit_per_ip:10m rate=10r/s;

server {
    listen 80;
    server_name example.com;
    
    # Apply limits
    limit_conn conn_limit_per_ip 5;
    limit_conn conn_limit_per_server 1000;
    limit_req zone=req_limit_per_ip burst=10 nodelay;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### IP Blacklisting
```nginx
# IP blacklist
geo $blacklist {
    default 0;
    192.168.1.100 1;
    10.0.0.50 1;
    # Add more IPs as needed
}

server {
    location / {
        if ($blacklist) {
            return 403 "IP blocked";
        }
        proxy_pass http://backend;
    }
}
```

## Client Certificate Authentication

### SSL Client Certificates
```nginx
server {
    listen 443 ssl;
    server_name secure.example.com;
    
    ssl_certificate /path/to/server.crt;
    ssl_certificate_key /path/to/server.key;
    
    # Require client certificates
    ssl_client_certificate /path/to/ca.crt;
    ssl_verify_client on;
    ssl_verify_depth 2;
    
    # Map client certificates to users
    map $ssl_client_s_dn $client_user {
        default "unknown";
        ~CN=admin admin;
        ~CN=user user;
    }
    
    location / {
        proxy_pass http://backend;
        proxy_set_header X-Client-User $client_user;
    }
}
```

### Certificate-Based Access Control
```nginx
# Certificate-based access control
map $ssl_client_s_dn $allowed_access {
    default 0;
    ~CN=admin 1;
    ~CN=manager 1;
}

server {
    location /admin {
        if ($allowed_access = 0) {
            return 403 "Certificate required";
        }
        proxy_pass http://backend;
    }
}
```

## OAuth/JWT Validation

### JWT Token Validation
```nginx
# JWT validation (requires lua-resty-jwt module)
server {
    location /api/ {
        access_by_lua_block {
            local jwt = require "resty.jwt"
            local auth_header = ngx.var.http_authorization
            
            if not auth_header then
                ngx.status = 401
                ngx.say("Missing authorization header")
                ngx.exit(401)
            end
            
            local token = auth_header:match("Bearer%s+(.+)")
            if not token then
                ngx.status = 401
                ngx.say("Invalid authorization format")
                ngx.exit(401)
            end
            
            local jwt_obj = jwt:verify("your-secret-key", token)
            if not jwt_obj.valid then
                ngx.status = 401
                ngx.say("Invalid token")
                ngx.exit(401)
            end
        }
        
        proxy_pass http://backend;
    }
}
```

### OAuth Token Validation
```nginx
# OAuth token validation
server {
    location /api/ {
        # Validate OAuth token
        auth_request /auth;
        auth_request_set $user $upstream_http_x_user;
        auth_request_set $scope $upstream_http_x_scope;
        
        proxy_set_header X-User $user;
        proxy_set_header X-Scope $scope;
        proxy_pass http://backend;
    }
    
    location = /auth {
        internal;
        proxy_pass http://oauth-server/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

## Comprehensive Security Headers

### Security Headers Configuration
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self'; frame-ancestors 'self';" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=(), payment=(), usb=(), magnetometer=(), gyroscope=(), speaker=()" always;
    
    # Remove server information
    server_tokens off;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### Dynamic Security Headers
```nginx
# Different security headers for different content types
map $request_uri $csp_policy {
    default "default-src 'self'";
    ~*\.html$ "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'";
    ~*\.json$ "default-src 'self'";
}

server {
    location / {
        add_header Content-Security-Policy $csp_policy always;
        proxy_pass http://backend;
    }
}
```

## Advanced Access Control

### IP Whitelisting with Geolocation
```nginx
# Geolocation-based access control
geo $country {
    default US;
    192.168.1.0/24 US;
    10.0.0.0/8 US;
    172.16.0.0/12 EU;
}

# Allow only specific countries
map $country $allowed_country {
    default 0;
    US 1;
    CA 1;
    EU 0;
}

server {
    location / {
        if ($allowed_country = 0) {
            return 403 "Access denied from your location";
        }
        proxy_pass http://backend;
    }
}
```

### Time-Based Access Control
```nginx
# Time-based access control
map $time_iso8601 $business_hours {
    default 0;
    ~T0[8-9]: 1;
    ~T1[0-7]: 1;
}

server {
    location /admin {
        if ($business_hours = 0) {
            return 403 "Access only during business hours";
        }
        proxy_pass http://backend;
    }
}
```

## Security Monitoring

### Security Event Logging
```nginx
# Custom security log format
log_format security '$remote_addr - [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for" '
                   '$ssl_protocol $ssl_cipher $ssl_client_s_dn';

server {
    location / {
        access_log /var/log/nginx/security.log security;
        proxy_pass http://backend;
    }
}
```

### Failed Authentication Tracking
```nginx
# Track failed authentication attempts
map $status $auth_failure {
    default 0;
    401 1;
    403 1;
}

server {
    location / {
        if ($auth_failure) {
            access_log /var/log/nginx/auth_failures.log;
        }
        proxy_pass http://backend;
    }
}
```

## Real-World Security Patterns

### E-commerce Security
```nginx
# E-commerce security configuration
server {
    location /checkout {
        # Rate limit checkout attempts
        limit_req zone=checkout burst=3 nodelay;
        
        # Require HTTPS
        if ($scheme != "https") {
            return 301 https://$server_name$request_uri;
        }
        
        # Add security headers
        add_header X-Frame-Options "DENY" always;
        add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self';" always;
        
        proxy_pass http://backend;
    }
}
```

### API Security
```nginx
# API security configuration
server {
    location /api/ {
        # Rate limit API calls
        limit_req zone=api burst=20 nodelay;
        
        # Require API key
        if ($http_x_api_key = "") {
            return 401 "API key required";
        }
        
        # Validate API key
        auth_request /validate-api-key;
        
        proxy_pass http://backend;
    }
    
    location = /validate-api-key {
        internal;
        proxy_pass http://api-auth-server/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-API-Key $http_x_api_key;
    }
}
```

## Security Best Practices

1. **Defense in Depth**: Implement multiple security layers
2. **Regular Updates**: Keep Nginx and modules updated
3. **Monitoring**: Implement comprehensive security monitoring
4. **Testing**: Regular security testing
5. **Documentation**: Maintain security configuration documentation

---

**Next**: [Performance](PERFORMANCE.md) for optimization strategies.

