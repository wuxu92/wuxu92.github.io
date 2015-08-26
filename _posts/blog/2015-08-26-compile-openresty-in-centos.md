---
layout: post
title: 编译安装openresty
description:  openresty是章亦春（agentzh）维护的项目，早期由淘宝网赞助，后来作者加入cloudflare公司后，也就由该公司支持了。这是一个扩展nginx的项目，在nginx的基础上添加很多作者开发的模块。该项目目前有很多公司在使用。
category: post
tags: nginx openresty linux web-server
published: true
lastUpdate: 2015-08-24
---
{{page.description}}

openresty项目主页 [http://openresty.org/cn/](http://openresty.org/cn/ "http://openresty.org/cn/")， [GITHUB](http://github.com/agentzh/openresty.org "http://github.com/agentzh/openresty.org")。

我们要安装一个openresty作为webserver同时把它配置成与nginx使用方式相同。

## 编译安装 ##
这一部分比较简单，按照主页上的说明走就可以了。

```
cd tars  // 进入一个源码下载目录，比如一个临时目录
wget https://openresty.org/download/ngx_openresty-1.9.3.1.tar.gz
tar xzvf ngx_openresty-1.9.3.1.tar.gz
cd ngx_openresty-1.9.3.1

// configure,这里可能需要再安装一些包
//  perl 5.6.1+, libreadline, libpcre, libssl
./configure --prefix=/data/nginx
make
make install
```
这样一个openresty/nginx安装在目录: /data/nginx
配置文件： /data/nginx/nginx/conf/nginx.conf
可运行文件: /data/nginx/nginx/sbin/nginx

## 配置 ##
起始openresty就是一台nginx服务器，它的配置文件和nginx的配置文件是一样的配置的。在这里不细说，只是openresty安装完后并不能使用service/systemctl控制nginx的开启与关闭。

可以使用nginx的service文件。 从这里下载一个nginx的initscript（把内容拷贝出来新建文件）。把脚本保存为nginx。

```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/sbin/nginx"  # 修改为 /data/nginx/nginx/sbin/nginx
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/etc/nginx/nginx.conf" #修改为 /data/nginx/nginx/conf/nginx.conf
 
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
 
lockfile=/var/lock/subsys/nginx
 
make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -z "`grep $user /etc/passwd`" ]; then
       useradd -M -s /bin/nologin $user
   fi
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs  #建议注释掉这一行
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```
记得修改nginx运行程序目录和配置文件目录两项，可恶意吧start部分的make_dirs注释掉。。。如果不注释掉可能在```service nginx start```时输出一堆无用的信息。注释掉可以正常工作。

对于centos 6的系统，把这个文件拷贝到init.d目录

```
sudo cp nginx /etc/init.d/nginx
```
对于centos 7，使用的是systemctl的，可以复制到这个目录:

```
sudo cp nginx /usr/lib/systemd/system/nginx.service
# 然后重新载入systemctl
sudo systemctl daemon-reload
```


也可以参考： [http://matsumpter.com/blog/2014/openresty-centos-7-systemd-script/](http://matsumpter.com/blog/2014/openresty-centos-7-systemd-script/ "http://matsumpter.com/blog/2014/openresty-centos-7-systemd-script/")

## 后续 ##
*关于nginx的配置*