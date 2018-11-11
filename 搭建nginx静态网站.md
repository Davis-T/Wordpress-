## 1.准备 LNMP 环境
LNMP 是 Linux、Nginx、MySQL 和 PHP 的缩写，是 WordPress 博客系统依赖的基础运行环境。我们先来准备 LNMP 环境

---

* 安装 Nginx

    使用 yum 安装 Nginx：
    yum install nginx -y

    修改 /etc/nginx/conf.d/default.conf，去除对 IPv6 地址的监听

    修改完成后，启动 Nginx：
    nginx

    此时，可访问实验机器外网 HTTP 服务（http://139.199.228.210）来确认是否已经安装成功。
    将 Nginx 设置为开机自动启动：
    chkconfig nginx on

* 安装 MySQL

    使用 yum 安装 MySQL：
    yum install mysql-server -y

    安装完成后，启动 MySQL 服务：
    service mysqld restart

    设置 MySQL 账户 root 密码：
    /usr/bin/mysqladmin -u root password 'MyPas$word4Word_Press'

    将 MySQL 设置为开机自动启动：
    chkconfig mysqld on

* 安装 PHP

    使用 yum 安装 PHP：
    yum install php-fpm php-mysql -y

    安装之后，启动 PHP-FPM 进程：
    service php-fpm start

    启动之后，可以使用下面的命令查看 PHP-FPM 进程监听哪个端口 
    netstat -nlpt | grep php-fpm

    把 PHP-FPM 也设置成开机自动启动：
    chkconfig php-fpm on


## 2.安装并配置 WordPress
---
* 安装 WordPress

    配置好 LNMP 环境后，继续使用 yum 来安装 WordPress：
    yum install wordpress -y

    安装完成后，就可以在 /usr/share/wordpress 看到 WordPress 的源代码了。

* 配置数据库

    进入 MySQL：
    mysql -uroot --password='MyPas$word4Word_Press'

    为 WordPress 创建一个数据库：
    CREATE DATABASE wordpress;

    MySQL 部分设置完了，我们退出 MySQL 环境：
    exit

    把上述的 DB 配置同步到 WordPress 的配置文件中，可参考下面的配置：
    wp-config.php

* 配置 Nginx

    WordPress 已经安装完毕，我们配置 Nginx 把请求转发给 PHP-FPM 来处理
    首先，重命名默认的配置文件：
    cd /etc/nginx/conf.d/
    mv default.conf defaut.conf.bak

    在 /etc/nginx/conf.d 创建 wordpress.conf 配置，参考下面的内容：
    wordpress.conf

    配置后，通知 Nginx 进程重新加载：
    nginx -s reload

## 3.准备域名和解析
---
* 域名注册

    如果您还没有域名，可以在腾讯云上选购

* 域名解析

    域名购买完成后, 需要将域名解析到实验云主机上，实验云主机的 IP 为：
    132.232.246.53

    博客访问地址：http://132.232.246.53/wp-admin/install.php
