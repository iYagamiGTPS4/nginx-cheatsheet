# Advanced Caching Strategies

Enterprise-level caching configurations for high-performance applications.

## Cache Types and Use Cases

### Proxy Cache (Reverse Proxy Caching)
```nginx
# Cache configuration
proxy_cache_path /var/cache/nginx/proxy 
    levels=1:2 
    keys_zone=proxy_cache:10m 
    max_size=1g 
    inactive=60m 
    use_temp_path=off;

server {
    listen 80;
    server_name example.com;
    
    location /api/ {
        proxy_cache proxy_cache;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        proxy_cache_use_stale error timeout updating;
        proxy_cache_key $scheme$proxy_host$request_uri;
        proxy_pass http://backend;
    }
}
```

### FastCGI Cache (PHP Applications)
```nginx
# FastCGI cache configuration
fastcgi_cache_path /var/cache/nginx/fastcgi 
    levels=1:2 
    keys_zone=php_cache:10m 
    max_size=1g 
    inactive=60m;

server {
    listen 80;
    server_name example.com;
    
    location ~ \.php$ {
        fastcgi_cache php_cache;
        fastcgi_cache_valid 200 60m;
        fastcgi_cache_valid 404 1m;
        fastcgi_cache_bypass $no_cache;
        fastcgi_no_cache $no_cache;
        fastcgi_cache_key $scheme$request_method$host$request_uri;
        fastcgi_pass 127.0.0.1:9000;
    }
}
```

## Advanced Cache Configuration

### Cache Zones with Different Policies
```nginx
# Multiple cache zones for different content types
proxy_cache_path /var/cache/nginx/static levels=1:2 keys_zone=static:100m max_size=10g;
proxy_cache_path /var/cache/nginx/api levels=1:2 keys_zone=api:50m max_size=1g;
proxy_cache_path /var/cache/nginx/dynamic levels=1:2 keys_zone=dynamic:20m max_size=500m;

server {
    listen 80;
    server_name example.com;
    
    # Static content - long cache
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg)$ {
        proxy_cache static;
        proxy_cache_valid 200 1y;
        proxy_cache_key $scheme$host$uri;
        proxy_pass http://backend;
    }
    
    # API responses - medium cache
    location /api/ {
        proxy_cache api;
        proxy_cache_valid 200 10m;
        proxy_cache_valid 404 1m;
        proxy_cache_key $scheme$host$request_uri$args;
        proxy_pass http://backend;
    }
    
    # Dynamic content - short cache
    location /dynamic/ {
        proxy_cache dynamic;
        proxy_cache_valid 200 1m;
        proxy_cache_key $scheme$host$request_uri$args;
        proxy_pass http://backend;
    }
}
```

### Cache Bypass Conditions
```nginx
# Bypass cache for specific conditions
map $request_method $no_cache {
    default 0;
    POST 1;
    PUT 1;
    DELETE 1;
}

map $http_authorization $no_cache {
    default 0;
    ~.+ 1;  # Bypass cache for authenticated requests
}

map $http_pragma $no_cache {
    default 0;
    "no-cache" 1;
}

server {
    location /api/ {
        proxy_cache api_cache;
        proxy_cache_bypass $no_cache;
        proxy_no_cache $no_cache;
        proxy_pass http://backend;
    }
}
```

## Cache Purging and Invalidation

### Manual Cache Purging
```nginx
# Cache purge endpoint
location ~ /purge(/.*) {
    proxy_cache_purge api_cache $scheme$host$1;
    access_log off;
    allow 127.0.0.1;
    allow 192.168.1.0/24;
    deny all;
}

# Usage: curl -X PURGE http://example.com/purge/api/users/123
```

### Automatic Cache Invalidation
```nginx
# Invalidate cache on content update
location /api/ {
    proxy_cache api_cache;
    proxy_cache_valid 200 10m;
    
    # Invalidate related cache entries
    proxy_cache_purge api_cache $scheme$host$request_uri;
    proxy_cache_purge api_cache $scheme$host/api/users;
    
    proxy_pass http://backend;
}
```

## Microcaching Strategies

### Short-Term Caching for Dynamic Content
```nginx
# Microcache for frequently accessed dynamic content
proxy_cache_path /var/cache/nginx/micro levels=1:2 keys_zone=micro:1m max_size=100m;

server {
    location / {
        proxy_cache micro;
        proxy_cache_valid 200 1s;  # Cache for 1 second
        proxy_cache_use_stale updating;
        proxy_cache_lock on;
        proxy_cache_lock_timeout 5s;
        proxy_pass http://backend;
    }
}
```

### Cache Warming
```nginx
# Warm cache with critical content
location /warm-cache {
    internal;  # Only internal requests
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

## Cache Lock Mechanisms

### Cache Lock for High Concurrency
```nginx
server {
    location /api/ {
        proxy_cache api_cache;
        proxy_cache_valid 200 10m;
        proxy_cache_lock on;                    # Enable cache lock
        proxy_cache_lock_timeout 5s;            # Lock timeout
        proxy_cache_lock_age 5s;                # Lock age
        proxy_cache_use_stale updating;         # Use stale while updating
        proxy_pass http://backend;
    }
}
```

### Cache Slice for Large Files
```nginx
# Cache large files in slices
proxy_cache_path /var/cache/nginx/slice levels=1:2 keys_zone=slice:10m max_size=1g;

server {
    location /large-files/ {
        proxy_cache slice;
        proxy_cache_valid 200 1h;
        proxy_cache_key $scheme$host$uri$slice_range;
        proxy_set_header Range $slice_range;
        proxy_pass http://backend;
    }
}
```

## Advanced Cache Headers

### Cache Control Headers
```nginx
server {
    location /api/ {
        proxy_cache api_cache;
        proxy_cache_valid 200 10m;
        
        # Add cache control headers
        add_header X-Cache-Status $upstream_cache_status always;
        add_header X-Cache-Key $proxy_cache_key always;
        add_header X-Cache-Valid $proxy_cache_valid always;
        
        proxy_pass http://backend;
    }
}
```

### Conditional Caching
```nginx
# Cache based on response headers
map $upstream_http_cache_control $cache_ttl {
    default 10m;
    "max-age=3600" 1h;
    "max-age=86400" 1d;
    "no-cache" 0;
}

server {
    location /api/ {
        proxy_cache api_cache;
        proxy_cache_valid 200 $cache_ttl;
        proxy_pass http://backend;
    }
}
```

## Cache Analytics and Monitoring

### Cache Hit Rate Monitoring
```nginx
# Log cache statistics
log_format cache_stats '$remote_addr - [$time_local] "$request" '
                      '$status $upstream_cache_status $upstream_response_time '
                      '$proxy_cache_key';

server {
    location /api/ {
        proxy_cache api_cache;
        access_log /var/log/nginx/cache.log cache_stats;
        proxy_pass http://backend;
    }
}
```

### Cache Performance Metrics
```nginx
# Custom cache metrics endpoint
location /cache-metrics {
    access_log off;
    return 200 "Cache Hit Rate: $upstream_cache_status\n";
    add_header Content-Type text/plain;
}
```

## Real-World Cache Patterns

### E-commerce Product Cache
```nginx
# Product pages with inventory cache
location /products/ {
    proxy_cache product_cache;
    proxy_cache_valid 200 5m;           # Short cache for inventory
    proxy_cache_valid 404 1m;           # Cache 404s briefly
    proxy_cache_key $scheme$host$request_uri$args;
    proxy_cache_use_stale error timeout updating;
    proxy_pass http://backend;
}

# Product search with longer cache
location /search {
    proxy_cache search_cache;
    proxy_cache_valid 200 30m;           # Longer cache for search results
    proxy_cache_key $scheme$host$request_uri$args;
    proxy_pass http://backend;
}
```

### API Response Caching
```nginx
# User profile cache
location /api/users/ {
    proxy_cache user_cache;
    proxy_cache_valid 200 1h;            # Cache user profiles for 1 hour
    proxy_cache_key $scheme$host$request_uri$http_authorization;
    proxy_cache_bypass $http_authorization;  # Bypass for authenticated users
    proxy_pass http://backend;
}

# Public API cache
location /api/public/ {
    proxy_cache public_cache;
    proxy_cache_valid 200 10m;          # Cache public API for 10 minutes
    proxy_cache_key $scheme$host$request_uri;
    proxy_pass http://backend;
}
```

## Cache Troubleshooting

### Debug Cache Issues
```nginx
# Add debug headers
add_header X-Cache-Status $upstream_cache_status always;
add_header X-Cache-Key $proxy_cache_key always;
add_header X-Cache-Valid $proxy_cache_valid always;
add_header X-Cache-Lock $proxy_cache_lock always;
```

### Cache Performance Analysis
```bash
# Analyze cache hit rates
awk '{print $4}' /var/log/nginx/cache.log | sort | uniq -c

# Monitor cache size
du -sh /var/cache/nginx/*

# Check cache configuration
nginx -T | grep -A 10 -B 10 cache
```

## Best Practices

1. **Cache Key Design**: Use consistent, meaningful cache keys
2. **TTL Strategy**: Set appropriate TTLs based on content type
3. **Cache Warming**: Pre-warm critical content
4. **Monitoring**: Track cache hit rates and performance
5. **Purging Strategy**: Implement efficient cache invalidation
6. **Memory Management**: Monitor cache size and cleanup

---

**Next**: [Security](SECURITY.md) for advanced security configurations.

