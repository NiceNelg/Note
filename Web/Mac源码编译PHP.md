# Mac源码编译PHP

---

1. [下载源码](https://php.net)

2. 安装依赖

   1. `brew install libpng`

3. 解压源码，编译安装

   ```shell
   ./configure \
   --prefix=/usr/local/php \
   --with-config-file-path=/usr/local/php/etc \
   --enable-fpm \
   --enable-pcntl \
   --enable-mysqlnd \
   --enable-opcache \
   --enable-sockets \
   --enable-sysvmsg \
   --enable-sysvsem  \
   --enable-sysvshm \
   --enable-shmop \
   --enable-zip \
   --enable-ftp \
   --enable-soap \
   --enable-xml \
   --enable-mbstring \
   --disable-rpath \
   --disable-debug \
   --disable-fileinfo \
   --with-mysqli=mysqlnd \
   --with-pdo-mysql=mysqlnd \
   --with-pcre-regex \
   --with-iconv \
   --with-zlib \
   --with-mcrypt \
   --with-mhash \
   --with-xmlrpc \
   --with-curl \
   --with-imap-ssl
   
   make
   sudo make install
   ```
   
4. 复制配置文件

   1. `sudo cp php.ini-development /usr/local/php/etc/php.ini`
   2. `sudo cp /usr/local/php/etc/php-fpm.conf.default /usr/local/etc/php-fpm.conf`
   3. `sudo cp sapi/fpm/php-fpm /usr/local/bin`
   4. `sudo cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf`
   
5. 安装`openssl`扩展

   1. `brew install openssl`
   2. 进入源码解压包中
   3. `cd ext/openssl`
   4. `/usr/local/php/bin/phpize`
   5. `./configure --with-php-config=/usr/local/php/bin/php-config --with-openssl=/usr/local/Cellar/openssl@1.1/1.1.1d/`
   6. `make`
   7. `sudo make install`

6. 重编译`GD`库

   1. 下载安装依赖

      1. freetype：`https://mirrors.sarata.com/non-gnu/freetype/freetype-2.10.0.tar.gz`

         ```shell
         tar -zxvf freetype-2.10.0.tar.gz
         cd freetype-2.10.0
         ./configure --prefix=/usr/local/freetype
         make
         sudo make install
         ```

      2. jpegsrc：`http://distfiles.macports.org/jpeg/jpegsrc.v9c.tar.gz`

         ```shell
         tar -zxvf jpegsrc.v9c.tar.gz
         cd jpeg-9c
         ./configure --prefix=/usr/local/jpeg-9c --enable-shared --enable-static
         make
         sudo make install
         ```

      3. zlib：`http://www.zlib.net/zlib-1.2.11.tar.gz`

         ```shell
         tar -zxvf zlib-1.2.11.tar.gz
         cd zlib-1.2.11
         ./configure --prefix=/usr/local/zlib
         make
         sudo make install
         ```

      4. libpng：`http://www.libpng.org/pub/png/libpng.html`

         ```shell
         tar -zxvf libpng-1.6.37.tar.gz
         cd libpng-1.6.37
         ./configure --prefix=/usr/local/libpng
         make
         sudo make install
         ```

   2. 重编译`GD`扩展

      ```shell
      # 进入源码解压包中
      cd ext/gd
      
      /usr/local/php/bin/phpize
      
      ./configure --with-php-config=/usr/local/php/bin/php-config --with-jpeg-dir=/usr/local/jpeg --with-png-dir=/usr/local/libpng --with-freetype-dir=/usr/local/freetype
      
      make
      
      sudo make install
      ```

7. `php-fpm`相关指令

   > `php 5.3.3`以后的`php-fpm`不再支持`php-fpm`以前具有的`/usr/local/php/sbin/php-fpm (start|stop|reload)`等命令，需要使用信号进行控制

   1. 启动：`/usr/local/php/sbin/php-fpm`

   2. 信号控制：

      > 先查找`php-fpm`的主进程id，然后根据pid进行下面的控制

      1. 立刻终止：`kill INT <pid>`
      2. 平滑终止：`kill QUIT <pid>`
      3.  重新打开日志文件：`kill USR1 <pid> `
      4. 平滑重载所有worker进程并重新载入配置和二进制模块：`kill USR2 <pid>`

8. 使用`php-fpm`进行`cgi`通讯的注意点

   1. 若要使用`Unix domain socket`方式进行通讯，需要修改`/usr/local/php/etc/php-fpm.d/www.conf`中的`listen`字段，可修改为`/tmp/php-cgi.sock`，指定有名管道的路径。
   2. 当使用`Unix domain socket`方式进行通讯并指定有名管道的路径后，需要注意有名管道的读写权限，可以在`listen.mode`字段中修改有名管道权限，推荐修改为`0666`。

