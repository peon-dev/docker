FROM php:8.1-cli as dev

ENV COMPOSER_MEMORY_LIMIT=-1

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Very convenient PHP extensions installer: https://github.com/mlocati/docker-php-extension-installer
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

RUN mkdir /.composer \
    && chown 1000:1000 /.composer

RUN apt-get update && apt-get install -y \
    git \
    zip \
    # gearman dependencies:
    libgearman-dev \
    libevent-dev \
    uuid

# Not available via docker-php-extension-installer yet
# see https://github.com/mlocati/docker-php-extension-installer/issues/495
RUN pecl install gearman

RUN install-php-extensions \
    amqp \
    apcu \
    ast \
    bcmath \
    blackfire \
    bz2 \
    calendar \
    csv \
    decimal \
    event \
    exif \
    gd \
    gettext \
    gnupg \
    http \
    igbinary \
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
    odbc \
    opcache \
    pcntl \
    pcov \
    pdo_mysql \
    pdo_pgsql \
    pdo_sqlsrv \
    pgsql \
    rdkafka \
    redis \
    soap \
    sockets \
    uuid \
    xdebug \
    xlswriter \
    xmldiff \
    xmlrpc \
    xsl \
    yaml \
    zip

USER 1000:1000
