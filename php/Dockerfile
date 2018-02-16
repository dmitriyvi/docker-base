FROM attractgrouphub/jtracker-php-base:7.1.1-mysql-client

#####################################
# php config
#####################################
#COPY ./php/php.ini /usr/local/etc/php/
COPY ./php/php.ini /usr/local/etc/php/conf.d
COPY ./php/php.pool.conf /usr/local/etc/php-fpm.d/www.conf

RUN touch /var/log/php-errors.log
RUN chmod 777 /var/log/php-errors.log

COPY ./project /var/www/project
RUN ln -sv ./project /var/www/project

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        libmemcached-dev \
        libz-dev \
        libjpeg-dev \
        libpng12-dev \
        libfreetype6-dev \
        libssl-dev \
        libmcrypt-dev \
        git \
        mysql-client

RUN apt-get purge --auto-remove -y zlib1g-dev \
        && apt-get -y install \
            libssl-dev \
            libc-client2007e-dev \
            libkrb5-dev \
            libfreetype6-dev \
            libjpeg62-turbo-dev \
            libmcrypt-dev \
            libpng12-dev \
        && docker-php-ext-configure imap --with-imap-ssl --with-kerberos \
        && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
        && docker-php-ext-install imap \
        && docker-php-ext-install -j$(nproc) gd \
        && docker-php-ext-install opcache \
        && docker-php-ext-install pdo_mysql \
        && docker-php-ext-install mysqli \
        && docker-php-ext-install pdo \
        && docker-php-ext-install mcrypt

#####################################
# OpCahce
#####################################
ARG INSTALL_OPCACHE=true
RUN if [ ${INSTALL_OPCACHE} != true ]; then { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=0'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
     } > /usr/local/etc/php/conf.d/opcache-recommended.ini; \
fi

# Copy ssl cert
#COPY ./ssl /etc/ssl/crm.movewheels.com/

#####################################
# xDebug:
#####################################

ARG INSTALL_XDEBUG=false
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Install the xdebug extension
    pecl install xdebug && \
    docker-php-ext-enable xdebug \
;fi

# allow root for php-fpm
ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=root --with-fpm-group=root

# Copy xdebug configration for remote debugging
#COPY ./php/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

WORKDIR /var/www/project

# Copy env file
#ARG PROJECT_ENV=local
#
#ENV P_ENV .env.${PROJECT_ENV}
#RUN if [ ! -f "/var/www/project/.env" ]; then \
#    cp /var/www/project/${P_ENV} /var/www/project/.env \
#;fi

# Use the crontab file.
#ARG USE_CRONTAB=false
#RUN if [ ${USE_CRONTAB} = true ]; then \
#    apt-get install -y --no-install-recommends cron && \
#    crontab /var/www/project/cron.conf \
#;fi


CMD ["php-fpm", "-R"]