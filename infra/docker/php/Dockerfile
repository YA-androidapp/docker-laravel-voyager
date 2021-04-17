FROM php:8.0-fpm-buster
LABEL maintainer="yaand"
SHELL ["/bin/bash", "-oeux", "pipefail", "-c"]

ARG DB_HOST
ARG DB_PORT
ARG DB_DATABASE
ARG DB_USERNAME
ARG DB_PASSWORD

RUN export DB_HOST=$DB_HOST
RUN export DB_PORT=$DB_PORT
RUN export DB_DATABASE=$DB_DATABASE
RUN export DB_USERNAME=$DB_USERNAME
RUN export DB_PASSWORD=$DB_PASSWORD

# timezone environment
ENV TZ=UTC \
  # locale
  LANG=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LC_ALL=en_US.UTF-8 \
  # composer environment
  COMPOSER_ALLOW_SUPERUSER=1 \
  COMPOSER_HOME=/composer

COPY --from=composer:2.0 /usr/bin/composer /usr/bin/composer

RUN apt-get update && \
  apt-get -y install git libicu-dev libonig-dev libzip-dev unzip locales && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* && \
  locale-gen en_US.UTF-8 && \
  localedef -f UTF-8 -i en_US en_US.UTF-8 && \
  mkdir /var/run/php-fpm && \
  docker-php-ext-install intl pdo_mysql zip bcmath && \
  composer config -g process-timeout 3600 && \
  composer config -g repos.packagist composer https://packagist.org

COPY ./infra/docker/php/php-fpm.d/zzz-www.conf /usr/local/etc/php-fpm.d/zzz-www.conf
COPY ./infra/docker/php/php.ini /usr/local/etc/php/php.ini

WORKDIR /work/backend

RUN composer create-project --prefer-dist laravel/laravel . && \
  php artisan key:generate && \
  php artisan storage:link && \
  chmod -R 777 storage bootstrap/cache

RUN php artisan migrate:fresh --seed

RUN composer require doctrine/dbal "^2" && \
  composer require --dev barryvdh/laravel-Debugbar && \
  php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider" && \
  composer require laravel/ui && \
  php artisan ui vue --auth && \
  sed -i 's/.sourceMaps()/.sourceMaps().vue({ version: 2 })/g' ./webpack.mix.js

# ダミーデータを追加する場合は --with-dummy を付けて php artisan voyager:install --with-dummy
RUN composer require tcg/voyager && \
  php artisan voyager:install
