---
title: 'Docker Development Environment'
date: 2022-01-29 00:00:00
description: This page is a demo that shows everything you can do inside portfolio and blog posts.
featured_image: '/images/demo/demo-square.jpg'
---
Docker run
Select an image https://hub.docker.com/search?type=image
Docker run centos
This makes it download and execute the centos machine, then it shuts down because we didn’t tell it to do anything.
To run bash and automatically log into centos add the -it
root@localhost:~/docker# docker run -it centos bash
[root@421b154a1d06 /]#
[root@421b154a1d06 /]# cat /etc/*release*
CentOS Linux release 8.4.2105
Derived from Red Hat Enterprise Linux 8.4
NAME="CentOS Linux"
VERSION="8"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="CentOS Linux 8"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:8"
HOME_URL="https://centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"
CENTOS_MANTISBT_PROJECT="CentOS-8"
CENTOS_MANTISBT_PROJECT_VERSION="8"
CentOS Linux release 8.4.2105
CentOS Linux release 8.4.2105
cpe:/o:centos:centos:8
exit

docker ps = list all running containers
docker run -d centos sleep 20 = sleep centos for 20 seconds then exit and -d to run in background
root@localhost:~/docker# docker run -d centos sleep 20
9284cadef63c416e96217825af0d6b9c37590fb47b6313131b2edbf614919566
root@localhost:~/docker# docker ps
CONTAINER ID   IMAGE     COMMAND      CREATED         STATUS         PORTS     NAMES
9284cadef63c   centos    "sleep 20"   4 seconds ago   Up 3 seconds             suspicious_rubin
Docker ps -a = to see all the containers you ran in the past
docker run -d centos sleep 2000 = sleep centos for 2000 seconds then exit
to stop before that time run 
docker stop “name” or “id”
docker ps
docker ps -a
docker rm “id” or “name”= to clean up and remove containers and re-claim disk space
you can also provide the first 3 letters if there unique
docker rm 878 998 765
docker ps -a
docker images =  to see your images you have on your localmachine
docker rmi centos= to remove the image, if a container is using it, you have to remove the container first.
Docker ps
docker pull ubuntu = download image without running it
docker run -d ubuntu sleep 100
docker ps
docker exec “id” cat /etc/*release*
