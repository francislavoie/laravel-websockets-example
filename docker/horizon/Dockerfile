FROM php:7.3.13-alpine

RUN apk add --no-cache \
    wget \
    curl \
    git \
    shadow \
    build-base \
    autoconf \
    hiredis \
    libxml2-dev \
    zlib-dev \
    libevent \
    libevent-dev \
    openssl-dev \
    gmp-dev \
    icu-dev \
    && \
    docker-php-ext-install \
        pcntl \
        mbstring \
        pdo \
        pdo_mysql \
        tokenizer \
        xml \
        sockets \
        gmp \
        bcmath \
        intl \
        && \
    # Libevent
    pecl install event && \
    # phpiredis
    apk add --no-cache --virtual .build-deps \
        hiredis-dev \
        && \
    git clone https://github.com/francislavoie/phpiredis.git && \
    ( \
        cd phpiredis && \
        phpize && ./configure --enable-phpiredis && \
        make && make install && \
        docker-php-ext-enable phpiredis \
    ) && \
    rm -rf phpiredis && \
    # Cleanup
    apk del .build-deps && \
    rm -rf /var/cache/apk/* && \
    mkdir -p /var/www/html

# Add non-root user, prevents files being created with root permissions on host machine
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}
RUN groupmod -g ${PGID} www-data && \
    usermod -u ${PUID} www-data

USER www-data

WORKDIR /var/www/html

CMD ["php", "artisan", "horizon"]
