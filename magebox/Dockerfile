FROM centos:latest
MAINTAINER Andrii Danyliuk <krasnuy.dyx@gmail.com>
ENV TZ=America/Los_Angeles
RUN rm /bin/sh; ln -s /bin/bash /bin/sh
# Add epel, remi, mysql repos, activate them; install mysql and base
RUN yum -q updateinfo; yum -y -q --setopt=tsflags=nodocs install epel-release yum-utils http://rpms.famillecollet.com/enterprise/remi-release-7.rpm \
 https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm; yum-config-manager --enable remi remi-php70 | grep "repo:\|enabled"; \
 yum -y -q --setopt=tsflags=nodocs update; yum -y -q --setopt=tsflags=nodocs install git mysql-community-server mysql-community-client sudo postfix\
 mailx wget which cronie openssh-server supervisor psmisc; yum -q clean all
# Install Apache and php with extentions for magento ("exit 0" is workground for aufs problem)
RUN yum -y -q --setopt=tsflags=nodocs install httpd mod_ssl php php-bcmath php-cli php-common php-gd php-intl php-json php-mbstring php-mcrypt \
 php-mysqlnd php-pdo php-pear php-pecl-xdebug php-process php-soap php-xml php-pecl-zip ; (exit 0) ; yum -q clean all
# Install phpunit and phpcpd
RUN wget -q -O /usr/bin/phpunit https://phar.phpunit.de/phpunit.phar; wget -q -O /usr/bin/phpcpd https://phar.phpunit.de/phpcpd.phar; \
 chmod +x /usr/bin/phpunit /usr/bin/phpcpd
# User for debug, apache preparations
RUN useradd -m -s /bin/bash -G wheel centos; usermod -a -G apache centos; usermod -a -G centos apache; \
 echo -e "123123q\n123123q" | passwd centos; mkdir -p /var/www/html /var/log/httpd /var/www/html; chown -R centos:apache /var/www; chmod -R 775 /var/www; \
 echo -e '<VirtualHost *:80>\n DocumentRoot /var/www/html\n ServerName default\n <Directory /var/www/html>\n  Options Indexes FollowSymLinks\n  AllowOverride All\n  Require all granted\n </Directory>\n</VirtualHost>' >> /etc/httpd/conf.d/mage.conf
# Enable xdebug, set timezone, set session.save_path and change memory limit
RUN echo -e 'zend_extension=xdebug.so\nxdebug.max_nesting_level=200\nxdebug.remote_enable=1\nxdebug.remote_connect_back=1' > /etc/php.d/15-xdebug.ini; \
 echo 'date.timezone = America/Los_Angeles' >> /etc/php.ini; echo 'session.save_path = /tmp' >> /etc/php.ini; \
 sed -i 's/memory_limit\ \=.*/memory_limit\ \=\ -1/g' /etc/php.ini
# Install rabbit 
RUN yum -y -q --setopt=tsflags=nodocs install rabbitmq-server ; /usr/sbin/rabbitmq-plugins enable rabbitmq_management ; yum -q clean all
# Generate ssh host keys
RUN ssh-keygen -q -t rsa -f /etc/ssh/ssh_host_rsa_key -N '' ; ssh-keygen -q -t dsa -f /etc/ssh/ssh_host_dsa_key -N '' ; \
 ssh-keygen -q -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ''
# mysql configuration
RUN sed -i '/\[mysqld\]/a explicit_defaults_for_timestamp=1' /etc/my.cnf ; sed -i '/\[mysqld\]/a performance_schema=OFF' /etc/my.cnf ; \
 sed -i '/\[mysqld\]/a collation-server=utf8_general_ci' /etc/my.cnf; sed -i '/\[mysqld\]/a init_connect="SET NAMES utf8"' /etc/my.cnf; \
 sed -i '/\[mysqld\]/a init_connect="SET collation_connection=utf8_general_ci"' /etc/my.cnf; sed -i '/\[mysqld\]/a character-set-server=utf8' /etc/my.cnf; \
 /usr/sbin/mysqld --user=mysql --initialize-insecure --explicit_defaults_for_timestamp
# Install mail
RUN /usr/libexec/postfix/aliasesdb; /usr/libexec/postfix/chroot-update; echo "* discard:" >> /etc/postfix/transport; postmap /etc/postfix/transport
# Create supervisor config
RUN sed -i "s|nodaemon=false|nodaemon=true|g" /etc/supervisord.conf; echo -e '[program:sshd]\ncommand=/usr/sbin/sshd -D' >> /etc/supervisord.d/sshd.ini; \
 echo -e '[program:postfix]\ncommand=/usr/sbin/postfix start' >> /etc/supervisord.d/postfix.ini; \
 echo -e '[program:mysqld]\ncommand=/usr/sbin/mysqld --user=mysql' >> /etc/supervisord.d/mysqld.ini; \
 echo -e '[program:httpd]\ncommand=/usr/sbin/httpd -DFOREGROUND' >> /etc/supervisord.d/httpd.ini; \
 echo -e '[program:rabbit]\ncommand=/usr/sbin/rabbitmq-server\nautostart=true\nautorestart=unexpected\nstdout_logfile=/var/log/rabbit.log\nredirect_stderr=true\nexitcodes=0,1,127' >> /etc/supervisord.d/rabbit.ini; \
 echo -e '[program:crond]\ncommand=/usr/sbin/crond -n' >> /etc/supervisord.d/crond.ini
# Install composer
RUN curl -sS https://getcomposer.org/installer | php; mv composer.phar /usr/bin/composer
# Clean yum cache
RUN yum -q clean all --enablerepo=*
CMD /usr/bin/supervisord
