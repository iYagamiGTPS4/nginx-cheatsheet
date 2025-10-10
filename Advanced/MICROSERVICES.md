# Microservices

Enterprise-level microservices configurations for modern architectures.

## Service Mesh Integration

### Istio Integration
```nginx
# Istio sidecar configuration
server {
    listen 80;
    server_name example.com;
    
    # Trust Istio headers
    real_ip_header X-Forwarded-For;
    set_real_ip_from 10.0.0.0/8;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Envoy Proxy Integration
```nginx
# Envoy proxy integration
server {
    listen 80;
    server_name example.com;
    
    # Trust Envoy headers
    real_ip_header X-Forwarded-For;
    set_real_ip_from 10.0.0.0/8;
    
    location / {
        proxy_pass http://backend;
    }
}
```

## API Gateway Patterns

### Service Discovery
```nginx
# Service discovery configuration
upstream auth_service {
    server auth-service:8080;
    server auth-service-2:8080;
}

upstream user_service {
    server user-service:8080;
    server user-service-2:8080;
}

upstream order_service {
    server order-service:8080;
    server order-service-2:8080;
}

# API gateway
server {
    listen 80;
    server_name api.example.com;
    
    # Authentication service
    location /auth/ {
        proxy_pass http://auth_service/;
        proxy_set_header X-Service auth;
    }
    
    # User service
    location /users/ {
        proxy_pass http://user_service/;
        proxy_set_header X-Service user;
    }
    
    # Order service
    location /orders/ {
        proxy_pass http://order_service/;
        proxy_set_header X-Service order;
    }
}
```

### Dynamic Service Discovery
```nginx
# Dynamic service discovery with Consul
upstream auth_service {
    server auth-service:8080;
}

upstream user_service {
    server user-service:8080;
}

# Service discovery endpoint
location /discover {
    internal;
    proxy_pass http://consul:8500/v1/catalog/service/$service_name;
}
```

## Circuit Breaker Patterns

### Circuit Breaker Implementation
```nginx
# Circuit breaker configuration
upstream backend {
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:8080 backup;
}

server {
    location /api/ {
        proxy_pass http://backend;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
        proxy_next_upstream_timeout 10s;
        
        # Circuit breaker fallback
        error_page 502 503 504 = @fallback;
    }
    
    location @fallback {
        return 503 "Service temporarily unavailable";
    }
}
```

### Retry Logic
```nginx
# Retry configuration
server {
    location /api/ {
        proxy_pass http://backend;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
        proxy_next_upstream_timeout 10s;
        proxy_connect_timeout 5s;
        proxy_send_timeout 5s;
        proxy_read_timeout 5s;
    }
}
```

## Request Routing Strategies

### Header-Based Routing
```nginx
# Route based on headers
map $http_x_service $backend_pool {
    default backend;
    auth auth_service;
    user user_service;
    order order_service;
}

upstream auth_service {
    server auth-service:8080;
}

upstream user_service {
    server user-service:8080;
}

upstream order_service {
    server order-service:8080;
}

upstream backend {
    server default-service:8080;
}

server {
    location / {
        proxy_pass http://$backend_pool;
    }
}
```

### Path-Based Routing
```nginx
# Path-based routing
server {
    listen 80;
    server_name api.example.com;
    
    # API versioning
    location /api/v1/ {
        proxy_pass http://v1-backend/;
    }
    
    location /api/v2/ {
        proxy_pass http://v2-backend/;
    }
    
    # Service-specific routing
    location /api/auth/ {
        proxy_pass http://auth-service/;
    }
    
    location /api/users/ {
        proxy_pass http://user-service/;
    }
}
```

## gRPC Proxying

### gRPC Load Balancing
```nginx
# gRPC load balancing
upstream grpc_backend {
    server 192.168.1.10:50051;
    server 192.168.1.11:50051;
    server 192.168.1.12:50051;
}

server {
    listen 443 ssl http2;
    server_name grpc.example.com;
    
    ssl_certificate /path/to/cert.crt;
    ssl_certificate_key /path/to/key.key;
    
    location / {
        grpc_pass grpc://grpc_backend;
        grpc_set_header Host $host;
        grpc_set_header X-Real-IP $remote_addr;
    }
}
```

### gRPC Health Checks
```nginx
# gRPC health checks
server {
    location /grpc.health.v1.Health/Check {
        grpc_pass grpc://grpc_backend;
    }
}
```

## WebSocket at Scale

### WebSocket Load Balancing
```nginx
# WebSocket load balancing
upstream websocket_backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    location /ws/ {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }
}
```

### WebSocket Sticky Sessions
```nginx
# WebSocket sticky sessions
upstream websocket_backend {
    ip_hash;  # Sticky sessions by IP
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

## Service Communication

### Inter-Service Communication
```nginx
# Inter-service communication
server {
    listen 80;
    server_name internal.example.com;
    
    # Service-to-service communication
    location /internal/ {
        proxy_pass http://backend;
        proxy_set_header X-Internal-Request "true";
        proxy_set_header X-Service-Name $service_name;
    }
}
```

### Service Mesh Headers
```nginx
# Service mesh headers
server {
    location / {
        proxy_pass http://backend;
        proxy_set_header X-Service-Mesh "nginx";
        proxy_set_header X-Trace-Id $trace_id;
        proxy_set_header X-Span-Id $span_id;
    }
}
```

## Monitoring and Observability

### Distributed Tracing
```nginx
# Distributed tracing
server {
    location / {
        proxy_pass http://backend;
        proxy_set_header X-Trace-Id $trace_id;
        proxy_set_header X-Span-Id $span_id;
        proxy_set_header X-Parent-Span-Id $parent_span_id;
    }
}
```

### Service Metrics
```nginx
# Service metrics
log_format service_metrics '$remote_addr - [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for" '
                          '$request_time $upstream_response_time '
                          '$service_name $service_version';

server {
    location / {
        access_log /var/log/nginx/service_metrics.log service_metrics;
        proxy_pass http://backend;
    }
}
```

## Real-World Patterns

### E-commerce Microservices
```nginx
# E-commerce microservices
upstream catalog_service {
    server catalog-service:8080;
}

upstream inventory_service {
    server inventory-service:8080;
}

upstream order_service {
    server order-service:8080;
}

upstream payment_service {
    server payment-service:8080;
}

server {
    listen 80;
    server_name shop.example.com;
    
    # Product catalog
    location /api/products/ {
        proxy_pass http://catalog_service/;
    }
    
    # Inventory management
    location /api/inventory/ {
        proxy_pass http://inventory_service/;
    }
    
    # Order processing
    location /api/orders/ {
        proxy_pass http://order_service/;
    }
    
    # Payment processing
    location /api/payments/ {
        proxy_pass http://payment_service/;
    }
}
```

### API Gateway with Authentication
```nginx
# API gateway with authentication
server {
    listen 80;
    server_name api.example.com;
    
    # Authentication service
    location /auth/ {
        proxy_pass http://auth-service/;
    }
    
    # Protected endpoints
    location /api/ {
        # Validate JWT token
        auth_request /validate-token;
        auth_request_set $user $upstream_http_x_user;
        auth_request_set $scope $upstream_http_x_scope;
        
        proxy_set_header X-User $user;
        proxy_set_header X-Scope $scope;
        proxy_pass http://backend;
    }
    
    # Token validation
    location = /validate-token {
        internal;
        proxy_pass http://auth-service/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

## Best Practices

1. **Service Discovery**: Use consistent service discovery
2. **Circuit Breakers**: Implement circuit breaker patterns
3. **Monitoring**: Monitor all services
4. **Security**: Implement proper authentication
5. **Documentation**: Maintain service documentation

---

**Next**: [Kubernetes](KUBERNETES.md) for container orchestration configurations.
