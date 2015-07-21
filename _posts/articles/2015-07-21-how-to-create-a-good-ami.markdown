---
layout: post
title: How to create a Good AMI?
modified:
categories: articles
excerpt:
tags: ['aws-cloud', 'ami']
image:
  feature: "ami_lifecycle.png"
  credit: "Apache CloudStack"
  creditlink: "http://people.apache.org/~gochiba/en-US/html-single/"
comments: true
share: true
author: sahilsk
date: 2015-07-21T16:25:58+05:30
---

What is Amazon Machine Image(AMI)?
-----------
Quoting from wikipedia:
> An Amazon Machine Image (AMI) is a special type of virtual appliance that is used to instantiate (create) a virtual machine within the Amazon Elastic Compute Cloud ("EC2"). It serves as the basic unit of deployment for services delivered using EC2.  ~wikipedia

Thank you wikipedia (:

Benefits of using AMI
--------------------

- fast server provisioning and spinning
   
    - With the base system ready you don't need to perform same provisioning steps evertime you spin up new instance. Server Provisioning may include ntp setup, user provisioning eg. grant team access to server, application provisioning eg. installing app/web servers , or embedding orgnaization/3rd party credentials in environment variables and many more.
    
    - Since all steps are pre-baked in AMI what left for server is to start the service manager and start serving requests. AMI help you reduce new server provisioning time from 20-30 minutes to mere couple of minutes.

- Be ready for traffic spikes in time
	
	- With surge in traffic if your infrastructure didn't scale in time, there is no point of putting auto-scaling policies. Your server should be fast enough to spin fast so that they can start receving requests fast. AMI help minimize this time and thus answer  your growing traffic in time.	

- Known state of server
  
    - Since all servers are spinned from same AMI it's guarranteed that all are running same packages at same version. Thereby, eliminating surprises. Nobody likes [snowflake servers](http://martinfowler.com/bliki/SnowflakeServer.html) and we should always strive to avoid configuration drift. This is one step towards [immuatable servers](http://martinfowler.com/bliki/ImmutableServer.html)


Considering benefit of AMI, it's essentials to create a good AMI. Here are some steps that you can consider when you create AMI next time.

How to create a good AMI?
-------------------------

Check List:
-----------

1. Let bootup complete proper. check `/var/log/boot.log`. Maybe even let run for 5-7 mins before proceeding.
2. Update all system packages and reboot. ensure all is good here.
3. Stop all running application services 
4. Check the mail queue (mailq/sendmail)
5. If needed flush the queue
6. once the queue is empty, you’re ready to actually make the image 
7. Delete Shell history
8. Clean  authorized keys
7. double check the code is deploying from the right source, boot it up again just in case, double check its using right source etc (if chef/puppet in use)
9. Issue stop to this instance 
10. Proceed with ami creation

--------

Delete Shell history
----------------------
Always delete the shell history before creating your AMI.

    find /root/.*history /home/*/.*history -exec rm -f {} \;

Clean  authorized keys
----------------------

If you're creating Public AMI you may want to perform following steps:

- Exclude SSH authorized keys before creating your AMI.  

        find / -name "authorized_keys" –exec rm –f {} \;

- Ensure that your private credentials for third-party applications and remote services are deleted

    locate "authorized_keys" files on disk, when run as root:   

      	find / -name "authorized_keys" -print -exec cat {} \;
      
WARNING: Please execute above commands only if you know what you're doing only.

----

Your check list may differ depending on your use case. However, with this article i want to share some best practices, i developed while working on AMI, across.

Thanks for dropping by ;)


References:
----------

- [How To Share and Use Public AMIs in A Secure Manner](https://aws.amazon.com/articles/0155828273219400)
