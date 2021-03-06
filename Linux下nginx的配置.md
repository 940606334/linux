##nginx常用命令及简单配置

nginx常用命令

    nginx -c /usr/local/nginx/conf/nginx.conf  启动nginx(windows下start nginx);  
    nginx -s quit       停止ngix  
    nginx -s reload     重新载入nginx(当配置信息发生修改时)  
    nginx -s reopen     打开日志文件  
    nginx -v            查看版本  
    nginx -t            查看nginx的配置文件的目录  
    nginx -h            查看帮助信息  
  
  
linux下搭建nginx环境 

    pwd 查看当前目录  
    cd /home/download  找到nginx安装包  
    tar -zxvf nginx-1.10.3.tar.gz   解压nginx安装包  
    cd nginx-1.10.3     进入nginx的目录  
    ./configure     运行nginx配置文件(如果出现错误,可能缺少库文件,安装后再执行这一步)  
        su  进入root权限,回车后输入密码  
        cd /    进入到根目录  
        yum -y install gcc gcc-c++ autoconf automake    安装gcc和gcc-c++(-y安装时选择同意选项,autoconf automake 自动配置自动安装,出现complete安装成功)  
        yum -y install pcre pcre-devel 安装pcre库  
        yum -y install zlib zlib-devel 安装zlip库  
    ./configure     进入到nginx目录再运行一次,直到成功后  
    make    编译  
    make install 安装nginx  
    cd /usr/local->ls    查看是否有nginx,如果有则安装完成  
    cd nginx    conf目录放着配置文件 htmL放着网页程序 logs放着日志文件 sbin放着nginx的启动程序  
    /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf      启动nginx  
    浏览器打开localhost查看  
          
windows下安装  
    下载解压安装包后直接双击运行nginx.exe配置文件(或者start nginx命令)  
    浏览器打开localhost查看  
  
  
linux将nginx配置到全局

    cd ~    进入用户根目录  
    ls -a   查看所有文件(包含隐藏)  
    vim .bashrc 进入环境变量配置文件  
    export NGINX=/usr/local/nginx/sbin/nginx  
    PATH=$PATH:$NGINX   修改环境变量  
    :qw 保存退出 (:q! 不保存退出ctrl d向下翻页ctrl u向上翻页)  
    source .bashrc 修改后的配置文件生效  
  
nginx配置文件修改

    nginx -t    查看nginx配置文件目录  
    cp nginx.conf nginx_bf.conf     将配置文件备份一下  
    vim /user/local/nginx/conf/nginx.conf 打开nginx配置文件  
  
    vim命令  
        :q! 不保存退出  
        :qw 保存退出  
        ctrl d向下翻页  
        ctrl u向上翻页  
    nginx -s reload     当配置信息发生修改时,重新载入nginx,才能生效  
  
nginx配置文件说明

      worker_processes  1;      //开启进程数小于CPU数   
      error_log  logs/error.log;  //自定义错误日志保存位置，全局设置，默认logs/error.log  
      events {  
          worker_connections  1024; //每个进程最大连接数（最大连接=连接数x进程数）每个worker允许同时产生多少个链接，默认1024  
      }  
        
      http {  
          include       mime.types;     //文件扩展名与文件类型映射表  
          default_type  application/octet-stream;    //默认文件类型  
          log_format  main  '$remote_addr - $remote_user [$time_local] "$request" ' //自定义日志文件输出格式 全局设置  
                    '$status $body_bytes_sent "$http_referer" '  
                    '"$http_user_agent" "$http_x_forwarded_for"';  
          access_log  logs/access.log  main;    自定义全局请求日志保存位置，全局设置，默认logs/access.log， 定义格式：文件存储位置 + 日志输出格式  
          sendfile        on;   //打开发送文件  
          keepalive_timeout  0;     //连接超时时间  
          keepalive_timeout  65;  
          gzip  on;     //打开gzip压缩  
          配置虚拟主机，基于域名、ip和端口，可以配置多个server  
          server {  
              listen       80;  //监听端口，可以是ip:port 或者 port  
              server_name  10.128.166.57; //监听域名,可以是ip或者域名，server_name有三种匹配方式：精准匹配（www.domain.com）、通配符匹配（*.domain.com 、www.*）、正则表达式匹配（~^(?.+)\.domain\.com$）  
              access_log  logs/host.access.log  main;   //自定义请求日志，局部，当前server有效  
              error_page   500 502 503 504  /50x.html;  //错误页面及其返回地址  
              charset UTF-8;    //设置字符集  
              location / {  //当访问10.128.166.57:80时  
                 proxy_pass http://10.128.166.57:80:8083; //实际上访问的时http://10.128.166.57:80:8083地址  
              }  
              location ^~/data {    //当访问10.128.166.57:80/data时  
                 proxy_pass http://10.128.166.57:80:8084; //实际上访问的时http://10.128.166.57:80:8084地址  
              }  
                
          }  
            
      }  
  
  
nginx日志分割备份

    mkdir /usr/local/nginx/back_up_logs //创建存放备份文件目录  
    vim /usr/local/nginx/sbin/log.sh //创建脚本log.sh  
    chmod 755 log.sh //脚本授权  
    crontab -e //执行该命令设置定时任务  
        */1 * * * * sh /usr/local/nginx/sbin/log.sh //每分钟执行一次，保存退出即可自动开始执行定时任务  
    crontab -l //查看所有定时任务  
    crontab -r //删除所有定时任务  
    log.sh文件的内容:  
         #!/bin/sh  
        #设置基路径  
        BASE_DIR=/usr/local/nginx  
        #要切割备份的日志文件名  
        BASE_FILE_NAME=access.log  
        #日志路径  
        LOG_PATH=$BASE_DIR/logs  
        #日志切割后备份路径  
        BAK_PATH=$BASE_DIR/back_up_logs  
        #切割日志文件  
        LOG_FILE=$LOG_PATH/$BASE_FILE_NAME  
        #获取时间  
        BAK_TIME=`/bin/date -d yesterday +%Y%m%d%H%M`  //以分钟为单位  
        #备份文件  
        BAK_FILE=$BAK_PATH/$BAK_TIME-$BASE_FILE_NAME  
        echo $BAK_FILE  
        #关闭nginx  
        $BASE_DIR/sbin/nginx -s stop  
        #移动切割文件  
        mv $LOG_FILE $BAK_FILE  
        #启动nginx  
        $BASE_DIR/sbin/ngin  
  
  
  
解决端跨域问题(保证ip和端口相同)修改配置文件\nginx-1.10.3\conf\nginx.conf文件  

    server {  
        listen       8081;//前端调试打开localhost:8081页面;js文件中后台接口访问localhost:8081/data;这样就保证不跨域了  
        server_name  localhost;  
        access_log  logs/host.access.log  main;  
        location / {    //访问localhost:8081实际上访问是前端端口http://localhost:8080/  
            proxy_pass http://localhost:8080/;  
        }  
        location ^~ /data {//访问localhost:8081/data实际上访问是后端接口http://10.128.166.42:8533/  
            proxy_pass http://10.128.166.42:8533/;  
        }


从nginx官网下载你所需要版本的压缩包
	
	wget http://nginx.org/download/nginx-1.10.3.tar.gz

解压后进入目录,并进行配置编译和安装

	tar zxvf nginx-1.10.3.tar.gz
	cd nginx-1.10.3
	./configure
	make &&make install
这样就安装在了默认路径/usr/local/nginx下,我们需要配置文件 conf/vhost/中的*.conf文件.