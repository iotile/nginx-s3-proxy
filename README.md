
## Motivation

This image was created for use with dogestry. We wanted a caching HTTP proxy between our
servers and S3 so that images were only downloaded once from S3.

## Usage

The image assumes a config file in the container at: `/nginx.conf` so use the `-v` option to
mount one from your host.


```
docker run -p 8000:8000 -v /path/to/nginx.conf:/nginx.conf coopernurse/nginx-s3-proxy 
```

If you want to store the cache on the host, bind a path to `/data/cache`:

```
docker run -p 8000:8000 -v /path/to/nginx.conf:/nginx.conf -v /my/path:/data/cache coopernurse/nginx-s3-proxy 
```

Feel free to alter the `-p` param if you wish to bind the port differently onto the host.


Example nginx.conf file:

```
worker_processes 2;
pid /run/nginx.pid;
daemon off;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_names_hash_bucket_size 64;

    include /usr/local/nginx/conf/mime.types;
    default_type application/octet-stream;

    access_log /usr/local/nginx/logs/access.log;
    error_log  /usr/local/nginx/logs/error.log;

    gzip on;
    gzip_disable "msie6";
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    server {
        listen     8000;

        aws_access_key "YOUR_KEY_ID";
        aws_signing_key "FIRST_LINE_FROM_generate_signing_key";
        aws_key_scope "SECOND_LINE_FROM_generate_signing_key";
        aws_s3_bucket arch-eng-shared;
        
        location / {
            aws_sign;
            proxy_pass https://arch-eng-shared.s3.amazonaws.com;
        }
    }
}
```

Things you want to tweak include:


* aws_access_key
* aws_secret_key
* s3_bucket
* proxy_cache_valid - change 24h to your cache duration as desired.


