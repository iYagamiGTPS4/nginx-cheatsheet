# Advanced Load Balancing

Enterprise-level load balancing configurations for high-traffic applications.

## Load Balancing Algorithms

### Round Robin (Default)
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### Weighted Round Robin
```nginx
upstream backend {
    server 192.168.1.10:8080 weight=3;  # 3x more requests
    server 192.168.1.11:8080 weight=2;  # 2x more requests
    server 192.168.1.12:8080 weight=1;  # Standard weight
}
```

### Least Connections
```nginx
upstream backend {
    least_conn;                         # Route to server with fewest connections
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### IP Hash (Sticky Sessions)
```nginx
upstream backend {
    ip_hash;                           # Sticky sessions by client IP
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### Consistent Hash
```nginx
upstream backend {
    hash $request_uri consistent;       # Consistent hashing by URI
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

## Health Checks

### Passive Health Checks
```nginx
upstream backend {
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:8080 max_fails=3 fail_timeout=30s;
}
```

### Active Health Checks (Nginx Plus)
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

# Health check configuration
match server_ok {
    status 200;
    header Content-Type ~ "application/json";
    body ~ '"status":"ok"';
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        health_check match=server_ok interval=10s fails=3 passes=2;
    }
}
```

## Advanced Server Configuration

### Slow Start
```nginx
upstream backend {
    server 192.168.1.10:8080 slow_start=30s;  # Gradual traffic increase
    server 192.168.1.11:8080 slow_start=30s;
    server 192.168.1.12:8080;
}
```

### Backup Servers
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080 backup;          # Only used when others fail
    server 192.168.1.13:8080 backup;
}
```

### Server States
```nginx
upstream backend {
    server 192.168.1.10:8080 down;             # Mark as down
    server 192.168.1.11:8080 backup;           # Backup server
    server 192.168.1.12:8080 max_conns=100;   # Limit connections
}
```

## Dynamic Upstream Configuration

### Using Nginx Plus API
```nginx
# Enable dynamic configuration
upstream backend {
    zone backend 64k;                          # Shared memory zone
    server 192.168.1.10:8080;
}

# API endpoint for dynamic configuration
location /upstream_conf {
    upstream_conf;
    allow 127.0.0.1;
    deny all;
}
```

### Using Consul Integration
```nginx
# Consul service discovery
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

# Dynamic upstream with Consul
location /api/ {
    proxy_pass http://backend;
    proxy_set_header Host $host;
    proxy_set_header X-Consul-Service backend;
}
```

## Session Persistence

### Cookie-Based Sticky Sessions
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        
        # Set sticky cookie
        proxy_cookie_path / "/; HttpOnly; Secure; SameSite=Strict";
        proxy_cookie_domain localhost backend;
    }
}
```

### Application-Level Session Affinity
```nginx
# Route based on session ID
map $cookie_sessionid $backend_pool {
    default backend;
    ~^session1 backend1;
    ~^session2 backend2;
}

upstream backend1 {
    server 192.168.1.10:8080;
}

upstream backend2 {
    server 192.168.1.11:8080;
}

upstream backend {
    server 192.168.1.12:8080;
}
```

## Load Balancing Strategies

### Geographic Load Balancing
```nginx
# Route based on client location
geo $client_country {
    default US;
    192.168.1.0/24 US;
    10.0.0.0/8 US;
    172.16.0.0/12 EU;
}

map $client_country $backend_pool {
    default backend_us;
    US backend_us;
    EU backend_eu;
}

upstream backend_us {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

upstream backend_eu {
    server 192.168.2.10:8080;
    server 192.168.2.11:8080;
}
```

### A/B Testing Load Balancing
```nginx
# Split traffic for A/B testing
split_clients "${remote_addr}${http_user_agent}" $variant {
    50% "A";
    50% "B";
}

map $variant $backend_pool {
    default backend_a;
    A backend_a;
    B backend_b;
}

upstream backend_a {
    server 192.168.1.10:8080;
}

upstream backend_b {
    server 192.168.1.11:8080;
}
```

## Monitoring and Metrics

### Custom Load Balancing Metrics
```nginx
# Log upstream metrics
log_format upstream_log '$remote_addr - $upstream_addr [$time_local] '
                       '"$request" $upstream_status $upstream_response_time '
                       '$upstream_connect_time $upstream_header_time';

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        access_log /var/log/nginx/upstream.log upstream_log;
    }
}
```

### Health Check Endpoints
```nginx
# Health check for load balancer
location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
}

# Detailed health status
location /status {
    stub_status on;
    access_log off;
    allow 127.0.0.1;
    deny all;
}
```

## Performance Optimization

### Connection Pooling
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Connection pooling
    keepalive 32;                          # Keep 32 idle connections
    keepalive_requests 100;                # Max requests per connection
    keepalive_timeout 60s;                 # Connection timeout
}
```

### Buffer Optimization
```nginx
server {
    listen 80;
    
    # Optimize proxy buffers
    proxy_buffering on;
    proxy_buffer_size 4k;
    proxy_buffers 8 4k;
    proxy_busy_buffers_size 8k;
    proxy_temp_file_write_size 8k;
    
    location / {
        proxy_pass http://backend;
    }
}
```

## Common Patterns

### Blue-Green Deployment
```nginx
upstream blue {
    server 192.168.1.10:8080;
}

upstream green {
    server 192.168.1.11:8080;
}

# Route based on deployment phase
map $cookie_deployment $backend_pool {
    default blue;
    blue blue;
    green green;
}
```

### Canary Deployment
```nginx
upstream production {
    server 192.168.1.10:8080 weight=90;   # 90% traffic
}

upstream canary {
    server 192.168.1.11:8080 weight=10;  # 10% traffic
}

upstream backend {
    server 192.168.1.10:8080 weight=90;
    server 192.168.1.11:8080 weight=10;
}
```

## Troubleshooting

### Debug Upstream Issues
```nginx
# Add debug headers
add_header X-Upstream-Addr $upstream_addr always;
add_header X-Upstream-Status $upstream_status always;
add_header X-Upstream-Response-Time $upstream_response_time always;
```

### Monitor Connection States
```bash
# Check upstream connections
curl -H "Host: example.com" http://localhost/status

# Monitor upstream health
watch -n 1 'curl -s http://localhost/status | grep -E "(active|reading|writing|waiting)"'
```

---

**Next**: [High Availability](HIGH-AVAILABILITY.md) for failover and redundancy configurations.

