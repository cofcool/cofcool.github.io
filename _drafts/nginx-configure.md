# Nginx 配置

环境：

* Ubuntu Server 16.04
* Nginx 1.10.3

### 1. 安装

### 2. 虚拟主机配置

```sh
sudo vim /etc/nginx/sites-enabled/default
sudo service nginx restart
```
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html index.php;

    server_name _;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    # deny access to .htaccess files, if Apache's document root
    #  concurs with nginx's one

    location ~ /\.ht {
        deny all;
    }
}
server {
    listen 80;

    server_name example.com;

    location /ZZDManagerWeb {
        try_files $uri $uri/ =404;
    }

    root /var/www/html/ZZDManagerWeb;
}
```

php:
```
sudo apt install php-cgi
sudo apt install php-fpm

sudo vim /etc/php/7.0/cgi/php.ini
cgi.fix_pathinfo=1
```
```
sudo vim /etc/nginx/sites-enabled/default
server {
    ...
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # With php7.0-cgi alone:
        # fastcgi_pass 127.0.0.1:9000;
        # With php7.0-fpm:
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    }
}
```
Nginx映射目录的所属用户需和Nginx的启动用户一致。

### 3. 列出目录

```sh
sudo vim /etc/nginx/nginx.conf

http {
    autoindex on;
    autoindex_exact_size off;
}

```

### 4. 端口转发

[Module ngx_http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
[Nginx: Everything about proxy_pass](https://dev.to/danielkun/nginx-everything-about-proxypass-2ona)

```
server {
    listen 80;

    server_name git.cofcool.net;

    location / {
        proxy_pass http://127.0.0.1:8181;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 5. gzip

http://nginx.org/en/docs/http/ngx_http_gzip_module.html

```
gzip            on;
gzip_min_length 1000;
gzip_proxied    expired no-cache no-store private auth;
gzip_types      text/plain application/xml;
gzip_comp_level 2;
```