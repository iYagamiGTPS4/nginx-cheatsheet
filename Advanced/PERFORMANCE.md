# Advanced Performance Optimization

Enterprise-level performance configurations for high-traffic applications.

## HTTP/2 and HTTP/3 Optimization

### HTTP/2 Configuration
```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # HTTP/2 optimizations
    http2_max_field_size 4k;                 # Max header field size
    http2_max_header_size 16k;               # Max header size
    http2_body_preread_size 64k;             # Body preread size
    http2_idle_timeout 3m;                   # Idle timeout
    http2_recv_timeout 4m;                   # Receive timeout
    
    # SSL optimization for HTTP/2
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### HTTP/3 Configuration (Nginx Plus)
```nginx
server {
    listen 443 ssl http3;
    server_name example.com;
    
    # HTTP/3 optimizations
    quic_retry on;                           # QUIC retry mechanism
    quic_gso on;                             # Generic Segmentation Offload
    quic_host_key /path/to/host.key;        # Host key for QUIC
    
    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;
    
    location / {
        proxy_pass http://backend;
    }
}
```

## Connection Pooling

### Upstream Connection Pooling
```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    
    # Connection pooling
    keepalive 32;                          # Keep 32 idle connections
    keepalive_requests 100;                # Max requests per connection
    keepalive_timeout 60s;                 # Connection timeout
    
    # Health checks
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;            # Required for keepalive
        proxy_set_header Connection "";
    }
}
```

### FastCGI Connection Pooling
```nginx
# FastCGI connection pooling
upstream php_backend {
    server 127.0.0.1:9000;
    keepalive 32;
    keepalive_requests 100;
    keepalive_timeout 60s;
}

server {
    location ~ \.php$ {
        fastcgi_pass php_backend;
        fastcgi_keep_conn on;              # Enable keepalive
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## Thread Pools

### Thread Pool Configuration
```nginx
# Main context
thread_pool default threads=32 max_queue=65536;

# Use thread pool for I/O operations
server {
    listen 80;
    server_name example.com;
    
    location / {
        aio threads=default;               # Use thread pool for async I/O
        sendfile on;
        proxy_pass http://backend;
    }
}
```

### File I/O Optimization
```nginx
server {
    location / {
        # Async file I/O
        aio threads=default;
        aio_write on;
        
        # Efficient file serving
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        
        try_files $uri $uri/ /index.html;
    }
}
```

## Memory Optimization

### Worker Process Configuration
```nginx
# Main context
worker_processes auto;                     # Auto-detect CPU cores
worker_rlimit_nofile 65535;               # Max open files per worker
worker_cpu_affinity auto;                 # Bind workers to CPU cores

# Events context
events {
    worker_connections 1024;               # Connections per worker
    use epoll;                             # Efficient event method
    multi_accept on;                       # Accept multiple connections
    accept_mutex off;                      # Disable accept mutex
}
```

### Buffer Optimization
```nginx
http {
    # Client buffer optimization
    client_body_buffer_size 128k;          # Client body buffer
    client_max_body_size 10m;              # Max request body size
    client_header_buffer_size 1k;          # Client header buffer
    large_client_header_buffers 4 4k;       # Large header buffers
    
    # Proxy buffer optimization
    proxy_buffering on;                    # Enable proxy buffering
    proxy_buffer_size 4k;                  # Proxy buffer size
    proxy_buffers 8 4k;                    # Number and size of buffers
    proxy_busy_buffers_size 8k;            # Busy buffer size
    proxy_temp_file_write_size 8k;         # Temp file write size
    
    # FastCGI buffer optimization
    fastcgi_buffering on;                  # Enable FastCGI buffering
    fastcgi_buffer_size 4k;                # FastCGI buffer size
    fastcgi_buffers 8 4k;                  # FastCGI buffers
    fastcgi_busy_buffers_size 8k;          # FastCGI busy buffers
}
```

## CPU Affinity

### CPU Binding Configuration
```nginx
# Bind workers to specific CPU cores
worker_processes 4;
worker_cpu_affinity 0001 0010 0100 1000;  # Bind to cores 0,1,2,3

# Or use auto-detection
worker_processes auto;
worker_cpu_affinity auto;
```

### NUMA-Aware Configuration
```nginx
# NUMA-aware configuration for multi-socket servers
worker_processes 8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
```

## Zero-Downtime Deployments

### Graceful Reload
```bash
# Graceful reload without dropping connections
nginx -s reload

# Or using systemctl
systemctl reload nginx
```

### Blue-Green Deployment
```nginx
# Blue-Green deployment configuration
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

server {
    location / {
        proxy_pass http://$backend_pool;
    }
}
```

### Canary Deployment
```nginx
# Canary deployment with traffic splitting
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

## Advanced Compression

### Gzip Optimization
```nginx
http {
    # Gzip compression
    gzip on;                               # Enable gzip
    gzip_vary on;                          # Add Vary header
    gzip_min_length 1024;                  # Minimum file size
    gzip_comp_level 6;                     # Compression level (1-9)
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # Gzip for specific content types
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
}
```

### Brotli Compression (Nginx Plus)
```nginx
# Brotli compression
brotli on;
brotli_comp_level 6;
brotli_types text/plain text/css application/json application/javascript;
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

### Real-Time Performance Headers
```nginx
server {
    location / {
        # Add performance headers
        add_header X-Response-Time $request_time always;
        add_header X-Upstream-Response-Time $upstream_response_time always;
        add_header X-Upstream-Connect-Time $upstream_connect_time always;
        add_header X-Connection-Requests $connection_requests always;
        
        proxy_pass http://backend;
    }
}
```

## Benchmarking and Profiling

### Load Testing Configuration
```nginx
# Optimized for load testing
server {
    listen 80;
    server_name example.com;
    
    # Disable logging for load testing
    access_log off;
    error_log off;
    
    # Optimize for high concurrency
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 0;                    # Disable keepalive for testing
    
    location / {
        proxy_pass http://backend;
    }
}
```

### Performance Profiling
```nginx
# Enable debug logging for profiling
error_log /var/log/nginx/debug.log debug;

# Debug specific connections
debug_connection 192.168.1.0/24;
debug_connection 127.0.0.1;
```

## Advanced Caching for Performance

### Multi-Level Caching
```nginx
# L1: Memory cache
proxy_cache_path /var/cache/nginx/l1 levels=1:2 keys_zone=l1:100m max_size=1g;

# L2: Disk cache
proxy_cache_path /var/cache/nginx/l2 levels=1:2 keys_zone=l2:1g max_size=10g;

server {
    location / {
        # Try L1 cache first
        proxy_cache l1;
        proxy_cache_valid 200 1m;
        proxy_cache_use_stale error timeout updating;
        
        # Fallback to L2 cache
        proxy_cache l2;
        proxy_cache_valid 200 1h;
        
        proxy_pass http://backend;
    }
}
```

### Cache Warming
```nginx
# Cache warming endpoint
location /warm-cache {
    internal;                              # Only internal requests
    proxy_cache api_cache;
    proxy_cache_valid 200 10m;
    proxy_pass http://backend$request_uri;
}

# Warm cache on startup
location /warm {
    access_log off;
    return 200 "Cache warming initiated\n";
    add_header Content-Type text/plain;
}
```

## Real-World Performance Patterns

### High-Traffic E-commerce
```nginx
# E-commerce performance configuration
server {
    listen 443 ssl http2;
    server_name shop.example.com;
    
    # HTTP/2 optimizations
    http2_max_field_size 4k;
    http2_max_header_size 16k;
    
    # Performance optimizations
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    keepalive_requests 100;
    
    # Static content optimization
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
    
    # API optimization
    location /api/ {
        proxy_cache api_cache;
        proxy_cache_valid 200 10m;
        proxy_cache_use_stale error timeout updating;
        proxy_pass http://backend;
    }
}
```

### High-Concurrency API
```nginx
# API performance configuration
server {
    listen 80;
    server_name api.example.com;
    
    # Connection optimization
    keepalive_timeout 65;
    keepalive_requests 1000;
    
    # Buffer optimization
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    proxy_buffering on;
    proxy_buffer_size 4k;
    proxy_buffers 8 4k;
    
    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```

## Performance Best Practices

1. **Measure First**: Profile before optimizing
2. **Incremental Changes**: Make one change at a time
3. **Monitor Impact**: Track performance metrics
4. **Test Under Load**: Use realistic load testing
5. **Document Changes**: Keep performance change records

---

**Next**: [Monitoring](MONITORING.md) for comprehensive monitoring strategies.

