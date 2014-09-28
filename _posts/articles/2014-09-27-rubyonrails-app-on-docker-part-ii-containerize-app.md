---
layout: post
title: "RubyOnRails App on Docker: Part-II Containerizing App"
modified:
categories: articles
excerpt:
tags: ['RoR', 'Docker', 'Deploy', '12factor.net']
comments: true
share: true
image:
  feature:
date: 2014-09-27T16:27:46+05:30
---

RubyOnRails App On Docker: Part-II How are we doing?
===============

Index

- Setup and Install database
- Containerize RoR Apps
- Setup Reverse Proxy using Nginx



1. Setup and Install database
-------------------------

	For this application we'll use MySQL. There are two ways to run MySQL
	- Run without container
	- Run inside Docker


	Q. Any performance impact when runng inside Container?
	---

	In docker, cpu performance is native, disk latency is native,
	memory is not native but could be made. Same is true for network latency.

	Nowadays hardware are cheap but software are costly. So, we don't need to worry about little memory that Docker keep aside. If you really want to squash every single drop, then there are ways to do so. Network latency also can be made as fast as of native. This small minor reduction we can bear.

	So, having compromised with Memory and Network latency, we're proceeding to Dockerize MySQL instance.

	Luckily, there is already mysql Dockerfile ready in the docker hub: [MySQL Dockerfile](https://github.com/dockerfile/mysql) 


	dockerfile/mysql 
	{% highlight bash linenos %}
	#
	# MySQL Dockerfile
	#
	# https://github.com/dockerfile/mysql
	#

	# Pull base image.
	FROM dockerfile/ubuntu

	# Install MySQL.
	RUN \
	  apt-get update && \
	  DEBIAN_FRONTEND=noninteractive apt-get install -y mysql-server && \
	  rm -rf /var/lib/apt/lists/* && \
	  sed -i 's/^\(bind-address\s.*\)/# \1/' /etc/mysql/my.cnf && \
	  sed -i 's/^\(log_error\s.*\)/# \1/' /etc/mysql/my.cnf && \
	  echo "mysqld_safe &" > /tmp/config && \
	  echo "mysqladmin --silent --wait=30 ping || exit 1" >> /tmp/config && \
	  echo "mysql -e 'GRANT ALL PRIVILEGES ON *.* TO \"root\"@\"%\" WITH GRANT OPTION;'" >> /tmp/config && \
	  bash /tmp/config && \
	  rm -f /tmp/config

	# Define mountable directories.
	VOLUME ["/etc/mysql", "/var/lib/mysql"]

	# Define working directory.
	WORKDIR /data

	# Define default command.
	CMD ["mysqld_safe"]

	# Expose ports.
	EXPOSE 3306
	{% endhighlight %}

Dockerfile is quite simple. Isn't it? In starting few lines, we're installing mysql server and granting user root all privileges.

Line 24, however, need little elboration. Data directory will enable direct access to configuration and data files. This I'll answer in my part-III. For now,  lets put parts rolling.

Let's setup and run mysql

{% highlight bash %}
#Pull and Run mysql image
sudo docker run -d --name mysql -p 3306:3306 dockerfile/mysql
{% endhighlight %}


This one line is suffice to run mysql server up and running.
To verify , we'll start mysql client using the same image but different command.


{% highlight bash %}
	sudo docker run -it --rm --link mysql:mysql dockerfile/mysql bash -c 'mysql -h $MYSQL_PORT_3306_TCP_ADDR'
{% endhighlight %}


2. Containerize RoR Apps
-------------------------

 	RoR framework already comes sensible default best practices of Software Development.
 	However, there are few configuration that i'd like to point out:

 	- Session Storage

 		Store session information in database. This will enable our app to behave more like stateless app. Also, this is essential if we want to scale our infrastructure further 

 	- Secrets

 		Database configuration, RoR Secret key (SECRET_KEY_BASE) , environment , smtp credentials, or other 3rd party addons secret that your app might be using, should not be hardcoded in configuration file. Instead, they should be picked from environment.


 	Here's one example showing database credentials being picked from environment.
	
	`config/database.yml`
	
 	{% highlight yaml %}
	production:
	  <<: *default
	  database: <%= ENV['DATABASE_PROD'] %>
	  username: <%= ENV['DATABASE_USERNAME'] %>
	  password: <%= ENV['DATABASE_PASSWORD'] %>
 	{% endhighlight %}


