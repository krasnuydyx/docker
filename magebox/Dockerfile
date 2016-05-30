FROM centos:latest
MAINTAINER Andrii Danyliuk <krasnuy.dyx@gmail.com>
# Replacing sh with bash
RUN rm /bin/sh && ln -s /bin/bash /bin/sh
# Add epel, remi, mysql repos and activate them
RUN yum -y -q install epel-release yum-utils && rpm --quiet -iU http://rpms.famillecollet.com/enterprise/remi-release-7.rpm; \
 rpm --quiet -iU https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm; \
 yum-config-manager --enable remi remi-php70 | grep "repo:\|enabled"; \
 yum-config-manager --disable mysql57-community | grep "repo:\|disabled"; \
 yum-config-manager --enable mysql56-community | grep "repo:\|enabled"
# Update system and install base packages
RUN yum -y -q update; yum -y -q install git mysql-community-server mysql-community-client sudo postfix mailx wget cronie openssh-server supervisor
# Install Apache and php with extentions for magento ("exit 0" is workground for aufs problem)
RUN yum -y -q install httpd mod_ssl php php-bcmath php-cli php-common php-gd php-intl php-json php-mbstring php-mcrypt \
 php-mysqlnd php-pdo php-pear php-pecl-xdebug php-precess php-soap php-xml php-pecl-zip ; exit 0
# Install phpunit and phpcpd
RUN wget -q https://phar.phpunit.de/phpunit.phar && wget -q https://phar.phpunit.de/phpcpd.phar && chmod +x php* ; \
 cp phpunit.phar /usr/bin/phpunit; cp phpcpd.phar /usr/bin/phpcpd; rm -rf php*
# User for debug, apache preparations
RUN useradd -m -s /bin/bash -G wheel centos; usermod -a -G apache centos; usermod -a -G centos apache; \
 echo -e "123123q\n123123q" | passwd centos; mkdir -p /var/www/html /var/log/httpd /var/www/html; chown -R centos:apache /var/www; chmod -R 775 /var/www; \
 echo -e '<VirtualHost *:80>\n DocumentRoot /var/www/html\n ServerName default\n <Directory /var/www/html>\n  Options Indexes FollowSymLinks\n  AllowOverride All\n  Require all granted\n </Directory>\n</VirtualHost>' >> /etc/httpd/conf.d/mage.conf
# Enable xdebug, set timezone, set session.save_path and change memory limit
RUN echo -e 'zend_extension=xdebug.so\nxdebug.max_nesting_level=200\nxdebug.remote_enable=1\nxdebug.remote_connect_back=1' > /etc/php.d/15-xdebug.ini; \
 echo 'date.timezone = America/Los_Angeles' >> /etc/php.ini && echo 'session.save_path = /tmp' >> /etc/php.ini; \
 sed -i 's/memory_limit\ \=.*/memory_limit\ \=\ -1/g' /etc/php.ini
# Install supervisor, mods
RUN yum -y -q install rabbitmq-server ; /usr/sbin/rabbitmq-plugins enable rabbitmq_management
# Generate ssh host keys
RUN ssh-keygen -q -t rsa -f /etc/ssh/ssh_host_rsa_key -N '' ; ssh-keygen -q -t dsa -f /etc/ssh/ssh_host_dsa_key -N '' ; \
 ssh-keygen -q -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ''
# mysql configuration
#mysql57#RUN sed -i "s/--initialize/--initialize-insecure/g" /usr/bin/mysqld_pre_systemd ; \
#sed -i "s/--init-file=\"\$initfile\"//g" /usr/bin/mysqld_pre_systemd && /usr/bin/mysqld_pre_systemd
RUN /usr/bin/mysql-systemd-start pre
# Install mail
RUN /usr/libexec/postfix/aliasesdb && /usr/libexec/postfix/chroot-update && echo "* discard:" >> /etc/postfix/transport && postmap /etc/postfix/transport
# Install elastic
RUN echo -e '[el]\nname=Elasticsearch repository\nbaseurl=https://packages.elastic.co/elasticsearch/2.x/centos\ngpgcheck=1\ngpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch\nenabled=1' >> /etc/yum.repos.d/el.repo; \
 rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch ; yum -y -q install elasticsearch
# Create supervisor config
RUN sed -i "s|nodaemon=false|nodaemon=true|g" /etc/supervisord.conf; echo -e '[program:sshd]\ncommand=/usr/sbin/sshd -D' >> /etc/supervisord.d/sshd.ini; \
 echo -e '[program:postfix]\ncommand=/usr/sbin/postfix start' >> /etc/supervisord.d/postfix.ini; \
 echo -e '[program:mysqld]\nexitcodes=0,1,127\nstartsecs=0\ncommand=/usr/sbin/mysqld --user=mysql' >> /etc/supervisord.d/mysqld.ini; \
 echo -e '[program:httpd]\ncommand=/usr/sbin/httpd -DFOREGROUND' >> /etc/supervisord.d/httpd.ini; \
 echo -e '[program:rabbit]\ncommand=/usr/sbin/rabbitmq-server\nautostart=true\nautorestart=unexpected\nstdout_logfile=/var/log/rabbit.log\nredirect_stderr=true\nexitcodes=0,1,127' >> /etc/supervisord.d/rabbit.ini; \
 echo -e '[program:el]\ncommand=/usr/bin/sudo -u elasticsearch /usr/share/elasticsearch/bin/elasticsearch -Des.default.path.home=/usr/share/elasticsearch/ -Des.default.path.data=/var/lib/elasticsearch/ -Des.default.path.conf=/etc/elasticsearch/ -Des.default.path.logs=/var/log/elasticsearch/\nautostart=true\nautorestart=unexpected\nstdout_logfile=/var/log/el.log\nredirect_stderr=true\nexitcodes=0,1,127' >> /etc/supervisord.d/el.ini; \
 echo -e '[program:crond]\ncommand=/usr/sbin/crond' >> /etc/supervisord.d/crond.ini; \
 echo -e '[program:solr]\nautostart=true\nautorestart=unexpected\nstdout_logfile=/var/log/solr.log\nredirect_stderr=true\nexitcodes=0,1,127\nstartsecs=0\ncommand=/etc/init.d/solr start' >> /etc/supervisord.d/solr.ini
# Install composer
RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/bin/composer
# Clean yum cache
RUN yum clean all
CMD /usr/bin/supervisord
