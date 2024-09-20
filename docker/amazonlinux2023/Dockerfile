FROM amazonlinux:2023

RUN dnf update -y
RUN dnf swap -y libcurl-minimal libcurl-full && \
    dnf swap -y curl-minimal curl-full
RUN dnf install -y libzip libzip-devel
RUN dnf install -y php
RUN dnf install -y php-json 
RUN dnf install -y php-devel
RUN dnf install -y php-mbstring
RUN dnf install -y php-mysqli 
RUN dnf install -y php-mysqlnd
RUN dnf install -y php-gd
RUN dnf install -y php-xml
RUN dnf install -y php-fpm
RUN dnf install -y php-curl
RUN dnf install -y php-pdo
RUN dnf install -y php-devel
RUN dnf install -y php-pear
RUN dnf install -y php-bcmath
RUN dnf install -y php-cli
RUN dnf install -y php-intl
RUN dnf install -y php-ldap
RUN dnf install -y php-opcache
RUN dnf install -y php-readline
#RUN dnf install -y php-redis
RUN dnf install -y php-soap
RUN dnf install -y php-xml
RUN dnf install -y php-zip

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


RUN mkdir -p /run/php-fpm
RUN chmod 777 /run/php-fpm
CMD ["php-fpm", "-F"]

WORKDIR /var/www/html
