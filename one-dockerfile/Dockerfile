FROM phusion/baseimage
MAINTAINER Dmitriy Savchenko <login.was.here@gmail.com>

ENV PHP_VERSION 5.6.13*
ENV NGINX_VERSION 1.8.0*
ENV DEBIAN_FRONTEND noninteractive
ENV MYSQL_PASSWORD="root"

RUN echo "mysql root password '$MYSQL_PASSWORD'"

RUN apt-get install -y software-properties-common
RUN add-apt-repository -y ppa:nginx/stable

RUN echo "deb http://ppa.launchpad.net/ondrej/php5-5.6/ubuntu trusty main" > /etc/apt/sources.list.d/ondrej-php5-5_6-trusty.list && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F4EA0AAE5267A6C

RUN apt-get update && \
    apt-get upgrade -y -o Dpkg::Options::="--force-confold" && \

# Generate password for MySQL

    apt-get install -y pwgen && \
    echo mysql-server mysql-server/root_password password $MYSQL_PASSWORD | debconf-set-selections && \
    echo mysql-server mysql-server/root_password_again password $MYSQL_PASSWORD | debconf-set-selections && \

# Install MySQL, Nginx, PHP (both fpm and CLI)

    apt-get install -y --no-install-recommends \
        mysql-client mysql-server \
        nginx \
        php5-cli=${PHP_VERSION} \
        php5-curl=${PHP_VERSION} \
        php5-fpm=${PHP_VERSION} \
        php5-mysql=${PHP_VERSION} \
        git \
        wget && \
    rm -rf /var/lib/apt/lists/*

# Install composer
RUN cd /tmp && \
    curl -sS https://getcomposer.org/installer | php && \
    mv -v composer.phar /usr/local/bin/composer && \
    composer self-update && \
    apt-get clean

# Download git master version of Bluz skeleton CMS

RUN cd /tmp && \
    wget https://github.com/bluzphp/skeleton/archive/master.tar.gz && \
    tar -xf master.tar.gz && \
    rm -v master.tar.gz && \
    cd skeleton-master && \

# Create directory for future website, move distributive there, remove unnecessary files

    mkdir /web && \
    mv /tmp/skeleton-master/* /web/ && \
    cd /web && \
    rm -rf /tmp/skeleton-master

# Run MySQL server, create database

RUN service mysql start && \
    mysql -uroot -p"$MYSQL_PASSWORD" -e 'CREATE DATABASE `bluz`;' && \

    mysql -uroot -p"$MYSQL_PASSWORD" bluz < /web/structure.ddl && \
    mysql -uroot -p"$MYSQL_PASSWORD" bluz < /web/dump.sql && \
    mysql -uroot -p"$MYSQL_PASSWORD" bluz < /web/tests/dump.sql

# Install CleverStyle CMS

RUN cd /web && \
    composer install

# Move to special directory for volume support on running

RUN mv /web /web_tmp

# Add Nginx config
RUN rm -rf /etc/nginx/sites-enabled/*
ADD bluz.conf /etc/nginx/sites-enabled/bluz

# Add services execution script
ADD start_webserver.sh /etc/my_init.d/start_webserver.sh

ADD db.php /web/application/configs/dev/

# Expose HTTP port.
EXPOSE 80

# Make startup script executable
RUN chmod +x /etc/my_init.d/start_webserver.sh

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
