---
layout: post
title: 优化配置 Nginx 和 PHP-FPM
category: 编程学习
tags: [Nginx, PHP]
date: 2015-09-10
description:
keywords: nginx,php-fpm,optimize
---

## nginx

* `server_tokens off;`

    hide nginx server tokens

* `access_log off;`

    reduce nginx write to disk times

* `worker_processes 1;`

    always equals `grep processor /proc/cpuinfo | wc -l`

* `worker_connections 4096;`

    `ulimit -n` will echo the max connection of server.

* `client_body_buffer_size 32K;`

    This handles the client buffer size, meaning any POST actions sent to Nginx.

* `client_header_buffer_size 10k;`
* `large_client_header_buffers 4 32k;`
* `client_max_body_size 50m;`

* Gzip Compression

    Gzip can help reduce the amount of network transfer Nginx deals with. However, be careful increasing the gzip_comp_level too high as the server will begin wasting cpu cycles.

        gzip             on;
        gzip_comp_level  2;
        gzip_min_length  1000;
        gzip_proxied     expired no-cache no-store private auth;
        gzip_types       text/plain application/x-javascript text/xml text/css application/xml;

* Timeouts

    Timeouts can also drastically improve performance.

    The `client_body_timeout` and `client_header_timeout` directives are responsible for the time a server will wait for a client body or client header to be sent after request. If neither a body or header is sent, the server will issue a 408 error or Request time out.

    The keepalive_timeout assigns the timeout for keep-alive connections with the client. Simply put, Nginx will close connections with the client after this period of time.

    Finally, the send_timeout is established not on the entire transfer of answer, but only between two operations of reading; if after this time client will take nothing, then Nginx is shutting down the connection.

        client_body_timeout 12;
        client_header_timeout 12;
        keepalive_timeout 15;
        send_timeout 10;

## php-fpm

* `pm.max_spare_servers = 30;`

    The desired maximum number of idle server processes.

* `pm.max_children = 1024;`

    On an average each PHP-FPM process took ~75MB of RAM on my machine.

    Appropriate value for pm.max_children can be calculated as:

    pm.max_children = Total RAM dedicated to the web server / Max child process size - in my case it was 85MB

    The server has 8GB of RAM, so:

    pm.max_children = 6144MB / 85MB = 72

    assuming you are devoting 80% of available memory to PHP-FPM, the command below will suggest an approx pm.max_children value for all pools:

    `echo "pm.max_children = $(( $(awk '/MemTotal:/ { printf "%d\n", ($2*0.80) }' /proc/meminfo) / $(ps --no-headers -o "rss,cmd" -C php-fpm | awk '{ sum+=$1 } END { printf ("%d\n", sum/NR) }') ))"`

* `pm.max_requests = 800;`

    The number of requests each child process should execute before respawning.

Refers:

[http://myshell.co.uk/blog/2012/07/adjusting-child-processes-for-php-fpm-nginx/](http://myshell.co.uk/blog/2012/07/adjusting-child-processes-for-php-fpm-nginx/)

[https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration](https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration)

[http://www.hjue.me/post/php-fpm-nginx](http://www.hjue.me/post/php-fpm-nginx)

