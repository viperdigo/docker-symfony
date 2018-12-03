FROM php:5.6-fpm
ARG TIMEZONE

RUN apt-get update && apt-get install -y git unzip openssl procps acl zlib1g-dev php5.6 libz-dev libmemcached-dev libmemcached11 libmemcachedutil2 build-essential

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo ${TIMEZONE} > /etc/timezone
RUN printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini

# Install extensions
RUN docker-php-ext-install pdo pdo_mysql bcmath zip

# Install Redis extension
RUN pecl install -o -f redis && rm -rf /tmp/pear && docker-php-ext-enable redis

# Install xdebug
RUN pecl install xdebug-2.5.5
RUN pecl install memcache
RUN pecl install memcached-2.2.0 
RUN docker-php-ext-enable xdebug
RUN echo "memory_limit=-1" > /usr/local/etc/php/conf.d/memory-limit.ini
RUN echo "extension=memcache.so" >> /usr/local/etc/php/conf.d/memcache.ini
RUN echo "extension=memcached.so" >> /usr/local/etc/php/conf.d/memcached.ini 
RUN echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_connect_back=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.idekey=\"PHPSTORM\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_port=9001" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# Create an alias for Symfony2 console
RUN echo 'alias symfony="php app/console"' >> ~/.bashrc

WORKDIR /var/www/symfony
