# High Availability

Enterprise-level high availability configurations for mission-critical applications.

## Active-Passive Setup

### Keepalived Configuration
```bash
# /etc/keepalived/keepalived.conf
vrrp_script chk_nginx {
    script "/usr/bin/curl -f http://localhost/health || exit 1"
    interval 2
    weight -2
    fall 3
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1234
    }
    virtual_ipaddress {
        192.168.1.100
    }
    track_script {
        chk_nginx
    }
}
```

### Nginx Health Check
```nginx
# Health check endpoint
server {
    listen 80;
    server_name _;
    
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

## Active-Active Setup

### Multi-Master Configuration
```nginx
# Primary server
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### Load Balancer Configuration
```nginx
# Load balancer for active-active
upstream nginx_cluster {
    server 192.168.1.10:80;
    server 192.168.1.11:80;
    server 192.168.1.12:80;
}

server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://nginx_cluster;
    }
}
```

## Failover Configuration

### Automatic Failover
```nginx
# Failover configuration
upstream backend {
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:8080 backup;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_next_upstream_tries 3;
        proxy_next_upstream_timeout 10s;
    }
}
```

### Health Check Integration
```nginx
# Health check configuration
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

# Health check endpoint
location /health {
    access_log off;
    return 200 "healthy\n";
    add_header Content-Type text/plain;
}

# Backend health check
location /backend-health {
    proxy_pass http://backend/health;
    proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
}
```

## Cloud Load Balancer Integration

### AWS Application Load Balancer
```nginx
# ALB integration
server {
    listen 80;
    server_name example.com;
    
    # Trust ALB headers
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

### Google Cloud Load Balancer
```nginx
# GCP load balancer integration
server {
    listen 80;
    server_name example.com;
    
    # Trust GCP headers
    real_ip_header X-Forwarded-For;
    set_real_ip_from 10.0.0.0/8;
    set_real_ip_from 35.191.0.0/16;
    
    location / {
        proxy_pass http://backend;
    }
}
```

## HAProxy + Nginx

### HAProxy Configuration
```bash
# /etc/haproxy/haproxy.cfg
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http_front
    bind *:80
    default_backend nginx_servers

backend nginx_servers
    balance roundrobin
    option httpchk GET /health
    server nginx1 192.168.1.10:80 check
    server nginx2 192.168.1.11:80 check
    server nginx3 192.168.1.12:80 check
```

### Nginx Configuration
```nginx
# Nginx behind HAProxy
server {
    listen 80;
    server_name example.com;
    
    # Trust HAProxy
    real_ip_header X-Forwarded-For;
    set_real_ip_from 192.168.1.0/24;
    
    location / {
        proxy_pass http://backend;
    }
}
```

## Database High Availability

### Database Load Balancing
```nginx
# Database load balancing
upstream database {
    server 192.168.1.20:3306 max_fails=3 fail_timeout=30s;
    server 192.168.1.21:3306 max_fails=3 fail_timeout=30s;
    server 192.168.1.22:3306 backup;
}

stream {
    server {
        listen 3306;
        proxy_pass database;
        proxy_timeout 1s;
        proxy_responses 1;
    }
}
```

### Redis High Availability
```nginx
# Redis load balancing
upstream redis {
    server 192.168.1.30:6379 max_fails=3 fail_timeout=30s;
    server 192.168.1.31:6379 max_fails=3 fail_timeout=30s;
    server 192.168.1.32:6379 backup;
}

stream {
    server {
        listen 6379;
        proxy_pass redis;
        proxy_timeout 1s;
        proxy_responses 1;
    }
}
```

## Monitoring and Alerting

### Health Check Monitoring
```nginx
# Comprehensive health checks
server {
    listen 80;
    server_name monitoring.example.com;
    
    # Basic health check
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
    
    # Detailed health check
    location /health/detailed {
        access_log off;
        return 200 "nginx_up 1\nnginx_connections_active $connections_active\nnginx_connections_reading $connections_reading\nnginx_connections_writing $connections_writing\nnginx_connections_waiting $connections_waiting\n";
        add_header Content-Type text/plain;
    }
    
    # Backend health check
    location /health/backend {
        proxy_pass http://backend/health;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    }
}
```

### Alerting Configuration
```nginx
# Alerting endpoint
location /alerts {
    internal;
    return 200 "Alert: $alert_message\n";
    add_header Content-Type text/plain;
}
```

## Disaster Recovery

### Backup Configuration
```nginx
# Backup server configuration
upstream primary {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

upstream backup {
    server 192.168.2.10:8080;
    server 192.168.2.11:8080;
}

# Route based on availability
map $upstream_status $backend_pool {
    default primary;
    ~^[45] backup;
}

server {
    location / {
        proxy_pass http://$backend_pool;
    }
}
```

### Geographic Failover
```nginx
# Geographic failover
geo $client_country {
    default US;
    192.168.1.0/24 US;
    10.0.0.0/8 US;
    172.16.0.0/12 EU;
}

map $client_country $backend_pool {
    default primary_us;
    US primary_us;
    EU primary_eu;
}

upstream primary_us {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

upstream primary_eu {
    server 192.168.3.10:8080;
    server 192.168.3.11:8080;
}
```

## Best Practices

1. **Health Checks**: Implement comprehensive health checks
2. **Monitoring**: Monitor all components continuously
3. **Testing**: Regular failover testing
4. **Documentation**: Maintain detailed runbooks
5. **Automation**: Automate failover procedures

---

**Next**: [Microservices](MICROSERVICES.md) for service mesh and API gateway patterns.
