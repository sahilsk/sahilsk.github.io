---
layout: post
title: "Do We Need a Application Configuration Manager?"
modified:
categories: articles
excerpt: 
tags: [configuration management]
comments: true
share: true
image:
  feature:
author: sahilsk
date: 2014-11-11T14:09:10+05:30
---

Do we need a centralized application configuration manager ?
============================================================

NOTE: Please don't get confused with Configuration Management tools like Ansible or opscode-chef. Here, the point is after application deployment on server if any configuration change in run parameter is required, then how to avail those to your already running application?

Let's start with deployment. For deployment any CM tool will do: Ansible,chef. I prefer Ansible as it leave almost zero footprint on deployed server.

For application configuration management, you've two ways:

1. Run your cm script(ansbile,chef/puppet) again, OR

2. Make your application listen to configuration change event.
    
    In case of any change, your application should pull changed configuration, update itself and restart (if required,in most cases it'll be)

 
I'll chose first option. And I have reasons for it.
---------------------------------------------------

In production environment(especially, of payment sites) I cannot afford my servers to behave in unpredictable manner. Change in configuration should be tested first in staging, then should go through canary testing before replicating changes in all production instances. 

If you are planning of creating application configuration manager, then be prepared to make your applications to handle cases that comes with this change. Few of those cases are included here:

* what if centralized configuration server is down?
    
    Your centralized configuraiton server could easily become single point of failure and can become bottlneck. So, additional care need to put here of maintaining it. You need to find ways to make it more fault tolerant and highly available. After all you have given promise of 99.999% availability to your end customers.
    
* what to do if configuration change event triggered?

    This, however, is the toughest case to handle and the sole purpose of bringing centralize configuration manager in stack. 
	To propagate a change, first need to make sure that on first server changes reflected perfectly i.e _Canary Test Passed_ and it's running smoothly. Only then, change should be allowed to be reflected on rest of the instances. So, this incremental update you need to implement, handle & test.
    
    Simplest implementation could be just a fast database(preferably in-memory db like redis) with fault-tolerance, & high availability. If you use Redis then you can also make use of its pub-sub feature.
    If you need to reflect changes speedily, then distributed key-value db like zookeeper or etc can be used.

    Basically you'll end up writing one daemon that does and make decision likes rolling back on failure, notify your monitoring server about the propagated change and its effect: Was it successful ? Why it failed? blah blah?
    
Oh, dear you just added one more component in your architecture. That mean maintenance, support, bug fixing, release and finally, more headache.


I would prefer first option as it makes my life simple. Here is how?
---------------------------------------------------------------------

What makes an ideal production server?

* __Immutable server__:
    
    A server that once deployed, is never modified, merely replaced with a new updated instance. Even slightest change like configuration key change should not be allowed. This way we can keep our infrastructure in known-state all the time.
      
* __phoenix server__:

    Servers should automatically re-syncs with a known baseline. Tools like Ansible, Puppet and Chef have facilities to do this. Due to many chaotic reasons if server reboots then you can simply make use of CM features like `chef-client` in case of opscode-chef and `ansible-pull` in case of Ansible, to bring server back to known-state. 


I use Ansible to create immutable server instances. If any change i need propagate i re-run my ansible script, update changes, test, and if passes, then run the script on all other servers.
For automatically re-syncs on restart/boot i run `ansible-pull` that pulls changes automatically.

