FROM php:8.1-fpm

LABEL maintainer="ashrul"
WORKDIR /var/www

### ARG SECTION ###
ARG POSTGRES_VERSION=12
ARG NODE_VERSION=lts
ARG WKHTML_VERSION=0.12.6.1-2
ARG WKHTML_HOST_VERSION=bullseye_amd64

### ENV SECTION ###
ENV TZ=Asia/Kuala_Lumpur
ENV PHPFPM__access.format '"%R - %u [%t] \"%m %r\" %s %l %Q %f"'

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && \
    apt-get install -y \
    # php php-fpm \
    wget \
    nano \
    gnupg2 \
    git \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    zip \
    nano \
    systemctl \
    libfreetype6-dev \
    supervisor \
    gosu

## locales
RUN apt-get install -y \
    locales \
    && locale-gen \
    && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen

## Nginx 
RUN curl -sS https://nginx.org/keys/nginx_signing.key | apt-key add - \
    && echo "deb https://nginx.org/packages/mainline/debian/ bullseye nginx" | tee /etc/apt/sources.list.d/nginx.list \
    && apt-get update && apt-get install -y nginx 

## Node
RUN curl -fsSL https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && apt-get update && apt-get install -y nodejs

## SQLServer
# RUN apt-get install -y freetds-bin freetds-dev freetds-common
# RUN docker-php-ext-configure pdo_dblib --with-libdir=lib/x86_64-linux-gnu && \
#     docker-php-ext-install pdo_dblib

## PostgreSQL
# RUN apt install -y libpq-dev \ 
#     && echo "deb [signed-by=/usr/share/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt bullseye-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
#     && curl -sS https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | tee /usr/share/keyrings/pgdg.gpg >/dev/null \
#     && apt install -y postgresql-client-common
# RUN docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql
# RUN docker-php-ext-install pdo pdo_pgsql

## PHP EXT mysql
RUN docker-php-ext-configure pdo_mysql --with-pdo-mysql=mysqlnd \
    && docker-php-ext-configure mysqli --with-mysqli=mysqlnd \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install mysqli

# PHP EXT File exif
RUN docker-php-ext-install exif && docker-php-ext-enable exif

# PHP EXT Zip
RUN docker-php-ext-install zip

# PHP EXT GD
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install gd

# PHP EXT Redis
RUN pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis

# PHP EXT pcntl
RUN docker-php-ext-configure pcntl --enable-pcntl \
    && docker-php-ext-install \
    pcntl

# Install MongoDB Client
RUN pecl install mongodb && docker-php-ext-enable mongodb

# WKHTML
# RUN apt install -y libxrender1 fontconfig xfonts-75dpi xfonts-base xfonts-utils libfontenc1 x11-common xfonts-encodings
# RUN wget https://github.com/wkhtmltopdf/packaging/releases/download/$WKHTML_VERSION/wkhtmltox_$WKHTML_VERSION.$WKHTML_HOST_VERSION.deb 
# RUN dpkg -i wkhtmltox_$WKHTML_VERSION.$WKHTML_HOST_VERSION.deb
# RUN rm -rf wkhtmltox_$WKHTML_VERSION.$WKHTML_HOST_VERSION.deb

## Clean repo cache
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
RUN apt-get -y autoremove
RUN apt-get clean

COPY ./ /var/www/statsdigital
RUN mkdir /var/log/stacks

#FOR YII2 INIT
WORKDIR /var/wwwstatsdigital && php init

# for laravel only
# RUN chmod -R 775 storage bootstrap/cache
# RUN chown -R root:www-data storage bootstrap/cache
# RUN cp .env.production .env

# RUN composer install --prefer-dist --no-dev
# RUN composer install

# enable if your project have submodule or stacks is used as submodule
# RUN git submodule init && git submodule update

# for laravel only
# RUN composer require laravel/horizon league/flysystem-aws-s3-v3 "^3.0" && php artisan horizon:install
# RUN php artisan optimize \
    # disable migrate on testing with docker compose if you are using local databases
    # && php artisan migrate --force

RUN cp -r /var/www/statsdigital/setup/nginx/nginx.conf /etc/nginx/conf.d/default.conf
