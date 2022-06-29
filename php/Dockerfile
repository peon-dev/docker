FROM php:8.1-cli as UNIT_BUILDER

RUN set -ex \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y ca-certificates mercurial build-essential libssl-dev libpcre2-dev \
    && mkdir -p /usr/lib/unit/modules /usr/lib/unit/debug-modules \
    && hg clone https://hg.nginx.org/unit \
    && cd unit \
    && hg up 1.27.0 \
    && NCPU="$(getconf _NPROCESSORS_ONLN)" \
    && DEB_HOST_MULTIARCH="$(dpkg-architecture -q DEB_HOST_MULTIARCH)" \
    && CC_OPT="$(DEB_BUILD_MAINT_OPTIONS="hardening=+all,-pie" DEB_CFLAGS_MAINT_APPEND="-Wp,-D_FORTIFY_SOURCE=2 -fPIC" dpkg-buildflags --get CFLAGS)" \
    && LD_OPT="$(DEB_BUILD_MAINT_OPTIONS="hardening=+all,-pie" DEB_LDFLAGS_MAINT_APPEND="-Wl,--as-needed -pie" dpkg-buildflags --get LDFLAGS)" \
    && CONFIGURE_ARGS="--prefix=/usr \
                --state=/var/lib/unit \
                --control=unix:/var/lib/unit/unit.sock \
                --pid=/var/lib/unit/unit.pid \
                --log=/var/lib/unit/unit.log \
                --user=peon \
                --group=peon" \
    && ./configure $CONFIGURE_ARGS --cc-opt="$CC_OPT" --ld-opt="$LD_OPT" --modules=/usr/lib/unit/debug-modules --debug \
    && make -j $NCPU unitd \
    && install -pm755 build/unitd /usr/sbin/unitd-debug \
    && make clean \
    && ./configure $CONFIGURE_ARGS --cc-opt="$CC_OPT" --ld-opt="$LD_OPT" --modules=/usr/lib/unit/modules \
    && make -j $NCPU unitd \
    && install -pm755 build/unitd /usr/sbin/unitd \
    && make clean \
    && ./configure $CONFIGURE_ARGS --cc-opt="$CC_OPT" --modules=/usr/lib/unit/debug-modules --debug \
    && ./configure php \
    && make -j $NCPU php-install \
    && make clean \
    && ./configure $CONFIGURE_ARGS --cc-opt="$CC_OPT" --modules=/usr/lib/unit/modules \
    && ./configure php \
    && make -j $NCPU php-install \
    && ldd /usr/sbin/unitd | awk '/=>/{print $(NF-1)}' | while read n; do dpkg-query -S $n; done | sed 's/^\([^:]\+\):.*$/\1/' | sort | uniq > /requirements.apt


FROM php:8.1-cli

ENV COMPOSER_MEMORY_LIMIT=-1 \
    COMPOSER_HOME="/.composer" \
    PHP_OPCACHE_VALIDATE_TIMESTAMPS=1 \
    PHP_OPCACHE_MAX_ACCELERATED_FILES=15000 \
    PHP_OPCACHE_MEMORY_CONSUMPTION=192 \
    PHP_OPCACHE_MAX_WASTED_PERCENTAGE=10

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
COPY wait-for-it.sh /usr/local/bin/wait-for-it
COPY php.ini /usr/local/etc/php/conf.d/99-php-overrides.ini

COPY --from=UNIT_BUILDER /usr/sbin/unitd /usr/sbin/unitd
COPY --from=UNIT_BUILDER /usr/sbin/unitd-debug /usr/sbin/unitd-debug
COPY --from=UNIT_BUILDER /usr/lib/unit/ /usr/lib/unit/
COPY --from=UNIT_BUILDER /requirements.apt /requirements.apt

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Very convenient PHP extensions installer: https://github.com/mlocati/docker-php-extension-installer
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

RUN chmod +x /usr/local/bin/wait-for-it \
    && chmod +x /usr/local/bin/docker-entrypoint.sh \
    && mkdir /docker-entrypoint.d/ \
    && groupadd -g 998 docker \
    && useradd -u 1000 peon \
    && usermod -aG docker peon \
    && mkdir /.composer \
    && chown peon /.composer \
    && mkdir /usr/tmp \
    && apt-get update && apt-get install -y \
        git \
        zip \
        ca-certificates \
        curl \
        gnupg \
        lsb-release \
    && install-php-extensions \
        intl \
        zip \
        uuid \
        pdo_pgsql \
        opcache \
        apcu \
        xdebug \
    && curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
    && echo \
       "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
       $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
    && mkdir -p /var/lib/unit \
    && chown peon -R /var/lib/unit \
    && chown peon -R /usr/tmp \
    && chown peon -R /usr/local/etc/php/conf.d  \
    && apt-get update && apt-get -y install \
      docker-ce-cli \
    && apt-get --no-install-recommends --no-install-suggests -y install curl $(cat /requirements.apt) \
    && apt-get clean && rm -rf /var/lib/apt/lists/* \
    && rm -f /requirements.apt \
    && ln -sf /dev/stdout /var/lib/unit/unit.log \
    && ln -sf /dev/stdout /var/lib/unit/access.log

USER peon

STOPSIGNAL SIGTERM

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

CMD ["unitd", "--no-daemon"]

