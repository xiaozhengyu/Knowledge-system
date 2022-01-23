# Nginx 实战 - 反向代理

（环境：Win10 + Docker）

1.   查看 Nginx 全局配置文件 /etc/nginx/nginx.conf

     ```
     user  nginx;
     worker_processes  auto;
     
     error_log  /var/log/nginx/error.log notice;
     pid        /var/run/nginx.pid;
     
     
     events {
         worker_connections  1024;
     }
     
     
     http {
         include       /etc/nginx/mime.types;
         default_type  application/octet-stream;
     
         log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                           '$status $body_bytes_sent "$http_referer" '
                           '"$http_user_agent" "$http_x_forwarded_for"';
     
         access_log  /var/log/nginx/access.log  main;
     
         sendfile        on;
         #tcp_nopush     on;
     
         keepalive_timeout  65;
     
         #gzip  on;
     
         include /etc/nginx/conf.d/*.conf;
     }
     ```

     依照配置文件最后一句 “ include /etc/nginx/conf.d/*.conf; ”，到 /etc/nginx/conf.d 目录下创建自定义配置

2.   创建自定义配置

     ```
     server {
         listen       80;
         server_name  127.0.0.1;
         
         location ~ /aaa/ {
             proxy_pass http://192.168.0.106:8081;
         }
         
         location ~ /bbb/ {
             proxy_pass http://192.168.0.106:8082;
         }
     }
     ```

     -   127.0.0.1 指的是 Nginx 服务器自己
     -   127.0.0.1:80/aaa/xxx -> http://192.168.0.106:8081/aaa/xxx
     -   127.0.0.1:80/bbb/xxx ->http://192.168.0.106:8082/bbb/xxx

3.   重启 Nginx

4.   测试

     （1）不通过 Nginx 反向代理，直接访问目标地址

     ![image-20220123155922530](markdown/Nginx 实战 - 反向代理.assets/image-20220123155922530.png)

     ![image-20220123155930255](markdown/Nginx 实战 - 反向代理.assets/image-20220123155930255.png)

     （2）通过 Nginx 反向代理访问目标地址

     ![image-20220123160022718](markdown/Nginx 实战 - 反向代理.assets/image-20220123160022718.png)

     ![image-20220123160031275](markdown/Nginx 实战 - 反向代理.assets/image-20220123160031275.png)