FROM phusion/baseimage:bionic-1.0.0

RUN DEBIAN_FRONTEND=noninteractive
RUN locale-gen en_US.UTF-8

ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV LC_CTYPE=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV TERM xterm

RUN apt-get install -y software-properties-common && \
    add-apt-repository -y ppa:ondrej/php

RUN apt-get update && \
    apt-get install -y \
        --allow-downgrades \
        --allow-remove-essential \
        --allow-change-held-packages \
        php7.3-cli \
        php7.3-common \
        php7.3-curl \
        php7.3-intl \
        php7.3-json \
        php7.3-xml \
        php7.3-mbstring \
        php7.3-mysql \
        php7.3-sqlite \
        php7.3-sqlite3 \
        php7.3-zip \
        # php7.3-gmp \
        php7.3-bcmath \
        php7.3-memcached \
        php7.3-gd \
        php7.3-phpdbg \
        # php7.3-soap \
        php7.3-dev \
        pkg-config \
        libcurl4-openssl-dev \
        libedit-dev \
        libssl-dev \
        libxml2-dev \
        libhiredis-dev \
        xz-utils \
        libsqlite3-dev \
        sqlite3 \
        git \
        curl \
        vim \
        nano \
        unzip \
    && apt-get clean

###########################################
# Composer:
###########################################
RUN curl -s https://getcomposer.org/installer | php && \
    echo "export PATH=${PATH}:${APP_CODE_PATH}/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer && \
    . ~/.bashrc

###########################################
# phpiredis extension (to manage Horizon):
###########################################
RUN git clone https://github.com/francislavoie/phpiredis.git && \
    ( \
        cd phpiredis && \
        phpize && ./configure --enable-phpiredis && \
        make && make install && \
        echo "extension=phpiredis.so" >> /etc/php/7.3/cli/php.ini \
    ) && \
    rm -rf phpiredis

###########################################
# User setup:
###########################################
USER root

# Add non-root user, prevents files being created with root permissions on host machine
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}
RUN apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} laravel && \
    useradd -u ${PUID} -g laravel -m laravel -G docker_env && \
    usermod -p "*" laravel

###########################################
# Timezone:
###########################################
ARG TZ=UTC
ENV TZ ${TZ}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################
# Composer for root user:
###########################################
USER root
COPY ./composer.json /home/laravel/.composer/composer.json
RUN chown -R laravel:laravel /home/laravel/.composer

USER laravel
ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}
RUN composer global install && \
    if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
        composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
    ;fi && \
    echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc && \
    echo 'export PATH="${APP_CODE_PATH}/vendor/bin:$PATH"' >> ~/.bashrc

###########################################
# Cron:
###########################################
USER root
COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

###########################################
# Node / NVM:
###########################################
USER laravel
ARG NODE_VERSION=node
ENV NODE_VERSION ${NODE_VERSION}
ARG INSTALL_NPM_VUE_CLI=false
ARG NPM_REGISTRY
ENV NPM_REGISTRY ${NPM_REGISTRY}
ENV NVM_DIR /home/laravel/.nvm

RUN mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.37.2/install.sh | bash && \
    . $NVM_DIR/nvm.sh && \
    nvm install ${NODE_VERSION} && \
    nvm use ${NODE_VERSION} && \
    nvm alias ${NODE_VERSION} && \
    if [ ${NPM_REGISTRY} ]; then \
        npm config set registry ${NPM_REGISTRY} \
    ;fi && \
    if [ ${INSTALL_NPM_VUE_CLI} = true ]; then \
        npm install -g @vue/cli \
    ;fi && \
    ln -s `npm bin --global` /home/laravel/.node-bin && \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc

# Configure Node for root
USER root
RUN echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="/home/laravel/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc

ENV PATH $PATH:/home/laravel/.node-bin
RUN find $NVM_DIR -type f -name node -exec ln -s {} /usr/local/bin/node \; && \
    NODE_MODS_DIR="$NVM_DIR/versions/node/$(node -v)/lib/node_modules" && \
    ln -s $NODE_MODS_DIR/npm/bin/npm-cli.js /usr/local/bin/npm && \
    ln -s $NODE_MODS_DIR/npm/bin/npx-cli.js /usr/local/bin/npx && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue /usr/local/bin/vue && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue-init /usr/local/bin/vue-init && \
    ln -s $NODE_MODS_DIR/vue-cli/bin/vue-list /usr/local/bin/vue-list && \
    if [ ${NPM_REGISTRY} ]; then \
        . ~/.bashrc && npm config set registry ${NPM_REGISTRY} \
    ;fi

###########################################
# Yarn:
###########################################
USER laravel
ARG YARN_VERSION=latest
ENV YARN_VERSION ${YARN_VERSION}
RUN . "$NVM_DIR/nvm.sh" && \
    if [ ${YARN_VERSION} = "latest" ]; then \
        curl -o- -L https://yarnpkg.com/install.sh | bash; \
    else \
        curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version ${YARN_VERSION}; \
    fi && \
    echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.yarn/bin:$PATH"' >> ~/.bashrc

USER root
RUN echo "" >> ~/.bashrc && \
    echo 'export YARN_DIR="/home/laravel/.yarn"' >> ~/.bashrc && \
    echo 'export PATH="$YARN_DIR/bin:$PATH"' >> ~/.bashrc

# Add PATH for YARN
ENV PATH $PATH:/home/laravel/.yarn/bin

###########################################
# nasm (for laravel-mix):
###########################################
USER root
RUN apt-get update -yqq && \
    apt-get -yqq install nasm

###########################################
# MySQL CLI client and MyCLI:
###########################################
USER root
RUN apt-get update -yqq && \
    apt-get -y install mysql-client mycli

###########################################
# Cleanup:
###########################################
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www/html
