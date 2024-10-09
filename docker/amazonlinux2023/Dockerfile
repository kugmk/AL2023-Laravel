FROM amazonlinux:2023

RUN dnf update -y
RUN dnf install -y systemd
RUN dnf swap -y libcurl-minimal libcurl-full && \
    dnf swap -y curl-minimal curl-full
RUN dnf install -y libzip libzip-devel
RUN dnf install -y php8.3
RUN dnf install -y php8.3-common
##RUN dnf install -y php8.3-json 
RUN dnf install -y php8.3-devel
##RUN dnf install -y php8.3-mysqli
RUN dnf install -y php8.3-mysqlnd
RUN dnf install -y php8.3-gd
RUN dnf install -y php8.3-xml
RUN dnf install -y php8.3-fpm
##RUN dnf install -y php8.3-curl
RUN dnf install -y php8.3-pdo
##RUN dnf install -y php8.3-pear
RUN dnf install -y php8.3-bcmath
RUN dnf install -y php8.3-cli
RUN dnf install -y php8.3-intl
RUN dnf install -y php8.3-ldap
RUN dnf install -y php8.3-opcache
##RUN dnf install -y php8.3-readline
#RUN dnf install -y php8.3-redis
RUN dnf install -y php8.3-soap
RUN dnf install -y php8.3-zip

# php-redis
WORKDIR /tmp
RUN dnf install -y gcc make git
RUN git clone https://github.com/phpredis/phpredis.git -b 6.0.2
WORKDIR  phpredis
RUN phpize
RUN ./configure
RUN make && make install
RUN cp ./modules/redis.so /lib64/php8.3/modules/
COPY  php-redis.ini /etc/php.d/50-redis.ini
WORKDIR /tmp
RUN rm -rf phpredis

# Composer
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');" && \
    mv composer.phar /usr/local/bin/composer

# scheduleのコマンドの設定
COPY laravel-schedule.service /usr/lib/systemd/system/
COPY laravel-schedule.timer /usr/lib/systemd/system/
#RUN systemctl start laravel-schedule.timer コンテナ内で実行

RUN mkdir -p /run/php-fpm
RUN chmod 777 /run/php-fpm
CMD ["php-fpm", "-F"]

WORKDIR /var/www/html
