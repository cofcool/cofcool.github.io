---
layout: post
category : Tech
title : Apache笔记
tags : [apache, notes]
---
{% include JB/setup %}

目录:


<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [1. 重启时的提示未设置域名](#1-重启时的提示未设置域名)
* [2. wordpress使用固定链接设置时，提示没有权限](#2-wordpress使用固定链接设置时提示没有权限)
* [3. apache2更改端口](#3-apache2更改端口)
* [4. 自定义错误页面](#4-自定义错误页面)

<!-- /code_chunk_output -->


### 1. 重启时的提示未设置域名

**问题**：

* Restarting web server apache2                                                AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message

解决：
 打开终端，输入以下命令：

    sudo vim /etc/apache2/apache2.conf

默认情况下，这个是一个空文件，在文件中加入以下内容：

    ServerName localhost

保存文件退出，再次重启apache，错误提示没有了。

### 2. wordpress使用固定链接设置时，提示没有权限

**问题**：

 * wordpress使用*固定链接设置*时，提示没有权限

解决：

    sudo vim /etc/apache2/apache2.conf

    # 修改如下

    <Directory />
            Options FollowSymLinks
            AllowOverride ALL # 由None改为ALL
            Require all denied
    </Directory>

    <Directory /usr/share>
            AllowOverride None
            Require all granted
    </Directory>

    <Directory /var/www/>
            Options Indexes FollowSymLinks
            AllowOverride ALL  # 由None改为ALL
            Require all granted
    </Directory>

-------------

**问题**:

* 按照上述方法修改之后，未生效

解决：

    原因是: wordpress使用了mod_rewrite.c来映射虚拟路径，因此需要开启该模块，如下：

    sudo a2enmod rewrite

### 3. apache2更改端口

**问题**：

* 如何更改apache2的端口

    ```
    sudo vim /etc/apache2/ports.conf
    # 按照如下修改
    Listen 11111 # 根据需求修改

    <IfModule ssl_module>
            Listen 443
    </IfModule>

    <IfModule mod_gnutls.c>
            Listen 443
    </IfModule>
    ```

以上修改之后，重启服务器，ip+端口访问，会发现访问的是`/var/www/`，不会直接定向到`/var/www/html`。因为apache的默认站点设置我们还未更改，因此，还需要修改默认站点的端口号，具体操作如下：

    ```
    sudo vim /etc/apache2/sites-enabled/000-default.conf
    # 修改如下，把端口号改为之前设置的端口号
    <VirtualHost *:11111>
    ...
    ...
    ```

### 4. 自定义错误页面


```
vi /etc/httpd/conf/httpd.conf


#
# Putting this all together, we can internationalize error responses.
#
# We use Alias to redirect any /error/HTTP_<error>.html.var response to
# our collection of by-error message multi-language collections.  We use
# includes to substitute the appropriate text.
#
# You can modify the messages' appearance without changing any of the
# default HTTP_<error>.html.var files by adding the line:
#
#   Alias /error/include/ "/your/include/path/"
#
# which allows you to create your own set of files by starting with the
# /var/www/error/include/ files and
# copying them to /your/include/path/, even on a per-VirtualHost basis.
#

Alias /error/ "/var/www/error/"

...
#    ErrorDocument 400 /error/HTTP_BAD_REQUEST.html.var
#    ErrorDocument 401 /error/HTTP_UNAUTHORIZED.html.var
#    ErrorDocument 403 /error/HTTP_FORBIDDEN.html.var
    ErrorDocument 404 /error/HTTP_NOT_FOUND.html.var
#    ErrorDocument 405 /error/HTTP_METHOD_NOT_ALLOWED.html.var
#    ErrorDocument 408 /error/HTTP_REQUEST_TIME_OUT.html.var
#    ErrorDocument 410 /error/HTTP_GONE.html.var
#    ErrorDocument 411 /error/HTTP_LENGTH_REQUIRED.html.var
...

```

从以上可以看到，如果想要自定义错误页面，可以打开对应的错误页面，然后修改`/var/www/error/include/`目录下的页面文件。

最近想要重新使用`cofcool.net`域名，并把请求重定向到`cofcool.github.io`，但是之前被搜索引擎索引到的页面在我自己服务器上并不存在，于是想把这些请求重定向到`cofcool.github.io`。只要在`404`页面添加重定向代码即可实现：

```js
vi /var/www/error/include/top.html

<script type="text/javascript">
    var url ="https://cofcool.github.io"+window.location.pathname;
    // 使用substring, 是为了截掉尾部的"/"字符
    window.location.href=url.substring(0, url.length-1)
</script>
```


