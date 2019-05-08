FROM php:7-cli-alpine

COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN apk update && \
apk add --no-cache tzdata ca-certificates wget bash vim git libzip-dev zip curl openssh-client icu libpng freetype libjpeg-turbo postgresql-dev libffi-dev libsodium acl file gettext && \
apk add --no-cache --virtual build-dependencies icu-dev libxml2-dev freetype-dev libpng-dev libjpeg-turbo-dev g++ make autoconf libsodium-dev && \
rm -rf /var/cache/apk/*

RUN set -ex \
&& docker-php-source extract \
&& pecl install redis \
&& pecl clear-cache \
&& docker-php-ext-enable redis \
&& docker-php-source delete \
&& docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
&& docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
&& docker-php-ext-configure zip --with-libzip \
&& docker-php-ext-configure intl \
&& docker-php-ext-configure soap --enable-soap \
&& docker-php-ext-install -j$(nproc) pgsql pdo_pgsql intl zip gd soap \
&& apk del build-dependencies \
&& rm -rf /tmp/*v
