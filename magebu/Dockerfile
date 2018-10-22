FROM ubuntu:latest
# Don't prompt for configuration settings when using apt-get
ENV DEBIAN_FRONTEND=noninteractive
# Install distro Apache, JDK 8, PHP 7.2, etc
RUN apt-get -qq update; apt-get install -qq apt-utils; apt-get install -qq software-properties-common python3-software-properties > /dev/null; \
 add-apt-repository -y -u ppa:ondrej/php; apt-get -qq update; apt-get -qq full-upgrade > /dev/null; apt-get install -qq acl sudo apt-utils apache2 \
 ssh dialog debconf-utils libapache2-mod-php wget vim libmcrypt-dev php7.2 php7.2-curl php7.2-gd php7.2-mysql php7.2-intl curl git \
 php7.2-intl php7.2-opcache php7.2-zip zip unzip php7.2-xml php7.2-bcmath php7.2-mbstring php7.2-xsl php7.2-soap php7.2-xdebug > /dev/null
# Install MySQL for Xenial (v5.7)
RUN rm /bin/sh; ln -s /bin/bash /bin/sh; echo "mysql-server-5.7 mysql-server/root_password password" | debconf-set-selections ; \
 echo "mysql-server-5.7 mysql-server/root_password_again password" | debconf-set-selections ; apt-get -qq install mysql-server mysql-client > /dev/null
# Set timezone, set session.save_path, and change memory limit for both CLI and Apache PHP 7.2
RUN echo 'always_populate_raw_post_data = -1' >> /etc/php/7.2/cli/php.ini; echo 'date.timezone = America/Los_Angeles' >> /etc/php/7.2/cli/php.ini; echo 'session.save_path = /tmp' >> /etc/php/7.2/cli/php.ini; sed -i 's/memory_limit\ \=.*/memory_limit\ \=\ -1/g' /etc/php/7.2/cli/php.ini
RUN echo 'always_populate_raw_post_data = -1' >> /etc/php/7.2/apache2/php.ini; echo 'date.timezone = America/Los_Angeles' >> /etc/php/7.2/apache2/php.ini; echo 'session.save_path = /tmp' >> /etc/php/7.2/apache2/php.ini; sed -i 's/memory_limit\ \=.*/memory_limit\ \=\ -1/g' /etc/php/7.2/apache2/php.ini
# Install RabbitMQ, phpunit, phpcpd, mcrypt via pecl
RUN apt-get -qq install rabbitmq-server phpunit phpcpd php-pear php7.2-dev > /dev/null; pecl channel-update pecl.php.net > /dev/null; pecl install mcrypt-1.0.1 > /dev/null
# Configure RabbitMQ - must set NODENAME
RUN echo 'NODENAME=rabbit@localhost' > /etc/rabbitmq/rabbitmq-env.conf; /usr/sbin/rabbitmq-plugins enable rabbitmq_management > /dev/null; echo -e 'rabbitmq soft nofile 16384\nrabbitmq hard nofile 16384' >> /etc/security/limits.conf; echo '/usr/sbin/rabbitmq-server &' >> /tmp/rabbit.sh; echo 'sleep 10s' >> /tmp/rabbit.sh; echo 'rabbitmqctl add_user admin admin' >> /tmp/rabbit.sh; echo 'rabbitmqctl set_user_tags admin administrator' >> /tmp/rabbit.sh; echo 'rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"' >> /tmp/rabbit.sh; echo 'rabbitmqctl list_users' >> /tmp/rabbit.sh
RUN bash /tmp/rabbit.sh > /dev/null; rm -f /tmp/rabbit.sh
# MySQL configuration
RUN echo -e "[mysqld]\nuser=mysql\npid-file=/var/run/mysqld/mysqld.pid\nsocket=/var/run/mysqld/mysqld.sock\nport=3306\nbasedir=/usr\ndatadir=/var/lib/mysql\ntmpdir=/tmp\nlc-messages-dir=/usr/share/mysql\nskip-external-locking\nlog_error=/var/log/mysql/error.log\nopen_files_limit=16384\ntable_open_cache=4000\ncharacter-set-server=utf8\ninnodb_buffer_pool_size=256M\ninnodb_flush_log_at_trx_commit=0\nmax_allowed_packet=512M\nquery_cache_size=16M\nquery_cache_type=1\nperformance_schema=0\ncollation-server=utf8_general_ci\ninit_connect='SET collation_connection = utf8_general_ci\ninit_connect='SET NAMES utf8" > /etc/mysql/mysql.conf.d/mysqld.cnf
RUN echo "mysql soft nofile 16384" >> /etc/security/limits.conf; echo "mysql hard nofile 16384" >> /etc/security/limits.conf; echo "mysql soft nproc 4096" >> /etc/security/limits.conf; echo "mysql hard nproc 4096" >> /etc/security/limits.conf; usermod -d /var/lib/mysql mysql
# Resolve issue with password syntax
RUN sed -i 's/check_root_accounts/#&/g' /etc/mysql/debian-start
# Add magento user with sudo access
RUN useradd magento -s /bin/bash -m -d /home/magento; echo "magento ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/magento; echo -e "123123q\n123123q" | passwd magento
# Install composer
RUN curl -sS https://getcomposer.org/installer | php > /dev/null; mv composer.phar /usr/bin/composer; ln -s /usr/bin/composer /usr/local/bin/composer
RUN a2enmod ssl rewrite; sed -i 's/APACHE_RUN_GROUP=www-data/APACHE_RUN_GROUP=magento/g' /etc/apache2/envvars
RUN echo 'ServerName magebu' >> /etc/apache2/apache2.conf
RUN usermod -a -G magento www-data; chown -R www-data:magento /var/lib/php/*
# Add xvfb, elasticsearch, postfix, timezone
RUN apt-get -qq install xvfb lsof bzip2 > /dev/null ; \
 ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
ADD scripts/runserver /usr/local/bin/runserver
# Entry point into container
ENTRYPOINT ["bash", "-c"]
CMD ["/usr/local/bin/runserver"]
