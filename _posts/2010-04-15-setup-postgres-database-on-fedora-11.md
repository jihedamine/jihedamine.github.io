---
title: "Setup a PostgreSQL Database on Fedora 11"
description: "How to setup a PostgreSQL database on Fedora 11"
layout: "post"
comments: true
date: "2010-04-15 21:41:00"
categories: [Linux, postgres]
---
I found some blog posts and tutorials on how to set up a Postgres server and initiate a database. To make it work on my Fedora 11 system, I made a mix of the various instructions found here and there. Therefore, I'm publishing the steps I had to follow to get a Postgres server running and setup an initial database on Fedora 11 (it should be similar on other Unix systems).

##As root:
1- Install postgres-server, it will install the required postgres client dependency:

{% highlight bash %}
yum install postgresql-server
{% endhighlight %}

2- Initialize the cluster, then start the server

{% highlight bash %}
service postgresql initdb
service postgresql start
{% endhighlight %}

3- Edit the pg_hba configuration file to change authentication permissions. Open the file in a text editor:

{% highlight bash %}
vi /var/lib/pgsql/data/pg_hba.conf
{% endhighlight %}set au888thentication method to trust instead of ident sameuser for local socket, IPv4 and IPv6 connections.

{% highlight bash %}
# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD
# local is for Unix domain socket connections only
local   all         all                               trust
# IPv4 local connections:
host    all         all         127.0.0.1/32          trust
# IPv6 local connections:
host    all         all         ::1/128               trust
{% endhighlight %}

4- Restart Postgres server so that the changes take effect

{% highlight bash %}
service postgresql restart
{% endhighlight %}

##As regular user:

5- Connect to the database using the default account postgres. You will be prompted for a password. Account postgres has a password by default that is 'postgres'.
{% highlight bash %}
su - postgres
{% endhighlight %}

6- Initialize a database cluster and create your database:

{% highlight bash %}
initdb -D database_cluster_name
createdb database_name
{% endhighlight %}
