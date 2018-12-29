---
layout: post
title: "Adoption of Git Backed Workflow: GitOps"
modified:
categories: articles
excerpt:
tags: ['gitops', 'automation']
image:
  feature: "gitops-adoption.jpg"
comments: true
share: true
author: sahilsk
date: 2015-07-21T16:25:58+05:30  
date: 2018-12-29T16:33:43+05:30
---

Adoption of new workflows: GitOps
---

When I first read about GitOps workflow by [weave.works](https://www.weave.works/blog/gitops-operations-by-pull-request), I was a bit surprised by their audacity.
I mean moving your deploy button from a central policy & compliance control dashboard locked by active directory into the wild in git where it's visible to prying eyes, dying to commit to see their code up and running in production. That's really fearless move. Especially for a business where every outage means loss in revenue. Why would someone take chances of failures?

On second thought it all make sense. I can think of two main driving factor for adoption of gitops in coming days

- **Velocity**: Gone are days when quarterly or monthly deployments were enough. So, we need to minimize manual intervention as least as we could. Excuse of security and compliance should not hinder adoption of new technology. With open mindset you can bring the lost velocity back to your team
- **Adoption of more and more open-source tools**: People are trying their best to make this world a better place. Adoption of public clouds eg. AWS or Azure, has given all a common ground to solve a problem once and solve it for all. Tools like terraform, jenkins, and vast knowledge spread across stackoverflow is at your perusal.


Now, lets talk about what are the pre-requisites. So, what has made such pure automation backed deployment workflows like GitOps possibles?


- **Automate every manual steps**: First and foremost be reasonable behind every process you have. Have a constructive arguments for having a manual step for a automatable workflow. Chances are you don't need those manual interventions.
- **Good test coverage**: Adopt [test-pyramid](https://martinfowler.com/bliki/TestPyramid.html) thinking. Have a fine balance between unit and functional tests. More functional tests means more time to complete. So, as much as you can, move 70% of your test cases in unit tests and keep rests for functional and end-to-end tests. This will help you balance velocity and reliability.
- **Diff**: Find a way to generate a diff of your already deployed application and to be deployed one. If you are using k8s you can leverage [kubediff](https://github.com/weaveworks/kubediff) or if you are using terraform in your enviroment you can use [terraform plan](https://www.terraform.io/docs/commands/plan.html) command. This will help you make decisions whether the new version is good to go without any destructive consequences. This step is very important and it's very important to get it right. Destructive actions could be eg. giving application limit-requests that your cluster can't handle, or too open or too close firewall policy, opening ports that are not supposed to be opened, etc.
- **Git branch**: Respect git branches  and keep all branches as close as possible. For a promotion based environments you need to start with least two branches:  qa and master or (develop and master). Every commit in qa branch is deployed in the qa environment. When QA guys give you a go-ahead for production, create a PR out of QA branch and merge it in the master branch. This will be the trigger for deployment in the production. If anything fails, don't rollback anything manually. Commit and send a new pull request and repeat.


You just can't go ahead and start doing gitops. You need a headstart. So, here are simple steps to start using gitops right after reading this article


![GitOps Workflow](/images/gitops-gitrepo.jpg)

1. Create new repository
2. Create three(or more as per your enviroments) branches: QA, STAGING and PROD
3. Create deployment desciptors. One descriptors for each individual services which has the minimal application configuration required by your deployment automation. Eg.
  - `Application version`
  - `healthcheck_endpoints`
  - `healthcheck retry and timeout`: Some application e.g java ones just takes a bit time to bootstrap.
  - `domain and port`
  - `cpu/ram quota`: For docker/k8s applications, have a hard limit on resources
  - `required ports`
  - `notification channels: [Emails][,Slack channels]`
  - `maintainer address`

4. Copy same descriptors in all the branches.
5. Use jenkins to write following generic parametrized pipelines
  - **Healthcheck pipeline**: Given set of parameters eg. service name, it goes and run health check for the service. 
  - **Deploy Pipeline**: Given service with version and enviroment, it should go and deploy it.
  - **Smoke Test Pipeline**: Generic parametrized pipeline to run post deployment checks on a service specified
  - **Regression Test Pipeline**: Generic parametrized pipeline to run end-to-end tests for any service specificed in the run arguments.
6. Now, with all the generic pipelines ready, you need to build a high-level pipeline that combine all these steps togethers. These will be our Release Pipelines. It should have a notification stage before and end of pipeline to notify stakeholders including JIRA.
  - **QA Release Pipeline**:  `Deploy Pipeline` + `Healthcheck Pipeline`
  - **Stage Release Pipeline**: `Deploy Pipeline` + `Healthcheck Pipeline` + `Regression Pipeline`
  - **Prod Release Pipeline**: `Deploy Pipeline` + `Healthcheck Pipeline` + `Smoke Test Pipeline`
7. At this point you should be able to deploy your serivce using above release pipeline, passing parameters like serivce, version and environment to deploy any application in any envrionment. But only one flaw: It still require manual button press. Lets trigger these release pipeline using Git
8. Git-Monitor Pipeline: This pipeline will track pushes in each of the branches and based on the diff using tools discussed above, will trigger any of the **Release Pipeline**. Be caseful for writing this pipeline. It should have following notable checks
  - Commit message must not contain any message like `skip deployment`. If it does, then you're not suppose to deploy this commit
  - Committer should be a whitelisted candidates
  - Commit message should have a valid JIRA ticket, where the pipeline go and check for valid approvals and commit deployment statuses as per compliance requirement.
  - Lastly, take the diff of the deployed version and the requested version to make decision to allow or reject.
  - IF all is good, call the `{ <Environment> Release Pipeline }` to deploy your changes.
  - While deploying for higher enviornments eg. prod, please take more conditions under consideration that suits your company culture and policy.


Git Branch Policy
---

Idea of promotional deployments is to move changes from lower environment to higher environments and each increment gives you a sense of more and more reliable changes coming upstream. Hence, it's important that every environment has similar infrastructure setup( at low scale) and same set of instructions, and deployable artifacts.

- `QA Branch` : Ideally, every commit should be with least friction. Make your nightly build job pipeline to auto-commit the new version in this branch whenever the new application version is ready to trigger automatic deployment. `QA` are made to be broken.So, it should have zero manual intervention.
- `Stage Branch`: That should be comparatively stable and very close to production. If it's broken then folks like capacity planning, regression testers will be very mad at you. Once you have a stable QA version ready, create a PR and merge it in this stage branch after peer review.
- `Prod Branch`: Apply the same scrutiny here.

Extras
---

- Integration with JIRA and Slack.
- On every deployment failure, create a JIRA ticket and assign to the committer. This will help you improve reliability of your dpeloyments gradually
- Pull reports out of git commits, JIRA tickets and pipelines to keep track of frequency of deployments, time to deploy, frequency of failed deployments, and use these metrics to improve your process.


I've done something similar in our docker based services.

Benefits
---

- No baby sitting or hand holding during deployments.
- QA, Devs, Testers no longer has to maintain or worry about deployments. They know new version is just one commit away.
- Rollback is easy and transparent to the team.
- Bringing sense of ownership to every commit: **You break it, you fix it**


Well, think no more, with unwavering faith in your team plunge into GitOps. And you should be able to reduce a significant toil work with this full fledge automatic deployment and rollover workflow.

This post is a summary of our adoption to GitOps workflow and learnings that are worth sharing with others.


