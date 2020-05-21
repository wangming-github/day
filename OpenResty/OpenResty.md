

**OpenResty**通过 Lua 扩展 NGINX 实现的可伸缩的 Web 平台



#### 1.安装

1. 下载包。

   ```shell
   $ wget https://openresty.org/download/openresty-1.15.8.3.tar.gz
   $ tar -zxvf openresty-1.15.8.3
   ```

2. 使用Homebrew安装PCRE

   ```bash
   $ brew update
   $ brew install pcre openssl
   ```

3. pcre / openssl 都安装完成后，进入到openresty安装包的存放路径。可以使用下面的命令进行安装：

   ```shell
$ cd openresty-1.15.8.3
   $  ./configure \
      --with-cc-opt="-I/usr/local/opt/openssl/include/  \
      -I/usr/local/opt/pcre/include/" \
      --with-ld-opt="-L/usr/local/opt/openssl/lib/   \
      -L/usr/local/opt/pcre/lib/" \
      -j8
   ```
   
4. 编译并安装，默认会被安装到**/usr/local/openresty/**中

   ```shell
   $ sudo make
   $ sudo make install
   ```
   
5. 启动Nginx  

   ```shell
   $ sudo /usr/local/openresty/nginx/sbin/nginx
   $ ps -ef | grep nginx
   $ service nginx status
   ```
   
6. 到此 访问：http://127.0.0.1/
   Welcome to OpenResty!

   安装成功！





#### nginx命令

nginx -s reopen #重启Nginx

nginx -s reload #重新加载Nginx配置文件，然后以优雅的方式重启Nginx

nginx -s stop #强制停止Nginx服务

nginx -s quit #优雅地停止Nginx服务（即处理完所有请求后再停止服务）

nginx -t #检测配置文件是否有语法错误，然后退出

nginx -?,-h #打开帮助信息

nginx -v #显示版本信息并退出

nginx -V #显示版本和配置选项信息，然后退出

nginx -t #检测配置文件是否有语法错误，然后退出

nginx -T #检测配置文件是否有语法错误，转储并退出

nginx -q #在检测配置文件期间屏蔽非错误信息

nginx -p prefix #设置前缀路径(默认是:/usr/share/nginx/)

nginx -c filename #设置配置文件(默认是:/etc/nginx/nginx.conf)

nginx -g directives #设置配置文件外的全局指令

killall nginx #杀死所有nginx进程
  

#### 异常

   Nginx启动若出现

   ```shell
   nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
   nginx: [emerg] still could not bind()
   ```
   说明80端口并占用，查看80端口被占用的端口并重启。原因在于nginx先监听了ipv4的80端口之后又监听了ipv6的80端口，于是就重复占用了。

   ```shell
   $ sudo netstat -ntlp | grep 80
   $ sudo killall -9 nginx
   ```

重新编辑Nginx配置文件

```shell
$ sudo vim /etc/nginx/conf/nginx.conf

listen 80;
listen [::]:80 ipv6only=on default_server;
```

浏览器输入 http://127.0.0.1/ OK!



#### 将Nginx工具配置到当前用户的系统环境变量中

```shell
$ sudo vim ~/.bashrc
  export PATH=$PATH:/usr/local/openresty/nginx/sbin
$ source ~./bashrc
$ cd ~
$ sudo nginx -s reload
nginx: [alert] kill(12267, 1) failed (1: Operation not permitted)
```

