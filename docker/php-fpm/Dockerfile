FROM php:7.3.13-fpm

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    git \
    libz-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libsodium-dev \
    libgmp-dev \
    zlib1g-dev \
    libicu-dev \
    g++ \
    libhiredis-dev \
    libzip-dev \
    && rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd \
        --with-jpeg-dir=/usr/lib \
        --with-freetype-dir=/usr/include/freetype2 && \
    docker-php-ext-configure zip \
        --with-libzip && \
    docker-php-ext-configure intl && \
    docker-php-ext-install \
        gd \
        pdo_mysql \
        gmp \
        intl \
        exif \
        bcmath \
        opcache \
        zip

RUN apt-get update -yqq && \
    apt-get install -y apt-utils && \
    pecl channel-update pecl.php.net

######################################
# phpiredis (requires hiredis):
######################################
RUN git clone https://github.com/francislavoie/phpiredis.git && \
    ( \
        cd phpiredis && \
        phpize && ./configure --enable-phpiredis && \
        make && make install && \
        docker-php-ext-enable phpiredis \
    ) && \
    rm -rf phpiredis

######################################
# Final Touches:
######################################

# Copy the PHP config files (app, fpm, opcache)
COPY ./php.ini /usr/local/etc/php/php.ini
COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./laravel.pool.conf /usr/local/etc/php-fpm.d/
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Add non-root user, prevents files being created with root permissions on host machine
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}
RUN groupmod -g ${PGID} www-data && \
    usermod -u ${PUID} www-data

WORKDIR /var/www/html

CMD ["php-fpm"]

EXPOSE 9000
