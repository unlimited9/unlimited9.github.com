# set HTTPS/SSL to nginx

#### copy certificates
$ mkdir -p /apps/pki/tls/certs /apps/pki/tls/private

$ cp STAR.mediacategory.com.pem /apps/pki/tls/certs/  
$ cp STAR.mediacategory.com.key /apps/pki/tls/private/

#### configuration
$ vi /apps/nginx/1.17.6/conf/sites-enabled/tk.mediacategory.com.conf
```
#upstream tk {
#    #LB method : least_conn, ip_hash  
#    ip_hash;
#    
#    ## proxy server  
#    server 10.251.0.183;
#    server 10.251.0.184;
#
#    keepalive 4096;
#}  
server {
    listen 80;
    server_name tk.mediacategory.com;
    return 301 https://tk.mediacategory.com$request_uri;
}

server {  
    listen 443 ssl;
    server_name tk.mediacategory.com;

    ssl_certificate /apps/pki/tls/certs/STAR.mediacategory.com.pem;
    ssl_certificate_key /apps/pki/tls/private/STAR.mediacategory.com.key;
    
    access_log /logs/nginx/tk.mediacategory.com_access.log;
    
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
#	rewrite ^/tk/(.*)$ /$1 break;
#	rewrite ^/(.*)$ /tk/$1 break;

        proxy_set_header Host $http_host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        proxy_set_header X-Forwarded-Proto $scheme;  
        proxy_set_header X-NginX-Proxy true;
        
        proxy_pass http://10.251.0.184/tk$uri;
        #proxy_pass http://tk;
        proxy_redirect off;  
        charset utf-8;

        index index.jsp index.html;

        # setting CORS
        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            # Custom headers and headers various browsers *should* be OK with but aren't
            add_header 'Access-Control-Allow-Headers' '*';

            # Tell client that this pre-flight info is valid for 20 days
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 204;
        }
        if ($request_method = 'POST') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' '*';
            add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        }
        if ($request_method = 'GET') {
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
            add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
        }

#        if ($request_filename ~* ^.*?/([^/]*?)$) {
#            set $filename $1;
#        }

#        if ($filename ~* ^.*?\.(eot)|(ttf)|(woff)$) {
#            add_header Access-Control-Allow-Origin *;
#        }

    }

    location ~ ^/tk/(.*)$ {
#        return 301 /$1;
#        rewrite ^/tk/(.*)$ /$1 break;
        return 308 /$1$is_args$args;
    }

}

```

## 9. Appendix

#### reference site

* nginx 에 HTTPS/SSL 적용하기  
https://www.lesstif.com/pages/viewpage.action?pageId=27984443

* Redirect HTTP to HTTPS in Nginx  
https://linuxize.com/post/redirect-http-to-https-in-nginx/
