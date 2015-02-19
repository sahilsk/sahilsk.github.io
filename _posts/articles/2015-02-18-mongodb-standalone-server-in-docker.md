---
layout: post
title: "MongoDB Standalone Server in Docker"
modified:
categories: articles
excerpt:
tags: [mongodb, docker, standalone]
image:
  feature:
date: 2015-02-18T19:43:10+05:30
---

MongoDB 2.4.5 inside Docker
--------------------------


## Dockerfile

**Dockerfile** is to Docker what *Makefile* is to make

file _Dockerfile_

``` bash
#
# MongoDB 2.4.5 Dockerfile
#
#
FROM ubuntu:12.04
MAINTAINER stackexpress "http://stackexpress.com"
RUN apt-get update
RUN apt-get install -y make gcc wget
RUN wget http://fastdl.mongodb.org/linux/mongodb-linux-x86_64-2.4.5.tgz -O /tmp/pkg.tar.gz

RUN ln -s /opt/mongodb/bin/mongo /usr/local/bin/mongo
RUN ln -s /opt/mongodb/bin/mongod /usr/local/bin/mongod
RUN (cd /tmp && tar zxf pkg.tar.gz && mv mongodb-* /opt/mongodb)
RUN rm -rf /tmp/*

RUN mkdir -p /data/db
# Define mountable directories.
VOLUME ["/data/db"]

# Define working directory.
WORKDIR /data

EXPOSE 27017
EXPOSE 28017
CMD ["/opt/mongodb/bin/mongod", "--rest"]

```


## Build

Now we'll build the above Dockerfile and tag it under `sahilsk/mongo_2.4.5`.


``` bash
## CD to 'Dockerfile' directory
$ docker build -t sahilsk/mongo_2.4.5  .

```

## RUN

    $ docker run -d sahilsk/mongo_2.4.5


## Lets do some benchmarking

We'll employ [*mongoperf*](http://docs.mongodb.org/manual/reference/program/mongoperf/) here. 

> Mongoperf is a utility for checking disk i/o performance of a server independent of MongoDB. It performs simple timed random disk i/o’s. 


...to be continued
