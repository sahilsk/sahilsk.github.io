---
layout: post
title: "Application Monitoring Dashboard Solution"
modified:
categories: articles
excerpt: 
tags: [dashboard, monitoring, graphite, carbon, polymer, mongodb]
comments: true
share: true
image:
  feature:
author: sahilsk
date: 2014-12-20T05:23:42+05:30
---

In this post i'd discuss some of the option available for application
monitoring. As your application grows, your technology stack also grows
and so the surprises that comes along. These often lead to out-of-service
statuses which I presume no organization,small or big, can bear.

So, let's get started. 

__Application Monitoring__ : It can include minimum monitoring of application
live status and can extend upto application performance monitoring.
When you have all these metrics before you, you'll be able to gauge your
infrastructure efficiency and scalability in more predictive manner.

Imaging your application is getting popular and no. of users are increasing
day by day. With given metrics you'll be able to see how many RAM, CPU or instances
are being used and how much is available. With current resource utilization and
growing user base stats in hand, you can take necessary measures to ensure smooth operation of your
service.

What are these metrics that we should be wary about?
----------------------------------------------------

Let's start from server itself.

Few Important Metrics are:  __RAM__ , __CPU__, __Disk space__, __bandwidh__, __Disk I/O__
, __DB Read/Write__ etc.

* __Disk I/O__

  Processor will wait till process finish reading file. So, having a fast Disk input/output operations on a physical disk will improve your application performance significantly.

  That's why SSD's are nowadays popular. 30% faster file opening speed than traditional HDD. File Copy or Write Speed of SSD is typically about 200 MB/s -550 MB/s while of HDD it's very low around 50-12MB/s.

  You can easily find the difference in turnaround time of your Disk I/O heavy application by just changing a single hardware.
  But do you really require HDD? See your application metrics and answer yourself ;)

* __RAM__

  More the ram, more process can run and stay in memory for long.
  This mean less Disk I/O and page swapping.
  With hardware getting cheap, having extra RAM won't cost much.
  
  However, knowing when to increase RAM is the purpose of this post. One option is just take note of RAM usage in last couple of days and decide your next step.

* __CPU__

  For computation heavy application just RAM is not enough. You need powerful mind to process them all. So, do watch your CPU performances as well. Generally, if metrics shows CPU usage greater than 80% for most of the time, it means you should upgrade to more powerful machines with more cores inside.


* __Bandwidth__

  Not only it'll help track your monthly bandwidth bills but also help you trace anomaly. Many times watching bandwidth usage pattern give you clues on what's happing wrong if anything is going wrong. 


## Next Step is, how to get these metrics?

Start with Web server. If you uses nginx then gather its statistics.
Afterward, gather App Server statistics. If you have RoR then you might have used or tried unicorn. So, you might need to dig into its log file to gather metrics out of it.

* **[PHusion Passenger][0]**

  `Phusion Passenger` make all this easy as it's a web server and App Server, both.
  It provide Administration tools that allow you to detect whether an application is stuck and non-responsive.

  It let you Watch and monitor many application-level, process-level and system-level statistics from a central place.

  * CPU, memory and swap usage, both system-wide and per-process.
  * Connections and requests.
  * Hypervisor VM interference rates.
  * Load averages, fork rates and swap rates.
  * Application-level backtraces.

  Best part is _Easy integration with external tools_.
  These statistics are queryable over an HTTP JSON API, allowing you to easily integrate these statistics with external tools.

  These stats can be pushed into a centralized storage (Mongodb or mysql).


* **[CollectD][1]**

  collectd is a daemon which collects system performance statistics periodically and provides mechanisms to store the values in a variety of ways, for example in RRD files.
  Out of the box it provide multitude of [plugins][2] ( cpu, memory, network, disk etc).
  For bandwidth usage you can look at [collectd-network-bandwidth-usage][3] plugin.

  These stats can be collected in Mongodb through [ Write MongoDB plugin](4) or can be pushed into Graphite db through [collectd-carbon][5] plugin.


## How to create a monitoring Dashboard ?

Let me start with minimal monitoring to advance monitoring options that can be used to monitor your application and the servers it's running on.

* Minimum Application Monitoring: (Monit)

   [monit][6] can be used to monitor your endpoints. It provide a nice and simple web interface to see live status.

   > Monit is a small Open Source utility for managing and monitoring Unix systems. Monit conducts automatic maintenance and repair and can execute meaningful causal actions in error situations.

   It provide a nice Web GUI where you can see live status of application interfaces and services.

   If any service goes down or resource utilization crosses threshold, it'll take action specified by you. Action can be restarting the crashed service,  shooting mail to system admin or execution of your provided script or command.

   Monit can help you if you have small infrastrcture. But as the infrastructure grows, handling Dashboard for every server would be weary. So, to ease this you can use [Monittr][7].

   >Monittr provides a Ruby interface for the Monit systems management system. Its main goal is to aggregate statistics from multiple Monit instances and display them in an attractive web interface.

  For live monitoring and getting on-time system update on service crash/down or excessive resource utilization, Monit can serve your purpose.

  If you want to read resource utilization pattern, to gauge your infrastructure growth and scale infrastructure accordingly you need to store all these metrics somewhere in centralized storage or Database.
  Next solution let you read past stored data as well.


Recently I come across a application monitoring architecture being used in one of the pioneer bread & breakfast service provider.
This solution is taken out of them.

* Monitoring like a boss ( Design Yourself)

{% highlight bash %}
Phusion Passenger        CollectD+  
        +                    +      
        |                    |      
        |                    |      
        v                    v      
     Mongodb        Carbon(Graphite)
         +-------++----------+      
                 ||                 
                 ||                 
                 ||                 
           +-----vv-----+           
           |            |           
           |   JSON API |           
           |            |           
           |            |           
           +-----+------+           
                 |                  
                 v                  
                                    
          +------------+            
          | Your cool  |            
          | Dashboard  |            
          |            |            
          +------------+            

{% endhighlight %}

  Send _Passenger_ data into MongoDB. Send _CollectD_ metrics into _Graphite_ database(_whisper_). Graphite provide JSON API interface [render url api][8] which can provide data endpoint to real time metrics.

Now, with collected JSON data from MongoDB and Carbon, you can have a common central JSON API endpoint that can power your dashboard to visualize metrics.


### Little on choosing technology for building Dashboard

The big problem broken down into small sub problems.

{% highlight bash %}
  
  Get Data -> Store data ->  Provide JSON Interface -> Pull & Visualize Data

{% endhighlight %}
  
For building _realtime dashboard_ you can use any javascript MVC framework. __AngualarJS__ or __emberJS__ .

For creating widgets ( gauge, pie chart, geomap etc) you can use __AngularJS__ template [directives][9] or __emberjs__ [components][10].

While these approaches are fine, but i would like to introduce [web components][11] here, specially [polymer][12].

>Web Components usher in a new era of web development based on encapsulated and interoperable custom elements that extend HTML itself. Built atop these new standards, Polymer makes it easier and faster to create anything from a button to a complete application across desktop, mobile, and beyond.

With polymer you can create custom widgets very cleanly. Best part is you can re-use them in your next cool projects.

So, plugging peices together through standard interface you can build a nice appliclation monitoring dashboard for your infrastructure.



[0]: https://www.phusionpassenger.com/documentation_and_support
[1]: https://collectd.org/
[2]: https://collectd.org/wiki/index.php/Table_of_Plugins
[3]: https://github.com/Cosmologist/collectd-network-bandwidth-usage
[4]: https://collectd.org/wiki/index.php/Plugin:Write_MongoDB
[5]: https://github.com/indygreg/collectd-carbon
[6]: http://mmonit.com/monit/
[7]: https://github.com/karmi/monittr
[8]: http://graphite.readthedocs.org/en/latest/render_api.html
[9]: https://docs.angularjs.org/guide/directive
[10]: http://emberjs.com/guides/components/
[11]: http://webcomponents.org/
[12]: https://www.polymer-project.org/

