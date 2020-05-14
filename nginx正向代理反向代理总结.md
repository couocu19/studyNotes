# nginx正向代理/反向代理总结

## 配置文件解读

### nginx.conf

**位置：/usr/local/nginx/conf/nginx.conf**

```

#user  nobody;
worker_processes  1;
(工作进程：数目。根据硬件调整，通常等于CPU数量或者2倍的CPU)

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
(错误日志存放位置)

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
    (每个工作进程的最大连接数量。需要和前面工作进程配合起来用。尽量要大，但是不要把cpu跑到100%。
    理论上每台nginx服务器的最大连接数为：worker_processes * worker_connections )
}


##指定http服务器，利用他的反向代理功能提供负载均衡支持
http {
    include       mime.types;
    （设定mine类型，类型由mine.type文件定义）
    
    default_type  application/octet-stream;（指的是八进制文件）
        charset utf-8,gbk;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    （日志的格式设置,分别为：客户端的ip地址，客户端的用户名称，访问时间与时期，请求的url与http协议，请求状态，发送给客户端文件主体内容大小，从哪个页面链接访问过来的，客户端浏览器的相关信息，原有客户端的ip地址和原来客户端的请求的服务器地址）

    #access_log  logs/access.log  main;
    (用了log_format指令设置了日志格式之后，需要用access_log指令指定日志文件的存放路径)

    sendfile        on;
    （该指令指定nginx是否调用sendfile函数（zero copy方式）来输出文件，对于普通应用，必须设为on。如果用来下载等应用用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度，降低系统uptime）
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;
    （keepalive超时时间）

    #gzip  on;
    
    （配置虚拟机）
    server {
       （配置监听端口）
        listen       80;
        (配置访问域名)
        server_name  localhost;
        
        charset utf-8;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

    include vhost/*.conf;

    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```



- ### 项目配置

  ```
  server
  {
          listen 80;
          autoindex on;
          server_name 118.31.12.175;  
                  charset utf-8;
          access_log /usr/local/nginx/logs/acess.log.combined;
          index index.html index.htm index.jsp index.php;
          if ( $query_string ~* ".*[\;'\<\>].*" ){
                  return 404;
          }            
             #root  /home/coucou/main1;
             root /usr/local/nginx/html ;
  
       location / {
               try_files $uri $uri/ /index.html;         
          }
  
  
       location /xiyou{
              try_files $uri $uri/ /main1;
          
  
         }
  
      location /coucou
      {
          proxy_pass http://118.31.12.175:8080/; 
      }
  }
  ```

## nginx实现静态代理的配置

（以下的配置代码是我目前找到的感觉最靠谱的配置文件啦）

```

server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  ***.***.***;#域名替换成自己的
    add_header Access-Control-Allow-Origin '*';
    add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,PATCH,OPTIONS;
    set $root /xxx/nginx/; # 变量设置，设置公共路径
 
#项目一，路径替换成自己的
    location / {
        root $root;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
#项目二，路径替换成自己的
    location /dataplatform {
        alias $root/项目A;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
    
#后端项目用的是flask
        location /tagging {
            include uwsgi_params;
            uwsgi_pass 127.0.0.1:5001;
        }

}

```

#### 说明：关于以上静态代理的配置，大概的思路就是：首先在server里面配置公共的根目录的路径$root，然后你可以在根目录下放置不同的前端项目（多个），目录的结构大概为：

![1589462785075](C:\Users\11310\AppData\Roaming\Typora\typora-user-images\1589462785075.png)

然后需要注意的就是多项目配置的时候location中需要用 alias表示项目路径。

#### alias和root的区别：

```
1）alias指定的目录是准确的，location替换为aligs路径，即location匹配到的path路径下的文件在aligs目录下是可以找到的；
2）root指定的目录是location匹配访问path路径的上一级目录，这个path目录要真实存在在root路径下；
3）使用alias的location代码块中，不能存在rewrite的break；alias路径最后必须带上“/”；
4）alias的location匹配的path路径中，如果最后不加“/”，访问的时候会自动加上“/”；如果path后面加上“/”，访问的时候路径最后必须加上“/”，否则会保存
5）root的location匹配的path路径中，加不加“/”都可以
```

```
【root】
语法      root path;
默认值    root html; #代表nginx目录下html文件夹
作用域    http、server、location、if
 
【alias】
语法      alias path; #path结尾一定要加以“/”结束
作用域    location
```

 **建议：1.location / 使用root；2.location /path 使用alias；** 

