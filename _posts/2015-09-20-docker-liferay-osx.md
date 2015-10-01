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