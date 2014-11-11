---
layout: post
title: "Why docker?"
modified:
categories: articles
excerpt:
tags: [docker, DevOps]
image:
  feature:
date: 2014-11-09T20:15:14+05:30
comments: true
share: true
image: 
  feature: docker_buid_ship_run_lg.jpg
  credit: "mediaagility"
  creditlink: "http://www.mediaagility.com/"
author: sahilsk
published: true

---

Because we all love Docker. <3 <3 <3
====================================

> Build, Ship and Run Any App, Anywhere	
                   - The Docker team

 Docker is all the rage. Let me start this post by putting some metrics here taken from docker official [github repository](https://github.com/docker/docker) on 9th Nov'14 :

 * __16438__ github stars

	Stars are just another way of saying "I love _docker_"	

 * Forked __3225__ times

	Just another way of saying "I care for docker".

 * __666__ active contributors

	open-source community is committed in improving docker more and more. Not only this, with recent collaboration with RedHat, Microsoft and few other cloud providers docker is getting more secure, more reliable and now on a path to become cross-platform soon. Yes, window users soon be able to get the taste of this open-source recipe

 * __77__ releases   

	With more releases coming _Docker_ is getting _better and better_ and this metric conveyed the same. With every new release comes good news for the community. 

    Limit of 127 aufs layer has been removed in earlier version. With  0.9, Docker dropped LXC as the default execution environment,  0.11, comes tightened _security_  , API consistency and more stability, 0.12 with pause/unpause notable feature, 1.0.0 brought more stability and production support, and with 1.0.1 brought enhance security for lxc driver.

	You can read more about changelog [here](https://github.com/docker/docker/blob/release/CHANGELOG.md)

![Alt Contributors ](/images/contributor_github.png)

Docker Project was released as open source in March 2013. Since then these numbers are increasing day by day. 

Not only this, many startups have been built around docker. _Runnable.com_, _Stackdock_, _orchard_, _tutum_, _quay.io_, _deis.io_, etc to name a few. 
There is one beautifully created mindmeister map on [docker ecosyster](http://www.mindmeister.com/389671722/docker-ecosystem) created by Krishnan Subramanian. You will find more names in there.


With Heavily active community, more security enhancement  and soon getting cross-platform : I think these stats are enough to build confidence in Docker And to bring it in production without cringing.
Let me hight few more points on Docker here in this post.


Same performance at almost zero cost
-------------------------------------

> With negligible resource use, you get the same performance as you get running without it. 

Well, firstly docker provide a _virtual environment_ for your technology stack to run in, unlike others who merely provide you just another virtual machine where resources are tightly coupled and worst part is they are not shareable.
With `docker` you get almost zero performance downgrade. 

In performance, it's tantamount to speed you get natively. Here's a brief overview on it.

* __CPU__ :  Native
* __I/O__ : Native on volumes
		make sure that your data set etc. is on volumes)
* __Memory__ :  Docker set aside a little memory for memory accounting. However, it can be native. No overhead if you disable memory accounting (useful for hptc, probably not for everything else)
* __Network__:  There is no overhead if you run with ``--net host` (useful for > 1Gb/s workloads) (or if you have a high packet rate eg. VOIP, Gaming.)

> Productivity and efficiency are just nothing but synonyms of Docker.

So, with docker you can get the same performance for your technology stack as you gets natively. Besides,  you can share resources among many containers. Thereby, leveraging resources efficiently, and thus spinning up less number of servers.


> Where Security, Reliability, Community & production Support make Docker stand apart, performance and super awesome feature list makes it a must have component of every stack. 


Notable docker features
-----------------------------

- Pause/Unpause container
	
	While Stop and Start feature was already there in the Docker, but with recently released docker, you can now `pause` and `unpause` your container as well. 
	
	_Why do you care?_ You can always use SIGSTOP and SIGCONT on all the processes in the process tree for a container to implement it yourself but they are not always sufficient for stopping and resuming tasks in userspace. 

	That's where [cgroup freezer](https://www.kernel.org/doc/Documentation/cgroups/freezer-subsystem.txt) comes into picture and same is now implemented in docker to avail pause/unpause functionality.  
	Docker also exposes this functionality via its API.
	    
-  Build Once, Run Anywhere

	_No more configuration drift, no more surprises_ : This is what Docker guarantee. This is a huge win for DevOps community.

	A experience sysAdmin know steps used to successful deployment of stack on one machine doesn't necessarily bring the same result in other machine as well. 
	But with Docker now it's possible to achieve `immutable servers` , thereby, keeping infrastructure clean from _snowflakes_ ones.  Yeepeee!!!

- Zero Downtime Deployment

	With the advancement in technology zero downtime deployment is possible. Now you no longer need to put "out of service for maintenance" sign board any longer on your product.

	Docker just happen to be one such technology which allow you to achieve this at negligible cost. Without switching back/forth on environment servers, you can deploy your changes, do canary testing and see the features rolling in production right before your eyes without cringing.
	
- Spinning up new instances is a breeze
	
	Some scenerio demands spinning up more server in minimum time to accommodate sudden surge in traffic and when traffic goes down, these servers should perish. Whether you use _aws auto-scaling_ or in-house automation tools, it surely takes minimum of 2 or 3 minutes(without counting server spin-up time) to launch a new application instances. For heavily traffic sites like e-commerce on events like big sale or new product launch sale , these 2-3 minutes is enough to wipe out the entire stock in a flash. 
	Recenlty, leading ecommerce giant of India: Flipkart witnessed public outrage when their infrastructure failed to scale up as per demand. Though they claim to have 5000 servers ready at per usal but even then they failed to keep the end customer happy. Outburst of customer on social platform like twitter and fb was clearly [seen](http://gadgets.ndtv.com/internet/news/flipkart-big-billion-day-sale-riddled-with-problems-602498) . 
	
	As a DevOps, we used to provide on-demand instances using chef-server and aws auto-scaling prior to witnessing miracles of Docker. Whenever a new instance was needed auto-scaling spinned up a new server using pre-configured ami. When server boots up, upstart script run chef-client to pull latest files and configuration required to run the application. This all took 5 to 6 minutes typically.
	With docker now it takes 1-2 minutes or less, which is a big win.

- Maintainable Deployment Scripts
	
	Docker provide Dockerfile for building image. These images when run are called __"containers"__. Whether you want to install nginx or a framework like `ror` , everything you can define in Dockerfile in simple to learn and understand dsl.

	Best part is you can share your images with each other. If you want mysql image, you can simple pull [mySQL image](https://registry.hub.docker.com/_/mysql/) from [docker hub](https://registry.hub.docker.com).  Best part is, _write once, deployed many times_.

- SysAdmin chores made easy and tidy

	- Backups
	- Logging
	- Remote access
	- Moving containers around
	- etc etc

	With mountable volumes, such chores are made very easy. Just spin up your cleaning container using volumes from application containers and with separation-of-concerns you got a tidy way of cleaning and backups of your application. 

	> Docker comes as a boon to chaotic, cluttered life of sysAdmin by making everything easy to handle, maintain and share.


With the incorporation of one tool in the production not only entire vertical in the organisation get the benefits but also end customers 

* Fast change roll out & deployment
* Less number of server spins therefore, reduction in mothly 
* Easier to write and create Dockerfile leads to short learning curve for new member in the team 
* High availability of service keeps customer confident intact.

With Docker there is a win-win for all stakeholders.