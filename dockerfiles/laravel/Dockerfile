####################################
# A Dockerfile of many sources...
####################################

FROM phusion/baseimage

# Fixes some weird terminal issues such as broken clear / CTRL+L
ENV TERM=linux

# Install Ondrej repos for Ubuntu Xenial, PHP7.1, composer and selected extensions
RUN echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu xenial main" > /etc/apt/sources.list.d/ondrej-php.list \
    && echo "deb http://ppa.launchpad.net/ondrej/php-qa/ubuntu xenial main" > /etc/apt/sources.list.d/ondrej-php-qa.list \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F4EA0AAE5267A6C \
    && apt-get update \
    && apt-get -y --no-install-recommends install \
        php7.1-cli \
        php7.1-curl \
        php7.1-fpm \
        php7.1-json \
        php7.1-mbstring \
        php7.1-mysql \
        php7.1-opcache \
        php7.1-readline \
        php7.1-xml \
        php7.1-zip \
        curl \
        ca-certificates \
        php-apcu-bc \
        php-apcu \
        php-libsodium \
        php-redis \
        php-xdebug \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/* ~/.composer

# Configure FPM to run properly on docker
RUN sed -i "/listen = .*/c\listen = [::]:9000" /etc/php/7.1/fpm/pool.d/www.conf \
    && sed -i "/;access.log = .*/c\access.log = /proc/self/fd/2" /etc/php/7.1/fpm/pool.d/www.conf \
    && sed -i "/;clear_env = .*/c\clear_env = no" /etc/php/7.1/fpm/pool.d/www.conf \
    && sed -i "/;catch_workers_output = .*/c\catch_workers_output = yes" /etc/php/7.1/fpm/pool.d/www.conf \
    && sed -i "/pid = .*/c\;pid = /run/php/php7.1-fpm.pid" /etc/php/7.1/fpm/php-fpm.conf \
    && sed -i "/;daemonize = .*/c\daemonize = no" /etc/php/7.1/fpm/php-fpm.conf \
    && sed -i "/error_log = .*/c\error_log = /proc/self/fd/2" /etc/php/7.1/fpm/php-fpm.conf \
    && usermod -u 1000 www-data

# Open up fcgi port
EXPOSE 9000

# Add our PHP-FPM service
RUN mkdir /etc/service/php-fpm
ADD dockerfiles/laravel/php-fpm.sh /etc/service/php-fpm/run
RUN chmod a+x /etc/service/php-fpm/run

# Add our Laravel queue processor service
RUN mkdir /etc/service/artisan-queue
ADD dockerfiles/laravel/artisan-queue.sh /etc/service/artisan-queue/run
RUN chmod a+x /etc/service/artisan-queue/run

# Add our Laravel scheduled task processing job
RUN echo "* * * * * www-data php /app/artisan schedule:run >/dev/null 2>&1\n" >> /etc/cron.d/artisan-schedule

CMD ["/sbin/my_init"]
