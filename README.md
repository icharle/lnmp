# Centos7编译安装lnmp
## 杂说
lnmp指的是Linux、Nginx、MySQL、PHP这四个的第一个字母的缩写，也有叫一个lamp的，同理它是Linux、Apache、MySQL、PHP这四个的第一个字母的缩写。很明显，他们的区别就是Nginx与Apache的区别。之前，Apache占据了服务器领域的半壁江山，随着Nginx发展，高并发、轻量的优势，近几年逐渐走在大家的视野，正因为高并发、轻量优势，这次博主就来编译安装下lnmp环境。

## 准备阶段
* 先是需要升级下系统的软件包
  
  ```
  yum update -y
  ```
  
* 安装编译所需要的依赖包和库
  
  ```
  yum install -y make apr* autoconf automake curl curl-devel gcc gcc-c++  cmake  gtk+-devel zlib-devel openssl openssl-devel pcre-devel gd kernel keyutils patch perl kernel-headers compat* cpp glibc libgomp libstdc++-devel keyutils-libs-devel  libarchive   libsepol-devel libselinux-devel krb5-devel libXpm* freetype freetype-devel freetype* fontconfig fontconfig-devel libjpeg* libpng* php-common php-gd gettext gettext-devel ncurses* libtool* libxml2 libxml2-devel patch policycoreutils bison zlib pcre libxslt libxslt-devel libxml2 libxml2-devel libjpeg-devel libpng-devel freetype-devel libmcrypt-devel zip unzip gzip --skip-broken
  ```

    ```
    yum install -y zlib pcre pcre-devel openssl openssl-devel
    ```
* 开放端口(若是开启防火墙，必须开放，否则无法访问)
  
  ```
  firewall-cmd --zone=public --add-service=http --permanent   #80端口
  firewall-cmd --zone=public --add-service=https --permanent  #443端口
  firewall-cmd --zone=public --add-service=mysql --permanent  #3306端口
  systemctl restart firewalld.service                         #重启防火墙
  ```

## 编译安装Nginx

* 配置用户组及组帐户

  ```
  groupadd www
  useradd www -s /sbin/nologin -g www  #创建不可登录的组帐户
  ```
  
* 下载安装Nginx  

  ```
  # 下载
  wget http://nginx.org/download/nginx-1.12.2.tar.gz
  
  # 解压
  tar zxvf nginx-1.12.2.tar.gz     
  cd nginx-1.12.2/
  
  # 配置参数
  ./configure \
  --prefix=/usr/local/nginx \
  --user=www \
  --group=www \
  --without-http_memcached_module \
  --with-http_stub_status_module \
  --with-http_ssl_module \
  --with-pcre \
  --with-http_gzip_static_module \
  --with-http_realip_module \
  --with-http_sub_module
  
  # 安装
  make && make install
  ```
  
* 使用
  
  ```
  # 开启Nginx
  /usr/local/nginx/sbin/nginx
  
  # 快速停止Nginx
  /usr/local/nginx/sbin/nginx -s stop
  
  # 正常停止Nginx
  /usr/local/nginx/sbin/nginx -s quit
  
  # 重新加载配置文件
  /usr/local/nginx/sbin/nginx -s reload
  
  # 查看Nginx进程
  ps aux|grep nginx
  ```
  上面的命令太长，不方便使用，下面介绍使用脚本缩短命令:
  
  ```
  # 创建文件
  vi /etc/init.d/nginx
  
  # 将下面脚本复制到文件中
  #!/bin/bash
  # nginx Startup script for the Nginx HTTP Server
  # it is v.0.0.2 version.
  # chkconfig: - 85 15
  # description: Nginx is a high-performance web and proxy server.
  #              It has a lot of features, but it's not for everyone.
  # processname: nginx
  # pidfile: /usr/local/nginx/logs/nginx.pid
  # config: /usr/local/nginx/conf/nginx.conf
  nginxd=/usr/local/nginx/sbin/nginx
  nginx_config=/usr/local/nginx/conf/nginx.conf
  nginx_pid=/usr/local/nginx/logs/nginx.pid
  RETVAL=0
  prog="nginx"
  # Source function library.
  . /etc/rc.d/init.d/functions
  # Source networking configuration.
  . /etc/sysconfig/network
  # Check that networking is up.
  [ "${NETWORKING}" = "no" ] && exit 0
  [ -x $nginxd ] || exit 0
  # Start nginx daemons functions.
  start() {
  if [ -e $nginx_pid ];then
     echo "nginx already running...."
     exit 1
  fi
     echo -n $"Starting $prog: "
     daemon $nginxd -c ${nginx_config}
     RETVAL=$?
     echo
     [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
     return $RETVAL
   }
   # See how we were called.
   case "$1" in
   start)
          start
          ;;
   stop)
          stop
          ;;
   reload)
          reload
          ;;
   restart)
          stop
          start
          ;;
   status)
          status $prog
          RETVAL=$?
          ;;
   *)
          echo $"Usage: $prog {start|stop|restart|reload|status|help}"
          exit 1
   esac
   exit $RETVAL
  ```
  
  如果你懒得复制，你可以下载下面文件即可
  
  ```
  # 下载脚本
  wget https://raw.githubusercontent.com/icharle/lnmp/master/nginx
  
  # 移动到/etc/init.d/
  mv nginx /etc/init.d/nginx
  ```
  
  下面设置脚本权限以及设置Nginx开机自启动
  
  ```
  # 设置权限
  chmod 755 /etc/init.d/nginx
  
  #设置Nginx开机自启动
  chkconfig --add nginx
  chkconfig --level 345 nginx on
  ```
  
* 脚本下使用

  ```
  # 开启
  service nginx start
  
  # 停止
  service nginx stop
  
  # 重启
  service nginx restart
  
  # 重新加载配置文件
  service nginx reload
  
  # 查看状态
  service nginx status
  ```

![43-1](https://icharle-1251944239.cosgz.myqcloud.com/%E5%8D%9A%E5%AE%A2/lnmp/43-1.png)

## MySQL安装

* 配置用户组及组帐户

  ```
  groupadd mysql
  useradd mysql -s /sbin/nologin -g mysql  #创建不可登录的组帐户
  ```
  
* 下载安装MySQL 

  ```
  # 下载
  wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.20.tar.gz
  
  # 解压
  tar zxvf mysql-5.7.20
  cd mysql-5.7.20
  
  # 新建文件夹及文件(用于存储数据库文件及日志文件)
  mkdir -p /var/mysql/data
  mkdir -p /var/mysql/logs
  touch /var/mysql/logs/mysql.pid
  touch /var/mysql/logs/mysql-error.log
  touch /var/mysql/logs/mysql-slow.log
  
  # 更改属性组
  chown -R mysql:mysql /var/mysql/data
  chown -R mysql:mysql /var/mysql/logs
  chown -R mysql:mysql /var/mysql/logs/mysql.pid
  chown -R mysql:mysql /var/mysql/logs/mysql-error.log
  chown -R mysql:mysql /var/mysql/logs/mysql-slow.log
  
  # 配置参数
  cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
  -DMYSQL_DATADIR=/var/mysql/data \
  -DDOWNLOAD_BOOST=1 -DWITH_BOOST=boost/boost_1_59_0/ \
  -DMYSQL_USER=mysql -DMYSQL_TCP_PORT=3306 \
  -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
  -DDEFAULT_CHARSET=utf8 \
  -DDEFAULT_COLLATION=utf8_general_ci \
  -DWITH_MYISAM_STORAGE_ENGINE=1 \
  -DWITH_INNOBASE_STORAGE_ENGINE=1 \
  -DWITH_MEMORY_STORAGE_ENGINE=1 \
  -DWITH_READLINE=1 -DENABLED_LOCAL_INFILE=1
  
  # 安装
  make && make install
  ```
  
* 配置MySQL配置文件

  ```
  # 移除系统自带配置文件
  mv /etc/my.cnf /etc/my.cnf-`date +%F`
  
  # 创建配置文件并将下面复制其中
  vi /etc/my.cnf
  
  # 该配置文件供参考
  [mysqld]
  port = 3306
  socket = /tmp/mysql.sock
  basedir = /usr/local/mysql
  datadir = /var/mysql/data
  pid-file = /var/mysql/logs/mysql.pid
  user = mysql
  server-id = 1

  init-connect = 'SET NAMES utf8mb4'
  character-set-server = utf8mb4

  #skip-name-resolve
  #skip-networking
  back_log = 300

  max_connections = 1000
  max_connect_errors = 6000
  open_files_limit = 65535
  table_open_cache = 128
  max_allowed_packet = 4M
  binlog_cache_size = 1M
  max_heap_table_size = 8M
  tmp_table_size = 16M

  read_buffer_size = 2M
  read_rnd_buffer_size = 8M
  sort_buffer_size = 8M
  join_buffer_size = 8M
  key_buffer_size = 4M

  thread_cache_size = 8

  query_cache_type = 1
  query_cache_size = 8M
  query_cache_limit = 2M

  ft_min_word_len = 4

  log_bin = mysql-bin
  binlog_format = mixed
  expire_logs_days = 30

  log_error = /var/mysql/logs/mysql-error.log
  slow_query_log = 1
  long_query_time = 1
  slow_query_log_file = /var/mysql/logs/mysql-slow.log

  performance_schema = 0
  explicit_defaults_for_timestamp

  #lower_case_table_names = 1

  skip-external-locking

  default_storage_engine = InnoDB
  #default-storage-engine = MyISAM
  innodb_file_per_table = 1
  innodb_open_files = 500
  innodb_buffer_pool_size = 64M
  innodb_write_io_threads = 4
  innodb_read_io_threads = 4
  innodb_thread_concurrency = 0
  innodb_purge_threads = 1
  innodb_flush_log_at_trx_commit = 2
  innodb_log_buffer_size = 2M
  innodb_log_file_size = 32M
  innodb_log_files_in_group = 3
  innodb_max_dirty_pages_pct = 90
  innodb_lock_wait_timeout = 120

  bulk_insert_buffer_size = 8M
  myisam_sort_buffer_size = 8M
  myisam_max_sort_file_size = 10G
  myisam_repair_threads = 1

  interactive_timeout = 28800
  wait_timeout = 28800

  [mysqldump]
  quick
  max_allowed_packet = 16M

  [myisamchk]
  key_buffer_size = 8M
  sort_buffer_size = 8M
  read_buffer = 4M
  write_buffer = 4M
  EOF
  ```
  
  如果你觉得要复制很麻烦，可以直接下载
  
  ```
  # 下载脚本
  wget https://raw.githubusercontent.com/icharle/lnmp/master/my.cnf
  
  # 移动到/etc
  mv my.cnf /etc/my.cnf
  ```
  
* 配置数据库

  ```
  # 初始化数据库
  /usr/local/mysql/bin/mysqld --initialize --user=mysql
  
  # 获取临时密码
  cat /var/mysql/logs/mysql-error.log
  
  # 登录数据库
  /usr/local/mysql/bin/mysql -uroot -p    #密码为刚才的临时密码
  
  # 修改数据库密码
  ALTER USER 'root'@'localhost' identified by 'root';   #这里博主将它改为root
  khaesmVq#5->
  ```
  
* 优化操作

  ```
  # 添加数据库开机自启动
  cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld  
  chmod +x /etc/init.d/mysqld
  chkconfig --add mysqld
  chkconfig mysqld on
  
  # 设置系统环境变量(以后不用输入mysql安装路径就可以使用mysql命令)
  vim /etc/profile
  
  # 在最后添加这行代码
  export PATH=$PATH:/usr/local/mysql/bin
  
  # 使其生效
  source /etc/profile
  
  # 修改MySQL访问权限，使其可以外网访问
  use mysql;
  update user set host = '%' where user ='root';
  flush privileges;
  ```
  
  ![43-3](https://icharle-1251944239.cosgz.myqcloud.com/%E5%8D%9A%E5%AE%A2/lnmp/43-3.png)
  
## 安装PHP

* 下载安装PHP
  
  ```
  # 下载
  wget http://cn2.php.net/get/php-7.2.1.tar.gz/from/this/mirror
  
  # 解压
  tar zxvf mirror
  cd php-7.2.1/
  
  # 配置参数
  ./configure --prefix=/usr/local/php7 \
  --with-config-file-path=/usr/local/php7/etc \
  --with-mysqli=/usr/local/mysql/bin/mysql_config \
  --enable-mysqlnd \
  --with-mysql-sock=/tmp/mysql.sock \
  --with-gd \
  --with-iconv \
  --enable-calendar \
  --enable-opcache \
  --with-xsl \
  --with-zlib \
  --enable-xml \
  --enable-bcmath \
  --enable-shmop \
  --enable-sysvsem \
  --enable-inline-optimization \
  --enable-mbregex \
  --with-freetype-dir \
  --enable-fpm \
  --enable-mbstring \
  --enable-ftp \
  --with-png-dir \
  --with-openssl \
  --with-mhash \
  --enable-pcntl \
  --enable-sockets \
  --with-xmlrpc \
  --enable-zip \
  --enable-soap \
  --without-pear \
  --with-gettext \
  --enable-session \
  --with-curl \
  --with-jpeg-dir \
  --with-freetype-dir   \
  --with-pdo-mysql=/usr/local/mysql/
  
  # 安装
  make && make install
  ```
  
* 配置PHP

  ```
  # 设置开机自启动
  cp php-7.2.1/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
  chmod 755 /etc/init.d/php-fpm
  chkconfig --add php-fpm
  chkconfig php-fpm on
  
  # 配置php.ini
  cp php-7.2.1/php.ini-production /usr/local/php7/etc/php.ini
  
  # 删除自带配置文件
  rm -f /etc/php.ini  
  
  # 添加软链接
  ln -s /usr/local/php7/etc/php.ini /etc/php.ini  
  
  # 拷贝模板配置文件为 php-fpm 配置文件
  cp /usr/local/php7/etc/php-fpm.conf.default /usr/local/php7/etc/php-fpm.conf  
  
  # 修改文件
  vim /usr/local/php7/etc/php-fpm.conf
  将;pid = run/php-fpm.pid前面的;删除
  
  # 复制配置文件
  cp /usr/local/php7/etc/php-fpm.d/www.conf.default /usr/local/php7/etc/php-fpm.d/www.conf
  
  # 修改用户组及组用户
  vim /usr/local/php7/etc/php-fpm.d/www.conf
  将user = nobody 和 group = nobody 的 nobody 改成 www
  
  # 开启php-fpm服务
  service php-fpm start
  ```

## 配置Nginx及php-fpm

* 修改nginx.conf文件

  ```
  vim /usr/local/nginx/conf/nginx.conf
  
  #location ~ \.php$ {
  #    root           html;
  #    fastcgi_pass   127.0.0.1:9000;
  #    fastcgi_index  index.php;
  #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
  #    include        fastcgi_params;
  #}
  
  将前面#去掉，并将
  fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
  修改为:
  fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
  
  ```
  
* 优化配置
  Nginx默认的文件存放目录是`/usr/local/nginx/html/`,我们为了方便，可以将它软链接到根目录，这样大大方便我们存放。
  
  ```
  ln -s /usr/local/nginx/html/ /www
  ```
  
  再者，如果我们需要添加新主机的话，需要到默认的`nginx.conf`文件修改，但是这样容易误删出其它配置，而且新主机比较多的时候很难找出来，所以我们可以采用分开的方式将其独立开来。
  
  先在`nginx.conf`中添加`include "vhost/*.conf";`,这句话表示引入在文件夹下所有`.conf`的配置文件。
  接着新建`vhost`文件夹。
  
  ```
  cd /usr/local/nginx/conf
  mkdir vhost
  ```
  
  这样以后再增添一个主机时，就在`/usr/local/nginx/conf/vhost`中添加配置文件，并在`/www`中放对应的文件。
  
  vhost站点配置案例
  
  ```
  server {
     listen       80;
     server_name  somename  alias  another.alias;

     location / {
         root   html;
         index  index.php index.html index.htm;
     }
     
     location ~ \.php$ {
         fastcgi_pass   127.0.0.1:9000;
         fastcgi_index  index.php;
         fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
         include        fastcgi_params;
        }
  }
  ```

![43-2](https://icharle-1251944239.cosgz.myqcloud.com/%E5%8D%9A%E5%AE%A2/lnmp/43-2.png)

## 安装phpMyAdmin

```
# 下载安装
wget https://files.phpmyadmin.net/phpMyAdmin/4.7.7/phpMyAdmin-4.7.7-all-languages.tar.gz

# 解压
tar zxvf phpMyAdmin-4.7.7-all-languages.tar.gz

# 移动文件夹
mv phpMyAdmin-4.7.7-all-languages /www/phpMyAdmin

# 即可使用phpMyAdmin
```

![43-4](https://icharle-1251944239.cosgz.myqcloud.com/%E5%8D%9A%E5%AE%A2/lnmp/43-4.png)


