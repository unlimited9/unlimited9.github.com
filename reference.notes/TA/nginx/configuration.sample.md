# Configuration Sample

## nginx.conf

#### 1. default
```
#user nginx app;  
worker_processes 1;

#error_log logs/error.log;  
#error_log logs/error.log notice;  
#error_log logs/error.log info;

pid logs/nginx.pid;

events {  
    worker_connections 1024;
}

http {  
    
    ####################
    # Basic Settings
    ####################
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;
    
    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;
    
    include mime.types;
    default_type application/octet-stream;
    
    ####################
    # Logging Settings
    ####################
    access_log /storage/logs/nginx/access.log;
    error_log /storage/logs/nginx/error.log;
    
    ####################
    # Gzip Settings
    ####################
    gzip on;
    gzip_disable "msie6";
    
    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    
    ####################
    # nginx-naxsi config : Uncomment it if you installed nginx-naxsi
    ####################
    #include /etc/nginx/naxsi_core.rules;
    
    ####################
    # nginx-passenger config : Uncomment it if you installed nginx-passenger
    ####################
    #passenger_root /usr;
    #passenger_ruby /usr/bin/ruby;
    
    ####################
    # Virtual Host Configs
    ####################
    include conf.d/*.conf;
    include sites-enabled/*;
    
}
```

#### 2. Configuring NGINX for Maximum Throughput Under High Concurrency
```
user web;

# One worker process per CPU core.
worker_processes 8;

# Also set
# /etc/security/limits.conf
# web soft nofile 65535
# web hard nofile 65535
# /etc/default/nginx
# ULIMIT="-n 65535"
worker_rlimit_nofile 65535;

pid /run/nginx.pid;

events {
    #
    # Determines how many clients will be served by each worker process.
    # (Max clients = worker_connections * worker_processes)
    # Should be equal to `ulimit -n`
    #
    worker_connections 65535;
    
    #
    # Let each process accept multiple connections.
    # Accept as many connections as possible, after nginx gets notification
    # about a new connection.
    # May flood worker_connections, if that option is set too low.
    #
    multi_accept on;
    
    #
    # Preferred connection method for newer linux versions.
    # Essential for linux, optmized to serve many clients with each thread.
    #
    use epoll;
    
}

http {
    ##
    # Basic Settings
    ##
    
    #
    # Override some buffer limitations, will prevent DDOS too.
    #
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 8m;
    large_client_header_buffers 2 1k;
    
    #
    # Timeouts
    # The client_body_timeout and client_header_timeout directives are
    # responsible for the time a server will wait for a client body or
    # client header to be sent after request. If neither a body or header
    # is sent, the server will issue a 408 error or Request time out.
    #
    # The keepalive_timeout assigns the timeout for keep-alive connections
    # with the client. Simply put, Nginx will close connections with the
    # client after this period of time.
    #
    # Finally, the send_timeout is a timeout for transmitting a response
    # to the client. If the client does not receive anything within this
    # time, then the connection will be closed.
    #
    
    #
    # send the client a "request timed out" if the body is not loaded
    # by this time. Default 60.
    #
    client_body_timeout 32;
    client_header_timeout 32;
    
    #
    # Every 60 seconds server broadcasts Sync packets, so 90 is
    # a conservative upper bound.
    #
    keepalive_timeout 90; # default 65
    send_timeout 120; # default 60
    
    #
    # Allow the server to close the connection after a client stops
    # responding.
    # Frees up socket-associated memory.
    #
    reset_timedout_connection on;
    
    #
    # Open file descriptors.
    # Caches information about open FDs, freqently accessed files.
    #
    open_file_cache max=200000 inactive=20s;
    open_file_cache_valid 30s;
    open_file_cache_min_uses 2;
    open_file_cache_errors on;
    
    #
    # Sendfile copies data between one FD and other from within the kernel.
    # More efficient than read() + write(), since the requires transferring
    # data to and from the user space.
    #
    sendfile on;
    
    # Tcp_nopush causes nginx to attempt to send its HTTP response head in one
    # packet, instead of using partial frames. This is useful for prepending
    # headers before calling sendfile, or for throughput optimization.
    tcp_nopush on;
    
    #
    # don't buffer data-sends (disable Nagle algorithm). Good for sending
    # frequent small bursts of data in real time.
    #
    tcp_nodelay on;
    
    types_hash_max_size 2048;
    
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    ##
    # Logging Settings
    ##
    
    #
    # Use analytics to track stuff instead of using precious file IO resources.
    # Disabling logging speeds up IO.
    #
    access_log off;
    error_log /root/PROJECTS/logs/error.log crit;
    
    ##
    # Gzip Settings
    ##
    
    gzip on;
    gzip_disable "MSIE [1-6]\.";
    
    # Only allow proxy request with these headers to be gzipped.
    gzip_proxied expired no-cache no-store private auth;
    
    # Default is 6 (1<n<9), but 2 -- even 1 -- is enough. The higher it is, the
    # more CPU cycles will be wasted.
    gzip_comp_level 9;
    gzip_min_length 500; # Default 20
    
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
    
    ##
    # Virtual Host Configs
    ##
    
    include /etc/nginx/conf.d/*.conf;
    
}
```

## sites-available

#### product.mobon.net.conf
```
upstream tomcat {
    #LB method : least_conn, ip_hash  
    #ip_hash;
    
    ## proxy server  
    server 172.20.0.103:15080;
    server 172.20.0.103:16080;
    server 172.20.0.103:17080;
    server 172.20.0.103:19080;
    server 172.20.0.103:20080;
    server 172.20.0.103:21080;

    keepalive 4096;
}  

server {  
    listen 99;  
    server_name product.mobon.net;
    
    access_log /logs/nginx/product.mobon.net_access.log;

#    location / {  
#        root /pgms/www;  
#        index index.html index.htm;  
#    }
    
    # redirect server error pages to the static page /50x.html  
    error_page 500 502 503 504 /50x.html;  
    location = /50x.html {
        root html;
    }
    
    location / {
        #proxy_pass http://127.0.0.1:8080;
        proxy_pass http://tomcat;
        proxy_redirect off;

        proxy_http_version  1.1;

        proxy_set_header Host $http_host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        proxy_set_header X-Forwarded-Proto $scheme;  
        proxy_set_header X-NginX-Proxy true;
        
        charset utf-8;
        
        index index.jsp index.html;
        
        if ($request_filename ~* ^.*?/([^/]*?)$) {
            set $filename $1;
        }

        if ($filename ~* ^.*?\.(eot)|(ttf)|(woff)$) {
            add_header Access-Control-Allow-Origin *;
        }
    
    }
    
}
```

#### tracker.mobon.net.conf
```
upstream tomcat {
    #LB method : least_conn, ip_hash  
    #ip_hash;
    
    ## proxy server  
    server 172.20.0.103:15080;
    server 172.20.0.103:16080;
    server 172.20.0.103:17080;
    server 172.20.0.103:19080;
    server 172.20.0.103:20080;
    server 172.20.0.103:21080;

    keepalive 4096;
}  

server {  
    listen 99;  
    server_name product.mobon.net;
    
    access_log /logs/nginx/product.mobon.net_access.log;

#    location / {  
#        root /pgms/www;  
#        index index.html index.htm;  
#    }
    
    # redirect server error pages to the static page /50x.html  
    error_page 500 502 503 504 /50x.html;  
    location = /50x.html {
        root html;
    }
    
    location / {
        #proxy_pass http://127.0.0.1:8080;
        proxy_pass http://tomcat;
        proxy_redirect off;

        proxy_http_version  1.1;

        proxy_set_header Host $http_host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        proxy_set_header X-Forwarded-Proto $scheme;  
        proxy_set_header X-NginX-Proxy true;
        
        charset utf-8;
        
        index index.jsp index.html;
        
        if ($request_filename ~* ^.*?/([^/]*?)$) {
            set $filename $1;
        }

        if ($filename ~* ^.*?\.(eot)|(ttf)|(woff)$) {
            add_header Access-Control-Allow-Origin *;
        }
    
    }
    
}
```
