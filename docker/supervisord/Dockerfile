FROM php:7.2-alpine

RUN apk --update add wget \
  curl \
  git \
  build-base \
  libmemcached-dev \
  libmcrypt-dev \
  libxml2-dev \
  zlib-dev \
  autoconf \
  cyrus-sasl-dev \
  libgsasl-dev \
  supervisor

RUN docker-php-ext-install mysqli mbstring pdo pdo_mysql tokenizer xml pcntl bcmath
RUN pecl channel-update pecl.php.net && pecl install memcached mcrypt-1.0.1 && docker-php-ext-enable memcached

RUN apk --update add postgresql-dev \
    && docker-php-ext-install pdo_pgsql

RUN rm /var/cache/apk/* \
    && mkdir -p /var/www

WORKDIR /etc/supervisor/conf.d/