# Advanced Monitoring

Enterprise-level monitoring configurations for Nginx deployments.

## Prometheus Integration

### Nginx Prometheus Exporter
```nginx
# Enable stub_status for Prometheus
server {
    listen 80;
    server_name monitoring.example.com;
    
    location /nginx_status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        allow 192.168.1.0/24;
        deny all;
    }
}
```

### Custom Metrics Endpoint
```nginx
# Custom metrics for Prometheus
server {
    location /metrics {
        access_log off;
        return 200 "nginx_up 1\nnginx_connections_active $connections_active\nnginx_connections_reading $connections_reading\nnginx_connections_writing $connections_writing\nnginx_connections_waiting $connections_waiting\n";
        add_header Content-Type text/plain;
    }
}
```

## Real-Time Monitoring

### Connection Monitoring
```nginx
# Real-time connection monitoring
server {
    location /status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
    
    # Custom status endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### Performance Metrics
```nginx
# Performance monitoring headers
server {
    location / {
        # Add performance headers
        add_header X-Response-Time $request_time always;
        add_header X-Upstream-Response-Time $upstream_response_time always;
        add_header X-Upstream-Connect-Time $upstream_connect_time always;
        add_header X-Connection-Requests $connection_requests always;
        add_header X-Cache-Status $upstream_cache_status always;
        
        proxy_pass http://backend;
    }
}
```

## Log Aggregation

### Structured Logging
```nginx
# JSON log format for log aggregation
log_format json_combined escape=json
    '{"time":"$time_iso8601",'
    '"remote_addr":"$remote_addr",'
    '"request":"$request",'
    '"status":"$status",'
    '"body_bytes_sent":"$body_bytes_sent",'
    '"http_referer":"$http_referer",'
    '"http_user_agent":"$http_user_agent",'
    '"request_time":"$request_time",'
    '"upstream_response_time":"$upstream_response_time"}';

server {
    access_log /var/log/nginx/access.json json_combined;
}
```

### ELK Stack Integration
```nginx
# Log format optimized for Elasticsearch
log_format elk '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" "$http_x_forwarded_for" '
               '$request_time $upstream_response_time';

server {
    access_log /var/log/nginx/elk.log elk;
}
```

## Error Tracking

### Error Log Configuration
```nginx
# Detailed error logging
error_log /var/log/nginx/error.log warn;
error_log /var/log/nginx/debug.log debug;

# Debug specific connections
debug_connection 192.168.1.0/24;
debug_connection 127.0.0.1;
```

### Error Analysis
```nginx
# Error tracking endpoint
location /errors {
    internal;
    return 200 "Error count: $error_count\n";
    add_header Content-Type text/plain;
}
```

## Performance Monitoring

### Custom Performance Metrics
```nginx
# Performance monitoring
log_format performance '$remote_addr - [$time_local] "$request" '
                     '$status $request_time $upstream_response_time '
                     '$upstream_connect_time $upstream_header_time '
                     '$body_bytes_sent $connection_requests';

server {
    location / {
        access_log /var/log/nginx/performance.log performance;
        proxy_pass http://backend;
    }
}
```

### Upstream Monitoring
```nginx
# Upstream monitoring
log_format upstream '$remote_addr - $upstream_addr [$time_local] '
                   '"$request" $upstream_status $upstream_response_time '
                   '$upstream_connect_time $upstream_header_time';

server {
    location /api/ {
        proxy_pass http://backend;
        access_log /var/log/nginx/upstream.log upstream;
    }
}
```

## Security Monitoring

### Security Event Logging
```nginx
# Security monitoring
log_format security '$remote_addr - [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for" '
                   '$ssl_protocol $ssl_cipher';

server {
    location / {
        access_log /var/log/nginx/security.log security;
        proxy_pass http://backend;
    }
}
```

### Failed Authentication Tracking
```nginx
# Track authentication failures
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

## Real-World Monitoring Patterns

### E-commerce Monitoring
```nginx
# E-commerce monitoring configuration
server {
    location /checkout {
        # Track checkout performance
        access_log /var/log/nginx/checkout.log performance;
        proxy_pass http://backend;
    }
    
    location /api/orders {
        # Track order API performance
        access_log /var/log/nginx/orders.log upstream;
        proxy_pass http://backend;
    }
}
```

### API Monitoring
```nginx
# API monitoring
server {
    location /api/ {
        # Track API performance
        access_log /var/log/nginx/api.log performance;
        
        # Add monitoring headers
        add_header X-API-Response-Time $request_time always;
        add_header X-API-Upstream-Time $upstream_response_time always;
        
        proxy_pass http://backend;
    }
}
```

---

**Next**: [High Availability](HIGH-AVAILABILITY.md) for failover and redundancy configurations.
