
[github](https://github.com/laurent22/joplin)
[server Installing](https://github.com/laurent22/joplin/blob/dev/packages/server/README.md)

[Setting up a Joplin Server on Docker](https://jonasclaes.be/setting-up-a-joplin-server-on-docker/)

```sh
docker run -d --name joplin_server -v joplin:/home/joplin --env-file /home/app/joplin/.env -p 22300:22300 joplin/server:latest
```

Nginx 代理配置方式(不配置的话会提示错误: Invalid origi http://10.42.0.1):

```
server {
    server_name 10.42.0.1;
        # 注意路径 /joplin/ 和 /joplin 的区别
        location /joplin/ {
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Host localhost:22300;
                proxy_redirect off;
                proxy_pass http://localhost:22300;
                rewrite ^/joplin/(.*)$ /$1 break;
        }
    }
}
```