### 1.准备 LNMP 环境
LNMP 是 Linux、Nginx、MySQL 和 PHP 的缩写，是 WordPress 博客系统依赖的基础运行环境。我们先来准备 LNMP 环境

* 安装 Nginx

    使用 yum 安装 Nginx：
    ```
    yum install nginx -y
    ```

    修改 /etc/nginx/conf.d/default.conf，去除对 IPv6 地址的监听
    ```
    server {
        listen       80 default_server;
        # listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
    ```

    修改完成后，启动 Nginx：
    ```
    nginx
    ```

    此时，可访问实验机器外网 HTTP 服务（http://139.199.228.210）来确认是否已经安装成功。
    将 Nginx 设置为开机自动启动：
    ```
    chkconfig nginx on
    ```

* 安装 MySQL

    使用 yum 安装 MySQL：
    ```
    yum install mysql-server -y
    ```
    
    安装完成后，启动 MySQL 服务：
    ```
    service mysqld restart
    or :
    systemctl start mysqld.service
    ```

    查看 MySQL 运行状态：
    ```
    systemctl status mysqld.service
    ```

    想要进入 MySQL，需要知道 root 用户的密码，通过如下命令可以在日志文件中找出密码：
    ```
    grep "password" /var/log/mysqld.log   
    ```

    设置 MySQL 账户 root 密码：
    ```
    /usr/bin/mysqladmin -u root password 'MyPas$word4Word_Press'
    ```

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


### 2.安装并配置 WordPress
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
    ```
    <?php
    /**
    * The base configuration for WordPress
    *
    * The wp-config.php creation script uses this file during the
    * installation. You don't have to use the web site, you can
    * copy this file to "wp-config.php" and fill in the values.
    *
    * This file contains the following configurations:
    *
    * * MySQL settings
    * * Secret keys
    * * Database table prefix
    * * ABSPATH
    *
    * @link https://codex.wordpress.org/Editing_wp-config.php
    *
    * @package WordPress
    */

    // ** MySQL settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define('DB_NAME', 'wordpress');

    /** MySQL database username */
    define('DB_USER', 'root');

    /** MySQL database password */
    define('DB_PASSWORD', 'MyPas$word4Word_Press');

    /** MySQL hostname */
    define('DB_HOST', 'localhost');

    /** Database Charset to use in creating database tables. */
    define('DB_CHARSET', 'utf8');

    /** The Database Collate type. Don't change this if in doubt. */
    define('DB_COLLATE', '');

    /**#@+
    * Authentication Unique Keys and Salts.
    *
    * Change these to different unique phrases!
    * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
    * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
    *
    * @since 2.6.0
    */
    define('AUTH_KEY',         'put your unique phrase here');
    define('SECURE_AUTH_KEY',  'put your unique phrase here');
    define('LOGGED_IN_KEY',    'put your unique phrase here');
    define('NONCE_KEY',        'put your unique phrase here');
    define('AUTH_SALT',        'put your unique phrase here');
    define('SECURE_AUTH_SALT', 'put your unique phrase here');
    define('LOGGED_IN_SALT',   'put your unique phrase here');
    define('NONCE_SALT',       'put your unique phrase here');

    /**#@-*/

    /**
    * WordPress Database Table prefix.
    *
    * You can have multiple installations in one database if you give each
    * a unique prefix. Only numbers, letters, and underscores please!
    */
    $table_prefix  = 'wp_';

    /**
    * See http://make.wordpress.org/core/2013/10/25/the-definitive-guide-to-disabling-auto-updates-in-wordpress-3-7
    */

    /* Disable all file change, as RPM base installation are read-only */
    define('DISALLOW_FILE_MODS', true);

    /* Disable automatic updater, in case you want to allow
    above FILE_MODS for plugins, themes, ... */
    define('AUTOMATIC_UPDATER_DISABLED', true);

    /* Core update is always disabled, WP_AUTO_UPDATE_CORE value is ignore */

    /**
    * For developers: WordPress debugging mode.
    *
    * Change this to true to enable the display of notices during development.
    * It is strongly recommended that plugin and theme developers use WP_DEBUG
    * in their development environments.
    *
    * For information on other constants that can be used for debugging,
    * visit the Codex.
    *
    * @link https://codex.wordpress.org/Debugging_in_WordPress
    */
    define('WP_DEBUG', false);

    /* That's all, stop editing! Happy blogging. */

    /** Absolute path to the WordPress directory. */
    if ( !defined('ABSPATH') )
        define('ABSPATH', '/usr/share/wordpress');

    /** Sets up WordPress vars and included files. */
    require_once(ABSPATH . 'wp-settings.php');
    ```

* 配置 Nginx

    WordPress 已经安装完毕，我们配置 Nginx 把请求转发给 PHP-FPM 来处理
    首先，重命名默认的配置文件：
    ```
    cd /etc/nginx/conf.d/
    mv default.conf defaut.conf.bak
    ```

    在 /etc/nginx/conf.d 创建 wordpress.conf 配置，参考下面的内容：
    wordpress.conf
    ```
    server {
        listen 80;
        root /usr/share/wordpress;
        location / {
            index index.php index.html index.htm;
            try_files $uri $uri/ /index.php index.php;
        }
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        location ~ .php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
    }
    ```

    配置后，通知 Nginx 进程重新加载：
    ```
    nginx -s reload
    ```

## 3.准备域名和解析
* 域名注册

    如果您还没有域名，可以在腾讯云上选购

* 域名解析

    域名购买完成后, 需要将域名解析到实验云主机上

    域名设置解析后需要过一段时间才会生效，通过 ping 命令检查域名是否生效 ，如：
    ```
    ping www.yourdomain.com
    ```

    如果 ping 命令返回的信息中含有你设置的解析的 IP 地址，说明解析成功。大功告成！

    恭喜，您的 WordPress 博客已经部署完成，您可以通过浏览器访问博客查看效果。
    
    通过IP地址查看：
    博客访问地址：http://132.232.246.53/wp-admin/install.php

    通过域名查看：
    博客访问地址：http://www.yourdomain.com/wp-admin/install.php，其中替换 www.yourdomain.com 为之前申请的域名。