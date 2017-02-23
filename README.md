# Overview
Docker image for php, node, composer

```bash
docker build -t php-node-docker .
docker run -it --rm php-node-docker /bin/bash
```

# Usage

Where `app.conf` may be:

```apacheconf
server {
    server_name *.foo.com;

    root /var/www/app/public;
    index index.php index.html index.htm;

    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_pass unix:/run/php/php7.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

and `supervisor.conf` may be:

```bash
$ cat conf/supervisord.conf
[supervisord]
nodaemon=true

[program:php-fpm]
command=/usr/sbin/php-fpm7.1 -F
autorestart=true

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
autorestart=true
```

Your `Dockerfile` might be:

```bash
FROM sourceallies/php-node-docker
COPY src /var/www/app
RUN useradd -u 1001 -G www-data -m sai
RUN chown -R sai:www-data /var/www/app
RUN mkdir -p /var/log/supervisor /var/run/php
WORKDIR /var/www/app
ADD app.conf /etc/nginx/conf.d/app.conf
COPY conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
EXPOSE 80
ENTRYPOINT ["/usr/bin/supervisord"]
```
