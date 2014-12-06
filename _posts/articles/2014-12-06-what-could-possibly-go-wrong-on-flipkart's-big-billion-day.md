---
laflipkartt: post
title: "What could possibly go wrong on flipkart's Big-Billion-Day?"
modified:
categories: articles
excerpt: 
tags: [flipkart, Big-Billion-Day]
comments: true
share: true
image:
  feature:
author: sahilsk
date: 2014-12-06T16:18:48+05:30
---

This post is in context to flipkart Big Billion Day. Though flipkart guys were prepared for massive traffic with nearly 5000 servers which flipkart claimed were enough to handle 20 times the traffic on normal day. However, things didn't go as expected and traffic soared by many folds. Site ended up throwing 500 errors and customers were bursting out their anger on twitter.

I totally understand 3 lac transaction, just in 6 hours are huge numbers, let alone actual number of hits. I've no doubt that flipkart team had left no stone unturned to make user experience delightful. Sad that things went little off the track.

I've been thinking lately about real cause for this. flipkart have huge number of servers for their disposal. 
Is current architecture ready to scale the servers on Demand? What's the typical server spin up time? Or were flipkart gone out of servers back then? 
I also saw that flipkartr order queue was processing orders till next day. Were flipkart consumers daemon not enough in numbers ?

What's the average server utilization? What become bottleneck during this massive traffic? 

Let us understand flipkartr architecture. My knowledge is short and are from tech talks that flipkart guys gave. (unfortunately, all in Bengalore). So, please pardon my ignorance if i misunderstand anything.

Web Frontend
-------------------------------------

flipkart have frontend built with php that talks to **fk-w3-agent** that allows async logging & connection pooling. Basically flipkart has answered all php shortcomings using it.
Flipkart relies heavily on logs. On big-billion day amount of logs and other data generated must be huge. So, more I/O operation. But i assume flipkart has SSD disks( 30/40 times faster than hdd) with huge disk spaces to mitigate it, with additional logging servers for big day. However, could it be **fk-w3-agent**s consumer processes were spending more time in I/O? Being a java daemon and java procceses JVM memory usage must have soared high. So, more servers might have been required.

Spinning up On-Demand Instances
-------------------------------------

flipkart uses hostdb with puppet for configuration management. For CI/CD they have jenkins pipeline setup that output production ready debian packages and store them in flipkart private local package repository. All flipkart codes are packaged in debian. Spinning up new servers mean downloading and installing these packages which isn't that much fast enough. But i am sure flipkart had image of production servers ready with all packages pre-installed. So, spinning up new servers would have taken comparatively lesser time as all that is left is booting up server and pulling latest configurations.

Database   
-------------------------------------

Also, since number of writes were more during big billion day, flipkart might have also scale out MYSql servers with more masters to allow more write operation, besides scaling out read replicas as well.

Cache & loadbalancer  
-------------------------------------

I don't think Loadbalancers could be a bottleneck as flipkart had additional loadbalancers ready to work in hot-standby mode. However, adding new spinned up back-end servers in loadbalancer configuration could require HAProxy( tcp layer loadbalancer) restart . HAProxy restart time is very negligible and again i assume this cannot be the reason.

Caching layer
-------------------------------------

I doubt Varnish or memcache layer which could be the cause as data was refreshing continuously at rapid rate. Caching layers might be adding additional queries and time.
come to think of it, I found that though some products gone out of stock, site was  showing "out-of-stock" label. On clicking other products with no label, as well users were seeing "out-of-stock" statuses. So, this label update on "homepage" was not in sync with product availability.
Varnish/memcache metrics perhaps may provide more input on this. 

I totally understand flipkart are nowadays very busy and i totally value flipkartr time. But i can't help myself asking them these questions. So, i wrote a mail to flipkart few engineers. Only one of them replied asking me to wait for their blog post. After two months, with no blog update on their tech-blog, i asked the same person again and this time he ended the mail saying "Sorry".