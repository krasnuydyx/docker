# Build Mage dev machine
# Get latest centos image from hub
FROM centos:latest
MAINTAINER Andrii Danyliuk <krasnuy.dyx@gmail.com>
# Add epel, remi, mysql repos and activate them
RUN yum -y install epel-release yum-utils
RUN rpm -iU http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
RUN rpm -iU https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
RUN yum-config-manager --enable remi remi-php70
# Update system
RUN yum -y update
# Install sudo, git and mysql
RUN yum -y install sudo git mysql-community-server mysql-community-client
# Install php with extentions for magento (httpd installed as dependency, "exit 0" is workground for aufs problem)
RUN yum -y install php php-bcmath php-cli php-common php-gd php-intl php-json php-mbstring php-mcrypt php-mysqlnd php-opcache php-pdo php-pear php-pecl-xdebug php-precess php-soap php-xml phpunit ; exit 0
# Mods to avoid problems with perms in Mage dev env
RUN useradd -m -s /bin/bash -G wheel centos
RUN usermod -a -G apache centos
RUN usermod -a -G centos apache
RUN echo -e "123123q\n123123q" | passwd centos
RUN mkdir -p /var/www/html /var/log/httpd /var/www/html
RUN chown -R centos:apache /var/www
RUN chmod -R 775 /var/www
# Enable xdebug and set timezone
RUN echo -e 'zend_extension=xdebug.so\nxdebug.max_nesting_level=200\nxdebug.remote_enable=1\nxdebug.remote_connect_back=1' >> /etc/php.d/15-xdebug.ini
RUN echo 'date.timezone = America/Chicago' >> /etc/php.ini
RUN echo 'session.save_path = /tmp' >> /etc/php.ini
# Install sshd and supervisor
RUN yum -y install openssh-server supervisor rabbitmq-server
# Generate ssh host keys
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''
RUN ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key -N ''
RUN ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ''
# Configure rabbit
RUN /usr/sbin/rabbitmq-plugins enable rabbitmq_management
# mysql configuration
RUN sed -i "s/--initialize/--initialize-insecure/g" /usr/bin/mysqld_pre_systemd
RUN sed -i "s/--init-file=\"\$initfile\"//g" /usr/bin/mysqld_pre_systemd
RUN /usr/bin/mysqld_pre_systemd
# Create supervisor config
RUN sed -i "s|nodaemon=false|nodaemon=true|g" /etc/supervisord.conf
RUN echo -e '[program:sshd]\ncommand=/usr/sbin/sshd -D' >> /etc/supervisord.d/sshd.ini
RUN echo -e '[program:mysqld]\nexitcodes=0,1,127\nstartsecs=0\ncommand=/usr/sbin/mysqld --user=mysql' >> /etc/supervisord.d/mysqld.ini
RUN echo -e '[program:httpd]\ncommand=/usr/sbin/httpd -DFOREGROUND' >> /etc/supervisord.d/httpd.ini
RUN echo -e '[program:rabbit]\ncommand=/usr/sbin/rabbitmq-server\nautostart=true\nautorestart=unexpected\nstdout_logfile=/var/log/rabbit.log\nredirect_stderr=true\nexitcodes=0,1,127' >> /etc/supervisord.d/rabbit.ini
#RUN echo -e '[program:solr]\nautostart=true\nautorestart=unexpected\nstdout_logfile=/var/log/solr.log\nredirect_stderr=true\nexitcodes=0,1,127\nstartsecs=0\ncommand=/etc/init.d/solr start' >> /etc/supervisord.d/solr.ini
# Install composer
RUN curl -sS https://getcomposer.org/installer | php
RUN mv composer.phar /usr/local/bin/composer
# Clean yum cache
RUN yum clean all
# Run supervisord
CMD /usr/bin/supervisord
LABEL version="1.0"
EXPOSE 22 80 443 3389
ENV TESTENV=test
VOLUME /var/www/html
