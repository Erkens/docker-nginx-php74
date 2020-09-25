FROM composer:1.9.1 AS composer
FROM php:7.4.2-fpm-alpine3.11  AS php

FROM php AS php-build
RUN apk add --update --no-cache --virtual .phpize-deps-configure $PHPIZE_DEPS && \
    docker-php-source extract && \
    yes | pecl install xdebug && \
    yes | pecl install apcu && \
    docker-php-source delete && \
    apk del --no-network .phpize-deps-configure && \
    docker-php-ext-enable apcu && \
    apk add --update --no-cache icu-dev postgresql-dev && \
    docker-php-ext-install intl  && \
    docker-php-ext-install opcache && \
    docker-php-ext-install pdo_pgsql && \
    apk del --no-network icu-dev postgresql-dev && \
    apk add --update --no-cache --virtual .strip-binaries binutils && \
    strip /usr/local/lib/php/extensions/no-debug-non-zts-20190902/* && \
    apk del --no-network .strip-binaries

FROM php AS user-php
ARG TIMEZONE=Europe/Amsterdam
ARG USER_ID=1000
ARG GROUP_ID=1000

COPY --from=composer /usr/bin/composer /usr/local/bin/composer
ENV COMPOSER_MEMORY_LIMIT -1

RUN apk add --update --no-cache \
        icu-libs \
        git \
        libpq \
        tzdata \
        wget \
        openssh-client \
    && \
    cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && \
    echo ${TIMEZONE} > /etc/timezone && \
    apk del tzdata && \
    addgroup -g ${GROUP_ID} user && \
    mkdir -p /data /home/user && \
    adduser -D -u ${USER_ID} -h /home/user -G user user && \
    install -d -m 0755 -o user -g user /home/user && \
    chown -R user:user /data /home/user

COPY --from=php-build /usr/local/lib/php/extensions/no-debug-non-zts-20190902/* /usr/local/lib/php/extensions/no-debug-non-zts-20190902/
COPY --from=php-build /usr/local/etc/php/conf.d/* /usr/local/etc/php/conf.d/

COPY php.ini /usr/local/etc/php/
COPY xdebug.ini /usr/local/etc/php/conf.d/
COPY php-fpm.conf /usr/local/etc/

USER user:user

WORKDIR /data
