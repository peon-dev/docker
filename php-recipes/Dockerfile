FROM php:8.2-cli

ENV COMPOSER_MEMORY_LIMIT=-1

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Very convenient PHP extensions installer: https://github.com/mlocati/docker-php-extension-installer
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

RUN mkdir /.composer \
    && chown 1000:1000 /.composer

RUN apt-get update && apt-get install -y \
    git \
    zip \
    libevent-dev \
    uuid \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Not available via docker-php-extension-installer yet
# see https://github.com/mlocati/docker-php-extension-installer/issues/495
# RUN pecl install gearman

RUN install-php-extensions \
    amqp \
    apcu \
    ast \
    bcmath \
    blackfire \
    calendar \
    csv \
    decimal \
    event \
    exif \
    gd \
    gettext \
    http \
    imagick \
    imap \
    intl \
    json_post \
    ldap \
    lzf \
    mcrypt \
    memcache \
    memcached \
    mongodb \ \
    mysqli \
    oauth \
    opcache \
    pcntl \
    pcov \
    pdo_mysql \
    pdo_pgsql \
    redis \
    soap \
    sockets \
    uuid \
    xdebug \
    xmlrpc \
    xsl \
    yaml \
    zip

USER 1000:1000
