readme.mdNginx在运行时候，至少要加载几个核心模块和一个事件类模块。这些模块运行时所支持的配置项称为基本配置——所有其他模块执行时都依赖的配置项。

**由于配置项较多，所以把它们按照用户使用时的预期功能分成以下4类：**

1. 用于调试、定位问题的配置项；
2. 正常运行的必备配置项；
3. 优化性能的配置项；
4. 事件类配置项（有些事件类配置项归纳到优化性能类，这是因为它们虽然也属于event{}块，但作用是优化性能）

有一些配置项，其实没有显式的进行配置，他们会有默认的值，如：daemon，即是在nginx.conf中没有对它进行配置，也相当于打开了这个功能，这点需要注意。

***官网对各个模块参数配置的解释说明网址：*** [Nginx中文文档](http://www.nginx.cn/doc/index.html)

#### 1.代码块

代码块中的events、http、server、location、upstream等都是块配置项，块配置项可以嵌套。内层块直接继承外层快，例如：server块里的任意配置都是基于http块里的已有配置的

#### 2.用户及用户组

Nginx worker进程运行的用户及用户组 
语法：user  username [groupname] 
默认：user nobody nobody
user用于设置master进程启动后，fork出的worker进程运行在那个用户和用户组下。
当按照"user username;"设置时，用户组名与用户名相同。
若用户在configure命令执行时，使用了参数  --user=usergroup  和 --group=groupname,此时nginx.conf 将使用参数中指定的用户和用户组。

#### 3.进程个数

Nginx worker进程个数：其数量直接影响性能。
每个worker进程都是单线程的进程，他们会调用各个模块以实现多种多样的功能。如果这些模块不会出现阻塞式的调用，那么，有多少CPU内核就应该配置多少个进程，反之，有可能出现阻塞式调用，那么，需要配置稍多一些的worker进程。

```nginx
worker_processes  1;
```

#### 4.ssl硬件加速

用户可以用OpneSSL提供的命令来查看是否有ssl硬件加速设备：openssl engine -t

```nginx
ssl_engine device;
```

#### 5.守护进程

守护进程(daemon)。是脱离终端在后台允许的进程。它脱离终端是为了避免进程执行过程中的信息在任何终端上显示。这样一来，进程也不会被任何终端所产生的信息所打断。
关闭守护进程的模式，之所以提供这种模式，是为了放便跟踪调试nginx，毕竟用gdb调试进程时最繁琐的就是如何继续跟进fork出的子进程了。
如果用off关闭了master_proccess方式，就不会fork出worker子进程来处理请求，而是用master进程自身来处理请求

```nginx
daemon off;          #查看是否以守护进程的方式运行Nginx 默认是on 
master_process off;  #是否以master/worker方式工作 默认是on
```

#### 6.日志

error日志的设置
语法： `error_log /path/file level;`
默认： `error_log / log/error.log error;`
当path/file 的值为 /dev/null时，这样就不会输出任何日志了，这也是关闭error日志的唯一手段；leve的取值范围是debug、info、notice、warn、error、crit、alert、emerg从左至右级别依次增大。当level的级别为error时，error、crit、alert、emerg级别的日志就都会输出。大于等于该级别会输出，小于该级别的不会输出。
如果设定的日志级别是debug，则会输出所有的日志，这一数据量会很大，需要预先确保/path/file所在的磁盘有足够的磁盘空间。级别设定到debug，必须在configure时加入 --with-debug配置项。

```nginx
error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;
```

#### 7.主进程的ID

pid文件（master进程ID的pid文件存放路径）的路径

```nginx
pid         logs/nginx.pid;
```

#### 8.events模块

```nginx
#全局配置

events {

      accept_mutex on; #设置网路连接序列化，防止惊群现象发生，默认为on

      multi_accept on; #设置一个进程是否同时接受多个网络连接，默认为off

      #use epoll; #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport

      worker_connections 1024; # 工作进程的最大连接数量

      client_header_buffer_size 4k;

      open_file_cache max=2000 inactive=60s;

      open_file_cache_valid 60s;

      open_file_cache_min_uses 1

}
```

* accept_mutex on;惊群现象：一个网路连接到来，多个睡眠的进程被同事叫醒，但只有一个进程能获得链接，这样会影响系统性能。设置网路连接序列化，防止惊群现象发生，默认为on
* multi_accept on;设置是否允许同时接受多个网络连接：只能在events模块设置，Nginx服务器的每个工作进程可以同时接受多个新的网络连接，但是需要在配置文件中配置，此指令默认为关闭，即默认为一个工作进程只能一次接受一个新的网络连接，打开后几个同时接受多个。
* use epoll;使用epoll的I/O 模型(值得注意的是如果你不知道Nginx该使用哪种轮询方法的话，它会选择一个最适合你操作系统的)
* **worker_connections 2000;**工作进程的最大连接数量 理论上每台nginx服务器的最大连接数为worker_processes*worker_connections worker_processes为我们再main中开启的进程数
* keepalive_timeout 60;keepalive超时时间。 这里指的是http层面的keep-alive 并非tcp的keepalive 。
* client_header_buffer_size 4k;客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为系统分页大小。查看系统分页可以使用 getconf PAGESIZE命令

#### 9.http模块

```nginx

http {

  #嵌入其他配置文件 语法：include /path/file
  #参数既可以是绝对路径也可以是相对路径（相对于Nginx的配置目录，即nginx.conf所在的目录）
  include mime.types;
  default_type application/octet-stream;

  #server_tokens off; 隐藏版本号
  server_tokens off;
  
  #设置日志格式
  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"';

  #指定日志文件的存放路径、格式和缓存大小
  access_log /opt/crdb_proxy/logs/access.log main;

  #sendfile: 设置为on表示启动高效传输文件的模式。
  sendfile on;
  #tcp_nopush     on;

  #设置超时时间 65秒
  keepalive_timeout 65;

  #开启gzip压缩，提高传输效率，节约宽带
  gzip on;

  #openresty利用lua_package_path来引入moudule模块
  lua_package_path "/opt/crdb_proxy/lua/?.lua;";

  #负载均衡(默认方式轮询，即每个请求按照时间顺序轮流分配到不同的后端服务器)
  upstream apiLogServer {
    server 127.0.0.1:8088;
    server 127.0.0.1:8089;
  }

  # 虚拟机主机块定义
  server {

    # 监听端口为80，可以自定义其他端口，也可以加上IP地址，如，listen 127.0.0.1:8080;
    listen 80;

    # 定义网站域名，可以写多个，用空格分隔。
    server_name crdb_proxy;

    # 定义网站根目录，目录可以是相对路径也可以是绝对路径
    #root        /opt/crdb_proxy/pages;

    # 定义站点的默认页。
    index index.html;
    #charset koi8-r;

    # 定义访问日志，可以针对每一个server（即每一个站点）设置它们自己的访问日志。
    #access_log  logs/host.access.log  main;

    #location / {
    #    root   html;
    #    index  index.html index.htm;
    #}
    
    #
    #location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|js|pdf|txt) {
    #    root /opt/edr_proxy/pages;
    #}
    
    # 匹配以 /search开始的请求
    location /search {
      set $searchSvUrl 'http://127.0.0.1:8080/aaa/bb/c';
      add_header 'Content-Type' 'application/json';
      content_by_lua_block {
        local cln = require "search_api"
        cln.doSearch()
      }
    }

     # 匹配以 /crdb 开始的请求
    location /crdb {
      # 设置请求中的指定字段值
      set $nodeApiHost '127.0.0.1';
      set $nodeApiPort 8020;
      set $groupSearchUrl 'http://127.0.0.1:8080/a/b/c';
      set $nodeApiNewsUrl 'http://127.0.0.1:8080';
      set $logUrl 'http://127.0.0.1:80/aaa/bb';
      #set $debug false;
      
      # Rewirte配置 break 标记在本条规则匹配完成后，终止匹配，不再匹配后面的规则。
      rewrite ^/crdb/(.*) /$1 break;
      content_by_lua_block {
        local cln = require "api_proxy"
        ngx.header["Content-type"] = "application/json;charset=UTF8;"
        ngx.say(cln.request_node_api());
      }
    }

     # 匹配以 /apiLog 开始的请求 并且将其转发给 http://ServerA;
    location /apiLog {
      proxy_pass http://ServerA;
      proxy_connect_timeout 30;
    }
    
     # 匹配以 /logApi 开始的请求 并且将其转发给 http://ServerB; 标记在本条规则匹配完成后，终止匹配，不再匹配后面的规则。
    location /logApi {
      rewrite ^/logApi/(.*) /$1 break;
      proxy_pass http://ServerB;
      proxy_connect_timeout 30;
    }

    # 定义404页面
    # error_page  404              /404.html;
    # 当状态码为500、502、503、504时，则访问50x.html
    # error_page   500 502 503 504  /50x.html;
    # location = /50x.html {
    #    root   html;  #定义50x.html所在路径
    # }

  }
}
```




> **location匹配命令**
>
> ~    #波浪线表示执行一个正则匹配，区分大小写
> ~*  #表示执行一个正则匹配，不区分大小写
> ^~  #^~表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
> =    #进行普通字符精确匹配
> @   #"@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
>
> 
>
> location 优先级官方文档
>
> 1. =前缀的指令严格匹配这个查询。如果找到，停止搜索。
> 2. 所有剩下的常规字符串，最长的匹配。如果这个匹配使用^〜前缀，搜索停止。
> 3. 正则表达式，在配置文件中定义的顺序。
> 4. 如果第3条规则产生匹配的话，结果被使用。否则，使用第2条规则的结果。
>
>
> ```nginx
> location  = / {
> # 只匹配"/".
> [ configuration A ] 
> }
> location  / {
> # 匹配任何请求，因为所有请求都是以"/"开始
> # 但是更长字符匹配或者正则表达式匹配会优先匹配
> [ configuration B ] 
> }
> location ^~ /images/ {
> # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
> [ configuration C ] 
> }
> location ~* .(gif|jpg|jpeg)$ {
> # 匹配以 gif, jpg, or jpeg结尾的请求. 
> # 但是所有 /images/ 目录的请求将由 [Configuration C]处理.   
> [ configuration D ] 
> }
> ```
>
> 请求URI例子:
>
> - / -> 符合configuration A
> - /documents/document.html -> 符合configuration B
> - /images/1.gif -> 符合configuration C
> - /documents/1.jpg ->符合 configuration D
>
> ```
> location = / {
>    #规则A
> }
> location = /login {
>    #规则B
> }
> location ^~ /static/ {
>    #规则C
> }
> location ~ \.(gif|jpg|png|js|css)$ {
>    #规则D，注意：是根据括号内的大小写进行匹配。括号内全是小写，只匹配小写
> }
> location ~* \.png$ {
>    #规则E
> }
> location !~ \.xhtml$ {
>    #规则F
> }
> location !~* \.xhtml$ {
>    #规则G
> }
> location / {
>    #规则H
> }
> ```
>
> 那么产生的效果如下：
>
> 访问根目录/， 比如http://localhost/ 将匹配规则A
>
> 访问 http://localhost/login 将匹配规则B，http://localhost/register 则匹配规则H
>
> 访问 http://localhost/static/a.html 将匹配规则C
>
> 访问 http://localhost/a.gif, http://localhost/b.jpg 将匹配规则D和规则E，但是规则D顺序优先，规则E不起作用， 而 http://localhost/static/c.png 则优先匹配到 规则C
>
> 访问 http://localhost/a.PNG 则匹配规则E， 而不会匹配规则D，因为规则E不区分大小写。
>
> 访问 http://localhost/a.xhtml 不会匹配规则F和规则G，
>
> http://localhost/a.XHTML不会匹配规则G，（因为!）。规则F，规则G属于排除法，符合匹配规则也不会匹配到，所以想想看实际应用中哪里会用到。
>
> 访问 http://localhost/category/id/1111 则最终匹配到规则H，因为以上规则都不匹配，这个时候nginx转发请求给后端应用服务器，比如FastCGI（php），tomcat（jsp），nginx作为方向代理服务器存在。


> ##### sendfile        on;
>
> sendfile: 设置为on表示启动高效传输文件的模式。sendfile可以让Nginx在传输文件时直接在磁盘和tcp socket之间传输数据。如果这个参数不开启，会先在用户空间（Nginx进程空间）申请一个buffer，用read函数把数据从磁盘读到cache，再从cache读取到用户空间的buffer，再用write函数把数据从用户空间的buffer写入到内核的buffer，最后到tcp socket。开启这个参数后可以让数据不用经过用户buffer。




> 一、什么是MIME-TYPE
> MIME——Multipurpose Internet Mail Extension（多用途因特网邮件扩展）最初是为了满足电子邮件支持多字符集及附件而出现的。
> 通过MIME，我们可以写一封既含有英文，又含有中文，再加上一个文件作为附件的邮件。这种含有多种类型数据的文件被称为多部分对象集合（Multipart messages）。
> MIME Type 不是个人指定的，是经过 ietf 组织协商，以 RFC 的形式作为建议的标准发布在网上的，大多数的 Web 服务器和用户代理都会支持这个规范
>
> 二、MIME与HTTP协议
> 除了支持电子邮件的SMTP协议以外，MIME还被其他协议或者程序广泛使用着，这其中就包括大名鼎鼎的HTTP协议。HTTP服务器在发送一份报文主体时，在HTTP报文头部插入解释自身数据类型的MIME头部信息（Content-Type）。
> MIME-type和Content-Type的关系：
> 当web服务器收到静态的资源文件请求时，依据请求文件的后缀名在服务器的MIME配置文件中找到对应的MIME Type，再根据MIME Type设置HTTP Response的Content-Type，然后客户端如浏览器根据Content-Type的值处理文件。
>
> 三、MIME与Nginx
> nginx.conf配置文件http指令块有如下默认指令
>
> ```
> http {
>  include       mime.types;
>  default_type  application/octet-stream;
> 
> ```
>
>
> include表示纳入mime.types文件的配置，
> 如果nginx.conf有重复的指令，可以抽取出来，利用include来帮我们简化配置，避免修改一个相同的配置，要改动好几个地方。
>
> ```
> incluee   xxx.conf;
> ```
>
> 
>
> ##### default_type
>
> 如果Web程序没设置，Nginx也没找到对应文件的扩展名的话，就使用默认的Type，这个在Nginx 里用 default_type定义: default_type application/octet-stream，这是应用程序文件类型的默认值。
>
> ##### application/octet-stream
>
> 是HTTP规范中Content-Type的一种，意思是 未知的应用程序文件 ，浏览器一般不会自动执行或询问执行。
> 只能提交一个二进制，如果提交文件的话，只能提交一个文件,后台接收参数只能有一个，而且只能是流（或者字节数组）。
>
> conf/mime.type文件内容如下：
>  text/html                                        html htm shtml;
>    text/css                                         css;
>  text/xml                                         xml;
>    image/gif                                        gif;
>      image/jpeg                                       jpeg jpg;
>      application/javascript                           js;
>      application/atom+xml                             atom;
>      application/rss+xml                              rss;
>      text/mathml                                      mml;
>      text/plain                                       txt;
>      text/vnd.sun.j2me.app-descriptor                 jad;
>      text/vnd.wap.wml                                 wml;
>      text/x-component                                 htc;
>      image/png                                        png;
>      image/svg+xml                                    svg svgz;
>      image/tiff                                       tif tiff;
>      image/vnd.wap.wbmp                               wbmp;
>      image/webp                                       webp;
>      image/x-icon                                     ico;
>      image/x-jng                                      jng;
>      image/x-ms-bmp                                   bmp;
>      application/font-woff                            woff;
>      application/java-archive                         jar war ear;
>      application/json                                 json;
>      application/mac-binhex40                         hqx;
>    
>    这里形如text/html格式的字符串就是用来说明数据类型的，/前的是主类型，/之后的是该主类型下的子类型。详细的类型定义在RFC2046中。
>    Nginx通过服务器端文件的后缀名来判断这个文件属于什么类型，再将该数据类型写入HTTP头部的Content-Type字段中，发送给客户端。
>    
>比如，当我们打开OSC的一个页面，看到一个PNG格式的图片的时候，Nginx是这样发送格式信息的：
>  
> 服务器上有enter_narrow.png这个文件，后缀名是png；
>根据mime.types，这个文件的数据类型应该是image/png；
> 将Content-Type的值设置为image/png，然后发送给客户端。
>![在这里插入图片描述](https://img-blog.csdnimg.cn/20181109141426596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Zlcml0YXNfQw==,size_16,color_FFFFFF,t_70)





