---
layout: post
title: "RubyOnRails App on Docker: Part-I Understanding Specs"
modified:
categories: articles
excerpt:
tags: ['RoR', 'Docker', 'Deploy', '12factor.net']
comments: true 
share: true
image:
  feature:
author: sahilsk
date: 2014-09-27T15:12:48+05:30
---

RubyOnRails App On Docker: Part-I What are we doing?
===============

In this post, I'll try to pen down steps to deploy RubyOnRails using Docker.
Before i begin, i assume readers to have basic understanding of Docker and Dockerfile. Albeit i'm trying to keep my post generic, independent of application layer technology, but having little RoR knowledge will help them get best out of this article.

Docker is all the rage nowadays. Though linux containers are there in linux for many years, but their real potential was sighted by Jérôme Petazzoni. Being a nescent technology doesn't stop CTO from rolling it out on their Production.
[Runnable](http://runnable.com "runnable.com"), [NewRelic](newrelic.com "newrelic.com"), [Shippable](http://shippable.com "shippable.com") etc, are all living on the edge and using Docker in their Day-to-day production as well as development work.

### What influence CTO Decision?
As a CTO, you need to think on wide perspective before adopting any new technology. Scalability, High Availability, Downtime, skills availability in team, time to learn etc etc. To put in simple terms, its not easy for new technology to come in limelight so easily. Technologies like HAProxy, Zookeeper are all battle tested and proven ones. But same is not true for Docker. 

However, wide early adoption of Docker by many silicon valleys lean startups has debunked this fact. Their stories has inspired many others. You can read one from NewRelic [here](http://blog.newrelic.com/2014/08/12/docker-centurion/) 

Recently, Amazon web service have started giving Docker container support. 
Recent Docker [v1.2.0](https://blog.docker.com/2014/08/announcing-docker-1-2-0/) release comes with enhanced security features that further support Docker acceptance in Production.
RedHat collaboration with Docker further emphasize that **Docker is ready for production**.


### Coming to the post, let me paraphrase the title of this article:

> Deploying RoR app using Docker


Let me break down the word "Deployment" as per DevOps dictionary:

- Setup and configure Unicorn
- Setup and configure Reverse Proxy: Nginx
- Setup Database : MySQL
- Pre-compile Assets and configure db settings

Wait, there's more. How will you `scale` your application? How will you ensure `High availability`(further onwards as HA) of your service and commit 99.999% uptime to your customers?

Lets give them a short visit here:

- `SCALABILITY`

    If you application is based on [12 factors](http://12factor.net)  commandment , then your app is likely to be scalable. If you don't know these 12 factors, i strongly suggest you to visit the [site](http://12factor.net) and skim it in one go.
    
    Expanding further, Apps can be of two type:
    
    - Stateless
    
        Stateless applications are easy to scale up and down. If it's [12factor](http://12factor.net) app then scalability is as  easy as spinning up/down more instances. 12 factors commandment make scalability a breeze.        

    - Stateful
    
        Stateful application, like Databases, are comparatively difficult to scale. However,  some database does provide clustering and sharding, out of the box. You'll  like to consider this criteria while making DBMS decision for your app.
        

- `HIGH AVAILABILITY`

    With the advancement in technology, Zero Downtime deployment is no longer a dream. If your infrastructure is not yet support zzero downtime deployment, then indeed you're living in Rock age. 12 factors app also enable you to have zero  downtime of your production infrastructure. 
    
    
#### Finally, some sys-admin chores:

1. Backup & Restore
    
    Backup database data,  or configuration files periodically and ship them to a safe place (Centralized server or S3 Buckets )

2. Logs Management

    Logs are no longer neglected in today's age of Big-Data. Management need information to aid their decisions. Logs help them provide those inputs. These inputs can be Geographical, or User browsing or buying trend, all gathered from logs.

---

Now, having knowns the full requirement, we'll visit them one by one in this series of articles divided in three different parts.

1. Part-I : What are we doing?

	Understanding the full deployment requirement.

2. Part-II : How are we doing?

	This part will include: 

	-	Setup and Installation of Database
	-	Containerize RoR App with webserver
	-	Reverse Proxy : Why and How?

3. Parth-III : Conclusion

	In this last article I'll revisit this post to answer our deployment requirements.
