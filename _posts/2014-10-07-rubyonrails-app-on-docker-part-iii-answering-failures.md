---
layout: post
title: "RubyOnRails App On Docker: Part-III Making it Robust, final conclusion"
date: 2014-10-07T08:12:48+05:30
modified: null
categories: articles
excerpt: null
tags: 
  - RoR
  - Docker
  - Deploy
  - mysql
  - backup & restore
  - 12factor.net
comments: true
share: true
image: 
  feature: null
author: sahilsk
published: true
---


RubyOnRails App On Docker: Part-III Making it Robust, final conclusion
===============

At this point i assume you have containerized RoR app up and running. You can stop and start container at a breeze. Here, we'll try to answer Scale and HA questions for each of our components one by one. 


- Database
	
	- SCALE & HIGH AVAILABILITY 

		You can configure mysql replicas. There are two configuration, of which one you can choose based on your need.

		1. [Master-Master](https://www.digitalocean.com/community/tutorials/how-to-set-up-mysql-master-master-replication)

			Choose this if you have more writes(update, insert) than reads(select) operations.

		2. [Master-Slave](https://www.digitalocean.com/community/tutorials/how-to-set-up-master-slave-replication-in-mysql)

			This is useful if you have more read(select) operations

		(Setting up these configuration is out of scope of this article.)

	- Backup & Restore

		You can use [`mysql_dump` ](http://dev.mysql.com/doc/refman/5.1/en/mysqldump.html) Or [`mysqlhotcopy`](http://dev.mysql.com/doc/refman/5.1/en/mysqlhotcopy.html) , if  tables are MyISAM.
		There is one awesome article on backup & restore: [How To Backup MySQL Databases on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-backup-mysql-databases-on-an-ubuntu-vps). I urge you to read it.

	- Logs management

		logs are created under `/var/log` directory. You can use syslog to ship them to a centralized location, or better if you have `ELK` stack setup that will ship logs to elasticsearch database then you can create a nice visualization for team to look at.

- Application
	
	-	SCALE

		Now you can launch ror containers.Each container is running at different port eg. 49172, 49173 etc. If traffic increase, you want to scale your infrastructure by launching more similar instances. Here comes LoadBalancer into picture.


		On launching new container, just make its entry in this loadbalancer configuration.
		And when container goes down, take off the entry. There is a tool that can help you out with this chore: [BackendUpdater](https://github.com/sahilsk/BackendsUpdater). Basically, it listen to docker events and accordingly update your loadbalancer configuration file followed by configuration reload (`nginx reload`). More about it in next post.

	- High Availability

		For maintenance or upgrade you can take out containers and do your upgrade. When done, bring them back. One by one you can do upgrade, thereby performing canary testing. 

		Also, this enable you zero downtime deployment of your upgrades.

### Why do we need a loadbalancer?

__Loadbalancer__ :  Loadbalancers are one or more servers that forward the traffic to our backend application servers.  
Loadbalancer is required to distributed load among servers. Load distribution is required if there is huge traffic hitting your service and your single server is not able to withstand it.  
If you're running some resource (cpu, ram etc) intensive jobs, like image processing or rendering then also you need a way to distribute load to other idle or lightly utilized servers.  

Loadbalancer is not only used to distribute load only but also to ease deployment with minimal downtime. Just take one server out of pool of servers and do your maintenace or upgradation there. When you are done, put it back online. After your canary testing you can perform the same for other servers as well

So, here in RoR application we're assuming our application to go incredibly famous.Thereby, bringing thousands hits per second. So, to accomodate these many request we'll put loadbalancers.


{% highlight bash%}

...                  +-------+                                           
                     |       |                                           
                     |  L    |        +-+---------+---+                  
                     |  O    |        | +---------+   |                  
T T T   +------>     |  A    |        | | myApp|01|   |                  
R R R                |  D    |        | +------+--+   |                  
A A A   <------+     |  B    +-------->               |        +--------+
F F F                |  A    |        |   +------+--+ |        |        |
F F F   +------>     |  L    <--------+   | myApp|02| | +--->  |DATABASE|
I I I                |  A    |        |   +------+--+ | <---+  |        |
C C C   <-------     |  N    |        |               |        +--------+
                     |  C    |        | +------+--+   |                  
                     |  E    |        | | myApp|N |   |                  
                     |  R    |        | +---------+   |                  
                     |       |        +-+---------+---+                  
                     +-------+                                           

{% endhighlight %}

**loadbalancer**  
{% highlight javascript %}
# Set your server  
# server_name www.example.com;  

upstream containers {

    # Add a list of your application servers
    # Each server defined on its own line
    # Example:
    # server IP.ADDR:PORT fail_timeout=0;
	
    server 127.0.0.1:49172 fail_timeout=0;
    server 127.0.0.1:49173 fail_timeout=0;

}

server {

    # Port to listen on
    listen 80;

    location / {
        # Set proxy headers        
        proxy_set_header        Host $host;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://containers;
		
		# Turn on nginx stats
        stub_status on;
    }
}
{% endhighlight %}

- &nbsp;
	
	- Backup  ( No restoring here) & Logs management

		Since application is stateless, & all information is stored in Database. But still we sometime need to take backups for logs or other items. Here comes role of `volumes` into picture.

		In dockerfile, we'd mentioned mountable directories. 

{% highlight bash %}
# Define mountable directories.
VOLUME ["/etc/dailyReport", "/var/log/dailyReport", "/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx"] 		
{% endhighlight %}

Let's make use of them. 

### Volumes


__volume__ help us create mountable directories inside containers. They are [docker volumes](https://docs.docker.com/userguide/dockervolumes/). These are helpful in number of cases, one of which is __shipping logs__.  
Lets have one container that does the job of shipping logs. Separtion of concerns, by having one container that does one job and does it perfectly.  
We'll create new container called `data container` which will inherits volumes from our app container

	$sudo docker run -d --volumes-from app-001.example.com --name shiplogs.example.com myDockerfile/shiplogs

Inside this container `/var/log/dailyReport` is accessible where app container is squirting log. In our `shiplogs` container we can have a process that ships logs from there to centralized repository. (What that process could be, is left for future post. )  

Similary you can access reverse proxy logs by reading `/var/log/nginx`. It's Useful for debugging purpose for figuring out strange behavior of your application.  

Another mount directory is `/etc/dailyReport` . This one is created to store all configuration files.
Why? If you want to edit run.sh, or unicorn.rb or reverse proxy configuration then you can do this without actually re-building image.  

{% highlight javascript %}
	$docker run -it --rm  -v /var/log/dailyReport:/var/log/dailyReport -v /etc/dailyReport:/etc/dailyReport  -v /var/run/mysqld:/var/run/mysqld:ro -p 49173:80 --restart="always" -e "RAILS_ENV=production" --name app-001.example.com myDockerfiles/dailyreport /bin/bash
{% endhighlight %}

Here i've mounted `/var/log/dailyReport` and `/etc/dailyReport` directory of host onto container. This will make the container to use my configuration files stored at `/etc/dailyReport`. Also, i can see the logs created on host directory for debugging purpose.

## Conclusion

To recap everything,  we've containerized RoR application and we're running one or more instance of it behind loadbalancer. I've also tried my best to answer scale, availability and fault tolerance problem related to each component. So, with this i close my article.  
Comments are most welcome. If you've have any query or have better suggestion, you can write down in comments.  

Later on, i'll pen down centralized logging solution with ELK stack and my experience while working on it.
So, stay tuned. ;)
 











