
The image assumes a config file in the container at: `/nginx.conf` so use the `-v` option to
mount one from your host.


```
docker run -p 8152:8000 -v /absolute/path/to/nginx.conf:/nginx.conf iotile/nginx-s3-proxy
```

If you want to easily view the logs on the host, bind a path to `/usr/local/nginx/logs/`:

```
docker run -p 8152:8000 -v /absolute/path/to/nginx.conf:/nginx.conf -v /my/path:/usr/local/nginx/logs/ iotile/nginx-s3-proxy
```

(The port `8152` is the port that the Yocto config in iotile/meta-arch-systems/local.conf.sample expects)


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

        aws_access_key "AWS_ACCESS_KEY_ID";
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
* aws_signing_key
* aws_key_scope

The signing key and scope must be generated. This container uses the plugin from the anomalizer/ngx_aws_auth repo and it includes a script to generate both. See https://github.com/anomalizer/ngx_aws_auth for more details.



