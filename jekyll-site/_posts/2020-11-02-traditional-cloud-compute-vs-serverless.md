---
layout: post
title: "Traditional Cloud Compute vs. Serverless"
date: 2020-11-02
categories: serverless
published: true
---

# Traditional Cloud Compute vs. Serverless

Cloud computing is developing your applications on completely managed platforms so you can offload processes like provisioning servers, resource management, and scaling workloads. 

The big difference comes down to utilization. In traditional cloud computing, compute and memory resources are allocated in advance for your application.

However, applications run completely on-demand in a Serverless model, which allows for more efficient resource consumption during moments of peak usage and less idle time during off-peak.  


## Serverless in Practice

For the past few weeks I've been playing around with Serverless to evaluate whether it's a viable back-end for [ConversationsPlus](https://conversationsplus.com/).

The current data store is MongoDB hosted by mLab. However, mLab recently got acquired by Mongo and is getting replaced by the Mongo Atlas product, which means an imminent migration for us before they shut down in Jan. 2021. 

The easiest migration path is to another NoSQL store, so we've been considering DynamoDB on AWS. [AWS is the most dominant cloud platform](https://www.srgresearch.com/articles/cloud-market-growth-rate-nudges-amazon-and-microsoft-solidify-leadership) by far, which means well-understood failure scenarios for its services, as well as a large, active community.


## AWS Serverless Stack

With any app these days, at a bare minimum you need a data store, a place to drop all your business logic, and a way to communicate with it. 

The core of AWS serverless consists of:

* **API Gateway** - Service Entry point
* **DynamoDB** - NoSQL data store
* **Lambdas** - On-demand, self-contained functions that can be triggered through various means (API calls, cron scheduler, etc.)

Productions apps need additional functionality like logging and exception handling (**CloudWatch**), repeatable deployments (**CloudFormation**), Permissions and Security (**IAM**). Although the core of the stack is small and tightly integrated, it's this sprawl of adjacent services and connectors that is a big contributor to the steep learning curve. 

## Thoughts so far

Serverless on AWS is very easy to get started with - just walk through a few of the many tutorials and you can stand up some API endpoints to trigger Lambda functions that read/writes data DynamoDB. 

However, manually creating and linking services together is not a sustainable workflow once you're getting ready for production, which needs:

* monitoring for critical errors
* fast deployment of fixes
* support for multiple environments for dev/staging/prod
* handling of sensitive credentials for each of these envs


We are still early on in the migration, but it's becoming clear that we will need a repeatable and realiable process for development and deployment.

I'm planning on exploring the Serverless framework, which will help us automate this process by allowing us to define services, dependencies, and resource requirements up front, so that builds are identical. 


