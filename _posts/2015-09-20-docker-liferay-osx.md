---
title: "Setup a docker environment for Liferay with MySQL on OSX"
description: "How to setup a docker environment for Liferay with MySQL on OSX"
layout: post
comments: true
date: 2015-09-20
categories:
permalink: "docker-liferay-osx"
tags: [docker, liferay, osx]
---
##Install docker on OSX
To install:

* Download docker [This one](http://www.youtube.com/watch?v=V1vQf4qyMXg)
* [The second presentation](http://www.youtube.com/watch?v=RR1E5zO-eBo) is about GWT or "gwit" as they call it at Google. Very interactive session where Josh responds to the host and to attendees.
* [The third one](http://www.youtube.com/watch?v=aAb7hSCtvGw) gives a lot of precious advice on good API design.

##Use docker-machine
{% highlight bash%}
> docker-machine ls

NAME      ACTIVE   DRIVER       STATE     URL   SWARM
default            virtualbox   Stopped         

> docker-machine start default
Starting VM...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

> docker-machine env default
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/jihedamine/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"
# Run this command to configure your shell: 
# eval "$(docker-machine env default)"

> docker-machine ssh default

                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/
 _                 _   ____     _            _
| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
Boot2Docker version 1.8.2, build master : aba6192 - Thu Sep 10 20:58:17 UTC 2015
Docker version 1.8.2, build 0a8c2e3

docker@default:~$ ls /
Users/   dev/     home/    lib/     linuxrc  opt/     root/    sbin/    tmp      var/
bin/     etc/     init     lib64    mnt/     proc/    run/     sys/     usr/
{% endhighlight %}

##Create the containers and data volumes
Steps to reproduce: 

**TO DELETE ALL DOCKER IMAGES**

{% highlight bash%}
sudo docker images -q | xargs sudo docker rmi -f
{% endhighlight %}

**TO DELETE ALL DOCKER CONTAINERS**

{% highlight bash%}
sudo docker ps -aq | xargs sudo docker rm -f
{% endhighlight %}

**THE FOLLOWING COMMANDS USE PATHS RELATIVE TO THE FOLDER CONTAINING THIS FILE**

**CREATE MYSQL DATA VOLUME**

{% highlight bash%}
sudo docker create --name mysqldata_1 -v ${PWD}/volumes/mysqldata centos
{% endhighlight %}

**RUN MYSQL SERVER USING THE DATA VOLUME**

{% highlight bash%}
sudo docker run -t --name mysql_1 -e MYSQL_ROOT_PASSWORD=secret --volumes-from mysqldata_1 -d mysql:5.5
{% endhighlight %}

**POPULATE DATA VOLUME**

{% highlight bash%}
sudo docker build -t mysqlclient mysqlclient/.
sudo docker run --name mysqlclient_1 -it -v ${PWD}/volumes/mysqlclientdata:/var/mysql/dump:rw --link mysql_1:mysql mysqlclient
{% endhighlight %}

**CREATE LIFERAY DATA VOLUME**

{% highlight bash%}
sudo docker run -v ${PWD}/volumes/liferaydata:/opt/liferay-portal/data --name liferaydata_1 centos
{% endhighlight %}

**RUN LIFERAY CONNECTED TO MYSQL SERVER**

{% highlight bash%}
sudo docker build -t liferay liferay/.
sudo docker run --name liferay_1 -it -v /tmp/deploy:/opt/liferay-portal/deploy --volumes-from liferaydata_1 --link mysql_1:mysql --publish 127.0.0.1:8080:8080 -d liferay
{% endhighlight %}

**ACCESS LIFERAY'S CONTAINER**

{% highlight bash%}
sudo docker exec -it liferay_1 bash
{% endhighlight %}

**GET MYSQL SERVER IP ADDRESS**

{% highlight bash%}
sudo docker inspect mysql_1 | grep IPAddress
{% endhighlight %}