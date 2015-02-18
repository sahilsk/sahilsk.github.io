---
layout: post
title: "Database in Docker"
modified:
categories: articles
excerpt:
tags: [docker, database, mongodb]
comments: true
share: true
image:
  feature:
author: sahilsk  
date: 2015-02-18T18:59:37+05:30
---

Docker take on Database
------------------------

Rarely you might have heard of people running database in container in production environment. If there are any, gutsy they must be. 

Database is critical component of every three tier service. With advent of NoSQL especially Document type databases, like MongoDB, database selection also play a role in deciding your technology stack. Though with failure of any component in stack may take down your whole service along, however, with database down you also looses data. 

> Critics may say, Hey, same is true for any stateful component. But how many stateful component you see beside database nowadays? Smart people don't uses file for persisting any information. Almost, all application states including sessions are stored in persistent fast accessible layer which will be 99% a database. 


## So, Should i dare to run database in Docker?

Very daunting question it might look but if we paraphrase it, it looks like this:

With database running, 

1. I should be able to collect database metrics   
  - most of the metrics usually come from logs  
2. I should be able to shutdwon gracefully, if not in use  
3. I should be able to start it with last saved data  
4. I should have no latency i.e 
  - no network latency  
  - no disk i/o latency 
5. I should be able to scale it 
  - through master-master replica or master-slave replica 
  

Not that daunting. Right? Now, let's try to answer each one of them in this series.


Let me recap docker performance impact on application running inside:
In performance, it’s tantamount to speed you get natively. Here’s a brief overview on it.

- **CPU** :  CPU performance is totally Native. Same as you get without running inside Docker.

- **I/O** :   Docker support[ many storage backends](http://www.projectatomic.io/docs/filesystems/). By default it uses device Mapper by default and can be switched as per need. However, being said this simply switching storage backend Disk I/O won't get you faster **iops** that you get natively. However, if you use [docker volumes](https://docs.docker.com/userguide/dockervolumes/https://docs.docker.com/userguide/dockervolumes/) you'll get native speed. It also have few extra advantages which i'll cover in some future post.


- **Memory** : Docker set aside a little memory for memory accounting. However, it can be native. No overhead if you disable memory accounting (useful for hptc, probably not for everything else)

- **Network**: There is no overhead if you run with `–net host`. It's useful for > 1Gb/s workloads of if you have a high packet rate eg. VOIP, Gaming.



I might be missing few other things connected especially to application like Database. Here, i'd appreciate few help from community to add their views in the comment.


### Let's get started: 

- MongoDB standalone in Docker
- MongoDB master-slave in Docker
