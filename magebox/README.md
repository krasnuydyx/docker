# Magento docker base image on centos for developers
#### Software (Versions may be higher)
| Name | Version |
| ----- | ----- |
| CentOS | 7.3.1611 |
| Apache | 2.4.6 |
| Mysql | 5.7 |
| PHP | 7.0.17 |
| RabbitMQ | 3.3.5 |
| OpenSSH | 6.6.1 |
| Composer | 1.4.1 |

### Requirements
[Docker Install Guide](https://docs.docker.com/engine/installation)
### Installation:
  1. Download Dockerfile and build image in same dir
  
    ```
    docker build -t magebox .
    ```
  1. Run container from newly build image
  
    ```
    docker run -tdi magebox
    ```
  1. Find container id
  
    ```
    docker ps
    ```
  1. Get your container ip
  
    ```
    DOCKERIP=`docker inspect {container_id} | grep IPA | cut -d "\"" -f 4`
    ```
  1. Login into container using ssh
  
    ```
    ssh -l centos $DOCKERIP
    ```
### Default credentials
  Container access
  
  ```
  username => centos
  password => 123123q
  ```
  Mysql access
  
  ```
  username => root
  password => 
  ```
