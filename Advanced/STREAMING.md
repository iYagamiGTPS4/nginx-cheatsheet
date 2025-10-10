# Streaming

Enterprise-level streaming configurations for media delivery.

## RTMP Streaming

### RTMP Server Configuration
```nginx
# RTMP configuration
rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        
        application live {
            live on;
            record off;
            
            # HLS configuration
            hls on;
            hls_path /var/www/hls;
            hls_fragment 3;
            hls_playlist_length 60;
            
            # DASH configuration
            dash on;
            dash_path /var/www/dash;
            dash_fragment 3;
            dash_playlist_length 60;
        }
        
        application record {
            live on;
            record all;
            record_path /var/www/recordings;
            record_unique on;
            record_suffix .flv;
        }
    }
}
```

### RTMP with Authentication
```nginx
# RTMP with authentication
rtmp {
    server {
        listen 1935;
        
        application live {
            live on;
            record off;
            
            # Authentication
            on_publish http://auth-server/rtmp-auth;
            on_play http://auth-server/rtmp-auth;
            
            # HLS configuration
            hls on;
            hls_path /var/www/hls;
            hls_fragment 3;
            hls_playlist_length 60;
        }
    }
}
```

## HLS (HTTP Live Streaming)

### HLS Configuration
```nginx
# HLS streaming
server {
    listen 80;
    server_name stream.example.com;
    
    location /hls/ {
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    location /hls/playlist.m3u8 {
        alias /var/www/hls/playlist.m3u8;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
}
```

### HLS with Authentication
```nginx
# HLS with authentication
server {
    listen 80;
    server_name stream.example.com;
    
    location /hls/ {
        # Authentication
        auth_request /auth;
        auth_request_set $user $upstream_http_x_user;
        
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    location = /auth {
        internal;
        proxy_pass http://auth-server/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

## DASH (Dynamic Adaptive Streaming)

### DASH Configuration
```nginx
# DASH streaming
server {
    listen 80;
    server_name stream.example.com;
    
    location /dash/ {
        alias /var/www/dash/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    location /dash/manifest.mpd {
        alias /var/www/dash/manifest.mpd;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
}
```

### DASH with Multiple Bitrates
```nginx
# DASH with multiple bitrates
server {
    listen 80;
    server_name stream.example.com;
    
    location /dash/ {
        alias /var/www/dash/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    # Different bitrate streams
    location /dash/1080p/ {
        alias /var/www/dash/1080p/;
        add_header Cache-Control no-cache;
    }
    
    location /dash/720p/ {
        alias /var/www/dash/720p/;
        add_header Cache-Control no-cache;
    }
    
    location /dash/480p/ {
        alias /var/www/dash/480p/;
        add_header Cache-Control no-cache;
    }
}
```

## Live Streaming

### Live Stream Configuration
```nginx
# Live streaming setup
rtmp {
    server {
        listen 1935;
        
        application live {
            live on;
            record off;
            
            # HLS for live streaming
            hls on;
            hls_path /var/www/hls;
            hls_fragment 3;
            hls_playlist_length 60;
            hls_continuous on;
            
            # DASH for live streaming
            dash on;
            dash_path /var/www/dash;
            dash_fragment 3;
            dash_playlist_length 60;
            dash_nested on;
        }
    }
}

# HTTP server for HLS/DASH
server {
    listen 80;
    server_name stream.example.com;
    
    location /hls/ {
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    location /dash/ {
        alias /var/www/dash/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
}
```

### Live Stream with Authentication
```nginx
# Live stream with authentication
server {
    listen 80;
    server_name stream.example.com;
    
    location /hls/ {
        # Authentication for live streams
        auth_request /auth;
        auth_request_set $user $upstream_http_x_user;
        
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    location = /auth {
        internal;
        proxy_pass http://auth-server/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

## VOD (Video on Demand)

### VOD Configuration
```nginx
# VOD streaming
server {
    listen 80;
    server_name vod.example.com;
    
    location /vod/ {
        alias /var/www/vod/;
        add_header Cache-Control public;
        add_header Access-Control-Allow-Origin *;
        
        # Enable range requests for seeking
        add_header Accept-Ranges bytes;
    }
    
    # HLS VOD
    location /vod/hls/ {
        alias /var/www/vod/hls/;
        add_header Cache-Control public;
        add_header Access-Control-Allow-Origin *;
    }
    
    # DASH VOD
    location /vod/dash/ {
        alias /var/www/vod/dash/;
        add_header Cache-Control public;
        add_header Access-Control-Allow-Origin *;
    }
}
```

### VOD with Authentication
```nginx
# VOD with authentication
server {
    listen 80;
    server_name vod.example.com;
    
    location /vod/ {
        # Authentication for VOD
        auth_request /auth;
        auth_request_set $user $upstream_http_x_user;
        
        alias /var/www/vod/;
        add_header Cache-Control public;
        add_header Access-Control-Allow-Origin *;
        add_header Accept-Ranges bytes;
    }
    
    location = /auth {
        internal;
        proxy_pass http://auth-server/validate;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

## Stream Authentication

### Token-Based Authentication
```nginx
# Token-based stream authentication
server {
    listen 80;
    server_name stream.example.com;
    
    location /hls/ {
        # Validate token
        auth_request /validate-token;
        auth_request_set $user $upstream_http_x_user;
        auth_request_set $expires $upstream_http_x_expires;
        
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    location = /validate-token {
        internal;
        proxy_pass http://auth-server/validate-token;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Token $arg_token;
    }
}
```

### IP-Based Authentication
```nginx
# IP-based stream authentication
server {
    listen 80;
    server_name stream.example.com;
    
    location /hls/ {
        # Allow specific IPs
        allow 192.168.1.0/24;
        allow 10.0.0.0/8;
        deny all;
        
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
}
```

## Bandwidth Control

### Rate Limiting for Streams
```nginx
# Rate limiting for streams
server {
    listen 80;
    server_name stream.example.com;
    
    location /hls/ {
        # Rate limiting
        limit_rate 1m;  # 1MB/s per connection
        limit_conn one 1;  # 1 connection per IP
        
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
}
```

### Adaptive Bitrate Streaming
```nginx
# Adaptive bitrate streaming
server {
    listen 80;
    server_name stream.example.com;
    
    # High quality stream
    location /hls/1080p/ {
        limit_rate 5m;  # 5MB/s for 1080p
        alias /var/www/hls/1080p/;
        add_header Cache-Control no-cache;
    }
    
    # Medium quality stream
    location /hls/720p/ {
        limit_rate 3m;  # 3MB/s for 720p
        alias /var/www/hls/720p/;
        add_header Cache-Control no-cache;
    }
    
    # Low quality stream
    location /hls/480p/ {
        limit_rate 1m;  # 1MB/s for 480p
        alias /var/www/hls/480p/;
        add_header Cache-Control no-cache;
    }
}
```

## Real-World Streaming Patterns

### Live Event Streaming
```nginx
# Live event streaming
rtmp {
    server {
        listen 1935;
        
        application live {
            live on;
            record off;
            
            # HLS for live events
            hls on;
            hls_path /var/www/hls;
            hls_fragment 3;
            hls_playlist_length 60;
            hls_continuous on;
            
            # DASH for live events
            dash on;
            dash_path /var/www/dash;
            dash_fragment 3;
            dash_playlist_length 60;
            dash_nested on;
        }
    }
}

# HTTP server for live events
server {
    listen 80;
    server_name live.example.com;
    
    location /hls/ {
        alias /var/www/hls/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
    
    location /dash/ {
        alias /var/www/dash/;
        add_header Cache-Control no-cache;
        add_header Access-Control-Allow-Origin *;
    }
}
```

### Educational Content Streaming
```nginx
# Educational content streaming
server {
    listen 80;
    server_name education.example.com;
    
    location /courses/ {
        # Authentication for educational content
        auth_request /auth;
        auth_request_set $user $upstream_http_x_user;
        auth_request_set $course $upstream_http_x_course;
        
        alias /var/www/courses/;
        add_header Cache-Control public;
        add_header Access-Control-Allow-Origin *;
        add_header Accept-Ranges bytes;
    }
    
    location = /auth {
        internal;
        proxy_pass http://auth-server/validate-course;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
        proxy_set_header X-Original-URI $request_uri;
    }
}
```

## Best Practices

1. **CDN Integration**: Use CDN for global distribution
2. **Authentication**: Implement proper stream authentication
3. **Monitoring**: Monitor stream quality and performance
4. **Caching**: Implement appropriate caching strategies
5. **Security**: Protect against unauthorized access

---

**Back to**: [Advanced Topics](README.md) for more enterprise configurations.
