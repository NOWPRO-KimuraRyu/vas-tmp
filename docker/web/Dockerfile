FROM php:8.4-apache AS setup-stage

RUN apt-get update
RUN docker-php-ext-install pdo_mysql opcache

RUN apt-get install -y libzip-dev zlib1g-dev \
    && docker-php-ext-install zip

RUN apt-get install -y  \
    gnupg2 \
    libfreetype6-dev \
    libicu-dev \
    libjpeg-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libpq-dev \
    libzip-dev \
    unzip \
    vim \
    jq \
    sudo \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
    && docker-php-ext-install gd

RUN curl -o /tmp/composer-setup.php -L https://getcomposer.org/installer \
    && php /tmp/composer-setup.php --install-dir=/usr/bin --filename=composer \
    && rm /tmp/composer-setup.php

RUN apt-get install -y locales \
    && locale-gen ja_JP.UTF-8 \
    && localedef -f UTF-8 -i ja_JP ja_JP.UTF-8

# Node.js 20.xをインストールするためにNodeSourceリポジトリを設定
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
RUN apt-get install -y nodejs

RUN pecl install xdebug-3.4.2 \
    && docker-php-ext-enable xdebug

RUN apt-get -y install default-mysql-client
RUN a2enmod rewrite

FROM setup-stage AS build-stage
ARG UID=1000
ARG GID=1000
ARG PROJECT_ROOT=/app
ARG APP_USER=app_user

RUN groupadd -g ${GID} ${APP_USER} \
    && useradd -r -u ${UID} -g ${APP_USER} ${APP_USER}

USER root
RUN mkdir -p /home/${APP_USER}/.npm
RUN chown -R ${UID}:${GID} /home/${APP_USER}/.npm

# パスワードなしでsudoを実行できるように設定
RUN echo "${APP_USER} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/${APP_USER} \
    && chmod 0440 /etc/sudoers.d/${APP_USER}

# ServerTokens Prod を apache2.conf に設定するコマンドを追加
RUN echo "ServerTokens Prod" >> /etc/apache2/apache2.conf

USER ${APP_USER}
WORKDIR ${PROJECT_ROOT}

ENV HOME ${PROJECT_ROOT}

COPY --chown=${UID}:${GID} composer.json composer.lock ${PROJECT_ROOT}/
COPY --chown=${UID}:${GID} package.json package-lock.json ${PROJECT_ROOT}/
COPY --chown=${UID}:${GID} . ${PROJECT_ROOT}/

COPY ./docker/web/etc/apache2/envvars /etc/apache2/envvars
COPY ./docker/web/etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf
COPY ./docker/web/usr/local/etc/php/conf.d/docker-php-ext-opcache.ini /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini
COPY ./docker/web/usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
COPY ./docker/web/usr/local/etc/php/conf.d/docker-php-ext-memory-limit.ini /usr/local/etc/php/conf.d/docker-php-ext-memory-limit.ini

COPY --chown=${UID}:${GID} . ${PROJECT_ROOT}

EXPOSE 80/tcp
CMD ["apache2-foreground"]
