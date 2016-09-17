---
layout: post
title: FaaS for Ec2 Auditing
modified:
categories: _posts/articles
excerpt:
tags: []
image:
  feature:
date: 2016-09-17T17:55:59+05:30
---

We know FaaS, function as a service are now the big thing in the market.
AWS lambda, and webtask.io are few good implementations that help you write
serverless code.

You can read more about serverless architecture on [martin fowler
blog](http://www.martinfowler.com/articles/serverless.html).


Here i want to pen down one of the use case that fit perfectly for FaaS :
Auditing

With growing team it become difficult to audit your infrastructure as more and
more resources are being created. Is tagging standards are followed or not, are
security groups security guidelines are followed or not, or the right ssh-key
pair, image-id or what-not is used for creation of new resources are not.

So, wouldn't it be great if we can write one service that poll aws resources
periodically and get us this audit report? YES. But ...

You then have to create infrastructure for this service or cron to run. Launch server,
create image, put monitoring and what not. It's a pain.

But there is one friendly, quick and very cheap way of doing it. Enter: FAAS.

Create your function and you are done. Seriously. That's all it require.  

Webtask.io is one such FaaS. Though AWS lambda is also there but i found webtask.io deployment more easier.

Difference between AWS Lambda vs Webtask.io
--------


- webtask.io currently supports only node.js and it support it pretty well. AWS
  Lambda on the other hand have programming langauge supports.
- webtask.io deployment is easy but how does it handle new deployment is kind of
  scary. All happen behind the back for you. There is no canary testing support.
  Like in case of failure you want to rollback to a old version. In
  webtask.io you have to deploy it again, where in AWS Lambda you just need
  switch pointer to last working code as in capistrano. 
- Code written for webtask.io can be run as it in aws lambda without little or
  not change at all. Webtask.io support AWS lambda compatible [programming model](https://webtask.io/docs/model) as well.
- webtask.io has rich editor that not only highlight code, but also do syntax
  check. 70% errors you can avoid here only.
- AWS lambda docs are very rich and you can finally, every piece of information
  there.
- pricing ???


While working with webtask.io i found some areas that can be improved further.
Here are some of them: 
- Text editor has syntax check support but would be great if before deploying
  webtask client itself can run a syntax check and throw error on console. It's
  painfully very sad seeing your baby crying due to a typo.
- Webtask.io secrets and data storage strategy is very simple to use but
  difficult to figure out in the starting. Little more documentation could have helped here.



Here, in this short tutorial i'll be using webtask.io for writing one auditing
service.

What are we doing?
----

We'll be creating a auditing service that check if launched instance has tags as
per audit rules or not. 
Resource taggging has many benefits but important one is to track [aws resource billing](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html#tag-resources-for-billing)


How are we doing?
-------

We'll be writing a node.js program that will listen to cloudwatch events.
Cloudwatch is an aws service that emits aws events you can subscribe to. These
events can be a launch of an ec2 instance. This is what we'll be tracking.
To subscribe out audit service to cloudwatch events we must create AWS SNS topic
where cloudwatch will send filtered out events pertaining to our rules only.

![Alt cloudwatch](/images/cloudwatch_event_rule.png)


When an instance is launched, event is emitted by cloudwatch and goes to SNS topic.
SNS topic(`ec2_launches_check`) will then hit our audit serivce endpoint with payload
containing instance id. On this instance id, our auditing service will run
audit checks.

Currently, we're checking if instance has "name" and "service" tag
present or not. If it violate this rule, then our auditing service will dispatch alert. 
There are different places where this alert logic can be put:

- Write in the code to send mail/pagerduty alert using 3rd party services like mailgun, etc.

There is a problem with that. If tomorrow you have to update alert subscribers list, you then have to modify the code or update the environment. 

Wouldn't it be better if we can publish alert to one middleware where subscriber
can be added and removed conveniently through nice GUI? AWS SNS topic is one
such place. So, our audit service will publish alert to another AWS SNS topic(`ec2_audit_alert`). And
subscribers can subscribe to it via SMS, EMAIL or call another 3rd party
service.


![Alt architecture](/images/faas_ec2_auditing.png)


Imlementations
------


- Create two aws SNS topic: `ec2_launches_check`, and `ec2_audit_alert`
- Write audit service in node.js
- Create webtask.io task and deploy it with aws credentials and alert topic
  arn(`ec2_audit_alert`). Code is smart enough to subscribe to this sns automatically.
- Create aws cloudwatch event rule and point it to `ec2_launches_check` sns.



That's it. You can the find [project repo
here](https://github.com/sahilsk/webtask.io-examples)
