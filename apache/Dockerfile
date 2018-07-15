FROM debian:stretch
MAINTAINER asavoie
LABEL php.version=7.2

RUN apt-get update && \
    apt-get install -y \
    apt-transport-https \
    ca-certificates \
    wget \
    curl \
    sudo \
    locales \
    mysql-client \
    dialog \
    apt-utils \
    vim

# Apache
# -----------------------------------------------------------------------------
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y apache2

# Adding packages.sury.org repository for PHP 7.2
RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg && \
    echo "deb https://packages.sury.org/php/ stretch main" > /etc/apt/sources.list.d/php.list


# PHP packages
# -----------------------------------------------------------------------------
RUN apt-get update && \
    apt-get install -y \
    php7.2 \
    php7.2-readline \
    php7.2-cli \
    php7.2-mysql \
    php7.2-json \
    php7.2-xsl \
    php7.2-intl \
    php7.2-curl \
    php7.2-gd \
    php7.2-xmlrpc \
    php7.2-memcached \
    php7.2-mbstring \
    php7.2-zip\
    git \
    unzip


# Local user
# => to try : run container with --user option to set root user id with host user id
# -----------------------------------------------------------------------------
# nb: the 1013 used here for user id and group id is later on replaced by the code in bootstrap.sh...
RUN addgroup --gid 1013 site && \
    adduser --system --uid=1013 --gid=1013 \
        --home /home/site --shell /bin/bash site && \
    adduser site site && \
    adduser site www-data && \
    cp /etc/skel/.* /home/site/; \
    chown -R site:site /home/site;
#
#
## Apache config
## -----------------------------------------------------------------------------
#ENV APACHE_RUN_USER site
#ENV APACHE_RUN_GROUP site
#ENV APACHE_LOG_DIR /var/log/apache2
#ENV APACHE_PID_FILE /var/run/apache2.pid
#ENV APACHE_RUN_DIR  /var/run/apache2
#ENV APACHE_LOCK_DIR /var/lock/apache2
#
RUN sed -rie 's|export APACHE_RUN_USER=.*|export APACHE_RUN_USER=site|g' /etc/apache2/envvars && \
    sed -rie 's|export APACHE_RUN_GROUP=.*|export APACHE_RUN_GROUP=site|g' /etc/apache2/envvars && \
    sed -rie 's|export APACHE_LOG_DIR=.*|export APACHE_LOG_DIR=/var/log/apache2|g' /etc/apache2/envvars && \
    echo "ServerName localhost" >> /etc/apache2/apache2.conf
#
#RUN rm /etc/apache2/ports.conf && \
#    echo "# The single ports for http listening are defined in each vhost config file \
#          Listen 80 \
#          Listen 82 \
#          <IfModule ssl_module>  \
#          	Listen 443  \
#          </IfModule> \
#          <IfModule mod_gnutls.c> \
#          	Listen 443 \
#          </IfModule>" > /etc/apache2/ports.conf
#
#RUN a2enmod rewrite headers vhost_alias proxy_http proxy ssl info status


## vhost config
## -----------------------------------------------------------------------------
## default & controlpanel
#RUN curl -L -o /etc/apache2/sites-available/099-default.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/099-default.conf && \
#    curl -L -o /tmp/010-controlpanel.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/010-controlpanel.conf && \
#    cat /tmp/010-controlpanel.conf >> /etc/apache2/apache2.conf && \
#    rm /tmp/010-controlpanel.conf
#
#RUN curl -L -o /tmp/controlpanel.tar.gz https://github.com/kaliop/docker-images/raw/master/build_files/controlpanel.tar.gz && \
#    tar -zxf /tmp/controlpanel.tar.gz -C /home/site/ && \
#    rm /tmp/controlpanel.tar.gz
#
#RUN a2ensite 099-default.conf
#
## dynamic vhosts
## a2ensite is run in bootstrap.sh for these as they could also be mounted as volume at run time
#RUN curl -L -o /etc/apache2/sites-available/001-dynamic-vhost-ez5.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/001-dynamic-vhost-ez5.conf && \
#    curl -L -o /etc/apache2/sites-available/002-dynamic-vhost-ezplatform.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/002-dynamic-vhost-ezplatform.conf && \
#    curl -L -o /etc/apache2/sites-available/003-dynamic-vhost-ez4.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/003-dynamic-vhost-ez4.conf && \
#    curl -L -o /etc/apache2/sites-available/005-dynamic-vhost-wp.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/005-dynamic-vhost-wp.conf && \
#    curl -L -o /etc/apache2/sites-available/098-dynamic-vhost.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/098-dynamic-vhost.conf && \
#    curl -L -o /etc/apache2/sites-available/ez5-common.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/ez5-common.conf && \
#    curl -L -o /etc/apache2/sites-available/ezplatform-common.conf https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/vhosts/ezplatform-common.conf

COPY vhost /etc/apache2/sites-available/000-default.conf

# Clear archives in apt cache folder
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Get bootstrap script
COPY bootstrap.sh /root/bootstrap.sh
#RUN curl -o /root/bootstrap.sh https://raw.githubusercontent.com/kaliop/docker-images/master/build_files/apache/apache_bootstrap.sh && \
RUN chmod 755 /root/bootstrap.sh

COPY .bashrc /home/site/.bashrc
EXPOSE 80
EXPOSE 82

WORKDIR /var/www/

CMD ["/root/bootstrap.sh"]