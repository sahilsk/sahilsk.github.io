---
layout: post
title: How to Write a Cron?
modified:
categories: articles
excerpt:
tags: ["devops", "cron", "architecture"]
image:
  feature: "cron.gif"
comments: true
share: true
author: sahilsk
date: 2016-04-04T17:32:35+05:30
---

How to write a good cron?
--------

A good cron should adhere to unix philosophy of single-responsibility-principle. 
To elaborate further, your cron should do only one thing and should do it right.
Keeping cron code simple and modular will not only help you debug issues easily,
but also help you inherent learning from one cron to another.

Here are few learnings that one can use while writing workers.  
 
### Q. What cron should do?

- Cron script should be light weight and should work as a helpers for workers.
- Cron should create a job , push them to queue and finish. These job would then be processed by workers.
- Cron should fail fast and finish fast
  
### Q. What cron should not do?

-  Cron should never do any processing task. 
-  Cron should not to do any long running operation failure of which can create inconsistencies
   

Other important considerations:
----

###   Logging

- Logs are your single source of truth. If they're lost, you are left scratching scratching your head finding answers to simple bugs
- What to log and what not to log, however could be tricky. To keep it simple
   we advise you never to log any client/customer data. If credentials or
   secrets are required, that should also not be part of your log output. 
- Here are few simple commandments to follow: 
   - Do no log client/customer data
   - Secrets and credentials should not be part of logs
   - No secrets should be provided as run arguments instead use wrapper run script, if required
   - Make use of environment variables as much as possible.
- Ship logs to graylog

###   Alerting

- If your cron raise error it should raise alert right away. 
- Handle every exeception, categorise them and push them to sentry. If immediate attention is required then don't be afraid to raise pagerduty alert
-  post errors/warning to sentry

FAQ
--------

#### Q. How to know if my cron run successfully or not?
A.  — Read Logging part---
      
#### Q. How to get alert on failed cron run?
A. Cron can be failed due to three reasons:

- System kill it eg. oom killer
    Why was cron doing resource intensive operation at first place?  Make it light weight

- Exception raised in code
    Poor exception handing can abort cron. So, improve code quality and log exception on sentry or graylog.
    On exception raise pagerduty alerty right away

- Somebody kill/shift/removed cron from server
    -- Read next faq --


#### Q. What if somebody removed shift/removed cron from server?
A. Improve team collaboration. Keep everybody in sync. 
Avoid manual changes on server. Make crons part of code deployment.
If automation is not in place, contact DevOps to bring it ASAP

#### Q. How to check if cron run on particular day or not?
A. Check in graylog. If it's not there check in sentry. If it is nowhere,
god bless you

 
#### Q. Shouldn't devops monitor cron for us?
A. Go bald and die
