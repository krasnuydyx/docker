# Docker base image for magento developers
#### Centos7, php-7.0.2, php extentions for magento, mysql-community 5.7, apache 2.4, sshd
```
ssh user => centos
ssh pass => 123123q
```
[Docker Install Guide](https://docs.docker.com/engine/installation)
## Installation:
```
docker build -t magebox .
docker run -d magebox
# find new container id
docker ps
# get your container ip
DOCKERIP=`docker inspect {container_id} | grep IPA | cut -d "\"" -f 4`
# login into container
ssh -l centos $DOCKERIP
```
