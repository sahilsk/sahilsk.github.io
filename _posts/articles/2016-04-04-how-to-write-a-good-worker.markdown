---
layout: post
title: How to Write a Good Worker?
modified:
categories: articles
excerpt:
tags: ["worker", "devops", "architecture"]
image:
  feature: "workers.jpg"
comments: true
share: true
author: sahilsk
date: 2016-04-04T14:27:53+05:30
---

How to write a good worker?
---------

A good worker should adhere to unix philosophy of single-responsibility-principle. 
To elaborate further, your worker should do one thing and should do it right.
Keeping worker code simple and modular will not only help you debug issues
easily, but also help you inherent learning from one worker to another.

Here are few of these learnings that one can use while writing workers.  In the
end you'll have a robust worker that work as intended.
 
### Logging

- Logs are your single source of truth. If they're lost, you are left scratching scratching your head finding answers to simple bugs 

- What to log and what not to log, however could be tricky. To keep it simple we
 advise you never to log any client/customer data. If credentials or secrets are
 required, that should also not be part of your log output. 

- Here are few simple commandments to follow: 
    - Do no log client/customer data
    - Secrets and credentials should not be part of logs
    - No secrets should be provided as run arguments instead use wrapper run script,
    if required
    - Make use of environment variables as much as possible. They are more
    reliable than human provided configs
    - Use graylog to send logs to, to avoid SSH 


### Retry->Reque->Die

- If worker run on some queue than for every message it failed to process, it
     must push it back to queue to retry again.
- It should not retry forever. After , let say 3 times retry it should raise alert
- Use  aws SQS deadletter queue and visibility timeout feature for this


### Alert

- If your worker found any critical issue, it should raise alert or inform
other rather than depending on other to raise alert for you

Third Party Service

- If worker is relying on any third party service, it should use circuit-breaker pattern

    - Put timeout on these service response time
    - If service didn't respond back , retry again and fail fast
    - Report or alert if 3rd party service is not reliable enough.
    - http://doc.akka.io/docs/akka/snapshot/common/circuitbreaker.html

    ![Circuit-Breaker]({{ site.url }}/images/circuit-breaker.png)
