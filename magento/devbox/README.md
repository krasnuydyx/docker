# Docker base image for magento developers
# Centos7, php-7.0.2, php extentions for magento, mysql-community 5.7, apache 2.4, ssh
# ssh user centos, password 123123q
Installation:
docker build -t magebox .
docker run -d magebox
docker ps # find new container id
DOCKERIP=`docker inspect c6d35b5a6984 | grep IPA | cut -d "\"" -f 4`
ssh -l centos $DOCKERIP
