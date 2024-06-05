一、准备LNMP
```
# yum安装nginx
yum install nginx -y

# 修改配置
vi  /etc/nginx/conf.d/default.conf
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

# 启动nginx
nginx

# 设置nginx开机自启动
chkconfig nginx on

# 访问外网 HTTP 服务
http://xx.xx.xx.xx/

# yum安装mysql
yum install mysql-server -y

# 设置mysql开机自启
chkconfig mysqld on

# 重启mysql
service mysqld restart

# 设置mysql的root密码
/usr/bin/mysqladmin -u root password '123456'
```

二、搭建php环境
```
# yum安装php
yum install php php-fpm php-mysql -y

# 启动php-fpm进程
service php-fpm start

# 查看php-fpm进程监听哪个端口
netstat -nlpt | grep php-fpm

# 设置php-fpm开机自启
chkconfig php-fpm on
```

三、配置nginx并运行php程序
```
# 新建php.conf文件
touch /etc/nginx/conf.d/php.conf

# 修改文件，配置nginx端口
vi /etc/nginx/conf.d/php.conf
server {
    listen 8000;
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    location ~ \.php$ {
        root           /usr/share/php;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}

# 重启nginx服务
service nginx restart

# 新建info.php文件
touch /usr/share/php/info.php

# 配置 info.php
vi /usr/share/php/info.php
<?php phpinfo(); ?>

# 访问info.php页面
http://xx.xx.xx.xx:8000/info.php
```

四、安装并配置wordpress
```
# yum安装wordpress
yum install wordpress -y

# wordpress源代码目录
/usr/share/wordpress

# 进入mysql
mysql -uroot --password='123456'

# 为wordpress创建数据库
CREATE DATABASE wordpress;

#退出mysql环境
exit

#修改wordpress配置
vi /usr/share/wordpress/wp-config.php
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
define('DB_PASSWORD', '123456');

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

# 默认的 Server 监听 80 端口，与 WordPress 的服务端口冲突，将其重命名为 .bak 后缀以禁用默认配置
cd /etc/nginx/conf.d/ && mv default.conf defaut.conf.bak 

# 创建 wordpress.conf
touch /etc/nginx/conf.d/wordpress.conf

# 配置nginx
vi /etc/nginx/conf.d/wordpress.conf
server {
    listen 80;
    root /usr/share/wordpress;
    location / {
        index index.php index.html index.htm;
        try_files $uri $uri/ /index.php index.php;
    }
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}

# 重新加载nginx进程
nginx -s reload

# 访问博客地址
http://106.55.241.193:80/wp-admin/install.php

```
