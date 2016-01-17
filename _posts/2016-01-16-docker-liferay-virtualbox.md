---
title: "Setup a docker environment for Liferay with MySQL inside a VirtualBox VM"
description: "How to setup a docker environment for Liferay with MySQL inside a VirtualBox VM"
layout: post
comments: true
date: 2016-01-16
categories:
permalink: "docker-liferay-virtualbox"
tags: [docker, liferay, virtualbox]
---
#What is Docker?
Docker is a tool that packages an application and its dependencies in a virtual container that can run on any Linux server (The linux server would run inside a virtual machine on OS X and Windows environments).

Docker has been gaining momentum since it was first released two years ago. It commoditized operating system virtualization by providing an abstraction layer over <a href="https://linuxcontainers.org" target="_blank">LXC</a>. Docker is very useful to quickly recreate identical development environements on any machine, simply using container images. The main advantage of OS virtualization, as opposed to hardware virtualization using virtual machines, is that containers are a thin isolated layer that shares the host OS's binaries and libraries, resulting in minimal overhead.

Each docker container would run one process. Multiple containers can be linked together to setup a full environment.
In the following example we will be setting up a liferay environment with a container for Liferay's webapp, linked to a container for a MySQL database server. Each container uses a data volume to persist data.  
The setup can instantiate a vanilla liferay instance, but it can also be tweaked to use an existing database and existing liferay data.

#Using Docker in a non-Linux environment
Docker provides a 'Docker Machine' for OS X and Windows to host a Linux virtual machine that runs docker. However, we won't be using that Docker Machine, but we will set up our own Arch Linux VM and configure networking to enable access to the services running on docker containers from our Windows host.

#Setup docker inside an Arch Linux VM
Installing docker on a Linux system is pretty easy.  
In Arch Linux, we install the docker package with pacman
{% highlight bash%}sudo pacman -S docker{% endhighlight %}
and start the docker service
{% highlight bash%}sudo systemctl start docker{% endhighlight %}
The service can be set to start automatically when the system starts
{% highlight bash%}sudo systemctl enable docker{% endhighlight %}

#Create the docker containers
The docker image definitions and the file resources used in our setup are available in <a href="https://github.com/jihedamine/docker-liferay" target="_blank">this repository</a>

##MySQL
We will begin by creating a MySQL container that will run the MySQL server process and serve as a DBMS for Liferay.  

###Create a data volume for MySQL data
A separate data volume will be used by the MySQL server to store data.
Using separate data volumes to store data has several advantages among which separating specific data from generic container images, keeping the container image size small, reusing the same data across multiple containers, etc. More details on data volumes can be found in <a href="https://docs.docker.com/engine/userguide/dockervolumes/" target="_blank">docker's documentation</a>.

Docker's create command creates a new container. 
{% highlight bash%}
sudo docker create --name mysqldata_1 -v ${PWD}/volumes/mysqldata busybox
{% endhighlight %}
We name this container mysqldata_1, we bind the folder under the current directory/volumes/mysqldata as a volume and we use the busybox docker image to create our container instance.  
busybox is a minimal linux image (about 2.5 Mb in size) which combines tiny versions of many common Unix utilities. That image would be enough for us to create a data volume.

###Create the MySQL server container
We now run a MySQL server that will use the previously created data volume.
{% highlight bash%}
sudo docker run -t --name mysql_1 -e MYSQL_ROOT_PASSWORD=secret --volumes-from mysqldata_1 --publish 0.0.0.0:3306:3306 -d mysql:5.5
{% endhighlight %}
Docker's run command executes a command inside a new container and is equivalent to create followed by run. Here we create a shell container with the -t option and keep the container running in the background with the -d option.    
-t allocates a pseudo-TTY  
--name gives a name to the container (we name this container mysql_1)  
-e is used to set environment variables. Here we set the MYSQL_ROOT_PASSWORD, which is an environment variable used by the mysql docker image to set the MySQL root user's password.  
--publish publishes the container's port(s) to the host. Here we publish port 3306 to any interface (0.0.0.0) listening on port 3306. We publish to all interfaces to be able to connect to the MySQL server from our Windows host. If we have listened only to requests on localhost:3306, we would only be able to access the MySQL server from inside our Arch VM.  
-d makes the container run in background as we want the MySQL server to keep running after issuing the docker run command.  
Finally, we specify the docker image we'll use to create the container. The <a href="https://hub.docker.com/_/mysql" target="_blank">mysql image</a> is a public Docker image available on the Docker Hub. The image's docker hub page has an exhaustive description explaining how to use that image.  
We specify the version of the image to use with \<image-name\>:\<version\>. Here, we are using the 5.5 version of the mysql image. If we don't specify a version, the latest one is used.

###List docker images and containers
At this point, we can list the docker images available on our machine 
{% highlight bash%}
> sudo docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
mysql                5.5                 2b737bd9d030        9 days ago          256.4 MB
busybox              latest              c51f86c28340        4 weeks ago         1.109 MB
{% endhighlight %}
as well as the containers (which are container instances made from the container images)
{% highlight bash%}
> sudo docker ps -a
CONTAINER ID    IMAGE       COMMAND                  CREATED          STATUS                      PORTS                    NAMES
f2d9b7b5c0e3    mysql:5.5   "/entrypoint.sh mysql"   58 seconds ago   Up 57 seconds               0.0.0.0:3306->3306/tcp   mysql_1
f02b2760d37b    busybox     "sh"                     6 weeks ago      Exited (0) 42 seconds ago                            mysqldata_1
{% endhighlight %}
The -a option makes docker display all existing containers including those that were stopped

###Inspect containers
We can inspect a container (see its properties) with the command docker inspect \<container-name\>. 
To get the IPAddress of our mysql_1 container, we would grep the inspect command's results having the string 'IPAddress'.
{% highlight bash%}
> sudo docker inspect mysql_1 | grep IPAddress
 "SecondaryIPAddresses": null,
        "IPAddress": "172.17.0.2",
                "IPAddress": "172.17.0.2",
{% endhighlight %}

###Populate the MySQL database for Liferay
Having started a MySQL server, we will now create a sql file with instructions to create a DB user and a database for Liferay, minimally with the following content. 
{% highlight bash%}
create user lportal identified by 'lportal';
create database lportal CHARACTER SET utf8 COLLATE utf8_general_ci;
grant all privileges on lportal.* to lportal;
flush privileges;
{% endhighlight %}
We could also add the Liferay tables creation scripts as well as import some data in these tables in the initdb.sql file. 

The SQL file is imported in our MySQL server running in the mysql_1 container and persisting data in the mysqldata_1 data volume:
{% highlight bash%}
sudo docker exec -i mysql_1 mysql -uroot -psecret < initdb.sql
{% endhighlight %}

##Liferay
Now that we have a MySQL database ready for Liferay, we will create the Liferay docker image itself.
  
###Create a data volume for Liferay data
Similarly to the data volume used to persist the MySQL server's data, a second data volume will be used to persist our liferay container's data. The Liferay data volume is also based on the tiny linux image of busybox.

{% highlight bash%}
sudo docker run -v ${PWD}/volumes/liferaydata:/opt/liferay-portal/data --name liferaydata_1 busybox
{% endhighlight %}

###Create the Liferay image
A docker image, like the Liferay image we will create, is defined using a Dockerfile. To quote the official Docker documentation, a Dockerfile is "a text document that contains all the commands a user could call on the command line to assemble an image". 

The Dockerfile of our Liferay image is based on an Oracle JDK Docker image. It downloads Liferay, unzips and installs it, then adds configuration files, exposes the 8080 port and defines a data volume in /opt/liferay-portal/data folder. 

The full content of the Dockerfile is as follows:
{% highlight bash%}
# Based on oracle-jdk 1.7 from airdock
FROM airdock/oracle-jdk:1.7

# Install unzip
RUN apt-get update && apt-get install -y unzip

# Download and install liferay
RUN curl -OL http://downloads.sourceforge.net/project/lportal/Liferay%20Portal/6.2.3%20GA4/liferay-portal-tomcat-6.2-ce-ga4-20150416163831865.zip
RUN unzip liferay-portal-tomcat-6.2-ce-ga4-20150416163831865.zip -d /opt
RUN rm liferay-portal-tomcat-6.2-ce-ga4-20150416163831865.zip

RUN ln -s /opt/liferay-portal-6.2-ce-ga4 /opt/liferay-portal
RUN ln -s /opt/liferay-portal/tomcat-7.0.42 /opt/liferay-portal/tomcat

# Add configuration files
ADD resources/portal-ext.properties /opt/liferay-portal/portal-ext.properties
ADD resources/portal-setup-wizard.properties /opt/liferay-portal/portal-setup-wizard.properties
ADD resources/setenv.sh /opt/liferay-portal/tomcat/bin/setenv.sh

# Liferay data will be stored in a separate data volume
VOLUME /opt/liferay-portal/data

# Expose port 8080
EXPOSE 8080

# Set JAVA_HOME
ENV JAVA_HOME /usr/lib/jvm/java-7-oracle/

# Execute liferay
ENTRYPOINT ["/opt/liferay-portal/tomcat/bin/catalina.sh"]
CMD ["run"]
{% endhighlight %}

###Run the Liferay image
We build the Liferay image using the Dockerfile previously defined. Next, we run that image, linking it to the mysql_1 container that is executing the MySQL server, and using the liferaydata_1 data volume to store liferay's data.

{% highlight bash%}
sudo docker build -t liferay liferay/.
sudo docker run --name liferay_1 -it -v /tmp/deploy:/opt/liferay-portal/deploy --volumes-from liferaydata_1 --link mysql_1:mysql --publish 0.0.0.0:8080:8080 -d liferay
{% endhighlight %}

Some of the options passed to the run command:  
-i keeps STDIN open even if not attached. -i and -t (also written -it) must be used together to allocate a tty for interactive processes.  
-v creates a bind mount with \<host-dir:container-dir\>  
--volumes-from mounts all volumes from the given container(s)   
--link adds a link to another container 

Once the Liferay server container is started, we can connect to it with a shell session
{% highlight bash%}
sudo docker exec -it liferay_1 bash
{% endhighlight %}

#Connect from the Windows host
Now that the containers are running in our Arch Linux VirtualBox image, we need to setup port forwarding to be able to get, from our Windows host, to the ports for which the Liferay and MySQL servers are listening.

##Setup port forwarding
In VirtualBox manager, we open the Arch Linux VM settings, and go to the Network options. 
From there, we setup a NAT network adapter.
<image src="/images/docker-liferay-virtualbox/portForwarding.png" alt="VirtualBox port forwarding"/>
Clicking on "Port Forwarding", we can setup port forwarding rules.  
The following rules state that every request going to the ports 8080 and 3306 on our Windows localhost will be forwarded to the ports 8080 (tomcat server running Liferay) and 3306 (MySQL server), respectively, on the virtual machine.
<image src="/images/docker-liferay-virtualbox/portForwarding2.png" alt="VirtualBox port forwarding"/>
More information on VirtualBox's Networking and the NAT mode can be found <a href="https://www.virtualbox.org/manual/ch06.html#network_nat" target="_blank">here</a>.

##Connect to MySQL database from host
We can check that the MySQL server is up and that it is accessible from the Windows host by connecting to a localhost MySQL server on port 3306, using any MySQL client on the Windows host.
<image src="/images/docker-liferay-virtualbox/mysql-access.png" alt="Connect to the MySQL server from host"/>

##Connect to Liferay webapp from host
We can also get to the Liferay webapp from the Windows host by connecting to its port 8080 from a web browser. The request will be forwarded to the port 8080 of the Arch Linux VM, where the Liferay container is listening to all the requests coming to port 8080. 
<image src="/images/docker-liferay-virtualbox/liferay-access.png" alt="Connect to the MySQL server from host"/>

#Wrap-up
In this article, we have setup docker inside an Arch Linux VM running on a Windows host. Then, we created data volumes to store MySQL and Liferay data, as well as a MySQL container and a Liferay container linked to it. After that, we setup port forwarding in our Arch Linux VM to be able to get to the Docker containers from the Windows host.

You are welcome to have a look at the <a href="https://github.com/jihedamine/docker-liferay" target="_blank">project files</a> and contribute with your comments below.
