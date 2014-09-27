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



Setup and Install database
-------------------------

For this application we'll use MySQL. There are two ways of setting it up MySQL 
- Run isolately without using docker
- Run inside Docker


Q. Any performance impact when runng inside Container?
---

In docker, cpu performance is native, disk latency is native,
memory is not native but could be made. Same is true for network latency.

Nowadays hardware are cheap but software are costly. So, we don't need to worry about little memory that Docker keep aside. If you really want to squash every single drop, then there are ways to do so. Network latency also can be made as fast as of native. This small minor reduction we can bear.

So, having compromised with Memory and Network latency, we're proceeding to Dockerize MySQL instance.

Luckily, there is already mysql Dockerfile ready in the docker hub: [MySQL Dockerfile](https://github.com/dockerfile/mysql) 


{% highlight json %}
dockerfile/mysql 
```
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

Dockerfile is quite simple


