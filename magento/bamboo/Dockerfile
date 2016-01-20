# Build Bamboo Mage machine
# Get latest magebox image
FROM magebox:latest
MAINTAINER Andrii Danyliuk <krasnuy.dyx@gmail.com>
# Clean yum cache
RUN yum clean all
# Run supervisord
CMD /usr/bin/supervisord
LABEL version="1.0"
EXPOSE 22 80 443 3306 5672
ENV TESTVAR=testvar
VOLUME /var/www/html
