ARG INSTALL_PHP_VERSION

FROM php:${INSTALL_PHP_VERSION}-fpm-alpine

LABEL maintainer="Bryan Lee <me@lbryany.com>"

#下载镜像后要重新声明变量
ARG INSTALL_PHP_VERSION
ARG PHP_RUNENV="prod"

# 安装需要的基本扩展
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/' /etc/apk/repositories

# 尝试用libressl替代openssl
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \ 
    curl \
    libmemcached-dev \
    zlib-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    freetype-dev \
    openssl-dev \ 
    libmcrypt-dev \
    build-base \
    g++ \
    autoconf
    

# 启用扩展
RUN docker-php-ext-install mcrypt

RUN docker-php-ext-install pdo_mysql

# 启用GD库
# 判断是否是5.6，5.6还需要安装
RUN if [ ${INSTALL_PHP_VERSION} = "5.6" ]; then \
        docker-php-ext-install gd \
    ;fi && docker-php-ext-configure gd \
        --enable-gd-native-ttf \
        --with-jpeg-dir=/usr/lib \
        --with-freetype-dir=/usr/include/freetype2 && \
    docker-php-ext-install gd

### 安装zip扩展
RUN set -xe && \
    apk update && \
    pecl channel-update https://pecl.php.net/channel.xml && \
    apk add --no-cache  \
        libzip-dev zip unzip && \
        docker-php-ext-configure zip --with-libzip && \
        docker-php-ext-install zip && \ 
        php -m | grep -q 'zip'

### 安装Redis扩展
ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        pecl install -o -f redis-4.3.0; \
    else \
        pecl install -o -f redis; \
    fi \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis && \
    php -m | grep -q 'redis' \
;fi

### 安装Swoole扩展
ARG INSTALL_SWOOLE=false;

RUN if [ ${INSTALL_SWOOLE} = true ]; then \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install swoole-2.0.10; \
    else \
      if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
        pecl install swoole-2.2.0; \
      else \
        pecl install swoole; \
      fi \
    fi && \
    docker-php-ext-enable swoole && \
    php -m | grep -q 'swoole' \
;fi

### 安装AMQP扩展
ARG INSTALL_AMQP=false

RUN if [ ${INSTALL_AMQP} = true ]; then \
    apk update && apk add --no-cache cmake rabbitmq-c-dev && \
    pecl install amqp && \
    docker-php-ext-enable amqp && \
    docker-php-ext-install sockets && \
    php -m | grep -q 'amqp' \
;fi

### 安装Opcache扩展
ARG INSTALL_OPCACHE=false

RUN if [ ${INSTALL_OPCACHE} = true ]; then \
    docker-php-ext-install opcache \
;fi

# 复制配置文件
COPY ./config/${PHP_RUNENV}/opcache.ini /usr/local/etc/php/conf.d/opcache.ini

### 安装Mysqli扩展
ARG INSTALL_MYSQLI=false

RUN if [ ${INSTALL_MYSQLI} = true ]; then \
    docker-php-ext-install mysqli \
;fi

### 安装ImageMagick扩展
ARG INSTALL_IMAGEMAGICK=false

RUN if [ ${INSTALL_IMAGEMAGICK} = true ]; then \
    apk add --no-cache imagemagick-dev imagemagick && \
    pecl install imagick && \
    docker-php-ext-enable imagick \
;fi

### 安装IMAP扩展
ARG INSTALL_IMAP=false

RUN if [ ${INSTALL_IMAP} = true ]; then \
    apk add --no-cache imap-dev krb5-dev && \
    docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
    docker-php-ext-install imap \
;fi

### 安装YML扩展
ARG INSTALL_YAML=false
 
RUN if [ ${INSTALL_YAML} = true ]; then \
    apk add --no-cache yaml-dev ; \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        pecl install -a yaml-1.3.2; \
    else \
        pecl install yaml; \
    fi && \
    docker-php-ext-enable yaml \
;fi

### 检查PHP版本
RUN set -xe; php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

### 复制配置文件

COPY ./config/${PHP_RUNENV}/php${INSTALL_PHP_VERSION}.ini /usr/local/etc/php/php.ini
COPY ./config/${PHP_RUNENV}/php-ext.ini /usr/local/etc/php/conf.d
COPY ./config/${PHP_RUNENV}/php-fpm.ini /usr/local/etc/php-fpm.d

USER root 

### 清除产生的日志以及残留文件
RUN rm -rf /var/cache/apk/* /tmp/* /var/tmp/* 

# 等会儿进去看有没有，有就删掉    rm /var/log/lastlog /var/log/faillog

### 创建用户以及用户组
RUN set -x ; \
    addgroup -g 666 -S www ; \
    adduser -u 666 -D -S -G www www && exit 0 ; exit 1

USER www

### 设置工作目录
ARG PHP_WORKDIR="/var/www"

WORKDIR ${PHP_WORKDIR}

CMD ["php-fpm"]

EXPOSE 9000