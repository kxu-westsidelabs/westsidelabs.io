---
layout: post
title: "Event-Driven Applications in a Serverless Enviroment"
author: Kevin Xu
tags: serverless, aws, architecture, node.js
date: 2021-02-26
categories: serverless
published: true
---

### Background

Conversations Plus is a lead generation app that allows Shopify merchants to manage their customer-facing interactions within their shop's admin dashboard.

Merchants can view and respond to questions submitted through their shop's contact form or send targeted messages directly to existing customers - all of which flows through dedicated email accounts provisioned using AWS Simple Email Service (SES).

This allows for seamless customer interactions for both merchants and customers. 

### Challenges

Although the app saw healthy installs after launch, there were a few factors early on that led us to re-evaluate pieces of the technical architecture:

1. High infrastructure costs
2. Critical database services getting sunset
3. High user churn for freemium

The app was hosted on Heroku using a service-oriented architecture. Each service had its own dyno, which was sized based off of each service's resource requirements. Development and production environments had their own set of dynos as well. 

Although this architecture was very easy to set up initially, it became very expensive to support as the number of services increased. 

### Action

In order to optimize the infrastructure cost, we decided to move the majority of the services from Heroku to AWS Lambda. By leveraging a Serverless model, we would only use compute resources as necessary, instead of the "always-on" compute capacity of dynos. 

This pattern works very well for the Conversations app because most of the functionality is event-driven:

- initial app install
- handling shopify events
- email sending/receiving

In addition, events were already being saved in AWS Simple Queue Service (SQS), which aided the migration from Heroku compute model to AWS serverless. 

The final architecture diagram is shown below, along with some takeaways from the development and migration process. 

### Result

After completing the migration to AWS, we saw a 80% decrease in monthly infrastructure costs for the Conversations. 

## Solutions Architecture

![AWS_Architecture_Diagram_(vector)](/assets/cplus_reference_architecture.png)

1. Shopify merchants install the Conversations Plus app on their Shopify store. Customers submit questions to merchants through the shop's contact form.  
2. Requests are routed through the CPlus App and API servers hosted on Heroku and redirected to AWS.
3. Emails between Merchants and Customers are routed through SES, which eventually triggers a Lambda function which parses emails, extracts relevant shop and customer data, then forwards those emails to the proper recipient. 
4. Shopify webhooks get sent and saved to SQS whenever shop events occur (orders, customer updates, etc). Lambda consumes these webhooks and updates records in MongoDB based on webhook types.
5. CloudWatch Events initiate a Lambda function daily which handles async tasks like generating app metrics and calculating billing. New App install events are consumed by an ETL Lambda function which extracts data using the Shopify GraphQL API, transforms the records, and loads them into MongoDB.
6. MongoDB is the primary system of record, which gets used by all Lambda functions. 

## Lessons Learned

### Set SQS Message Visibilitiy to Longer than Lambda Timeout 

#### Problem

While migrating the Customer Import service, we encountered an issue with the Lambda function picking up and handling the same event multiple times concurrently. 

This wouldn't be a huge issue with the other services in the application, but the Customer Import service sends API requests to Shopify, and concurrent exections were causing API throttling for all concurrent executions.

#### Solution

This turned out to be a configuration error between SQS and Lambda.

When SQS is hooked up to Lambda, any events that get saved to a SQS queue will trigger the corresponding Lambda function to run - this decouples the event producer and consumer and allows for async processing of application events. 

SQS message visiblity timeout is a mechanism used to prevent multiple consumers from processing the same message concurrently. If this visiblity timeout is less than the Lambda timeout, the Lambda function will see that the same message is still available in SQS and attempt to process it again. 

In order to prevent the same event from causing multiple, simultaneous Lambda invocations, we set the SQS visiblity timeout to a value greater than the Lambda timeout. [AWS recommends a value 6x higher](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html#events-sqs-queueconfig)

### Re-Queue Lambda functions that exceed max timeout

#### Problem

Another issue we ran into while migrating the Customer Ingest service was long-running Lambda methods. 

The Customer Ingest service syncs Shopify data during the init process for new installs using the Shopify API. However, the amount of work that the service performs varies greatly based on the shop's customer volume. 

Small shops will sync relatively quickly, while much larger shops can take hours to sync because of API rate-limiting constraints, which is much longer than the 15 minute Lambda max execution time. 

#### Solution

The Shopify API returns a `cursor` parameter which is used for pagination purposes. 

In order to gracefully handle long-running Customer Ingest jobs for larger stores, we pass the `cursor` to the service, along with with other shop data required to use the API.

During Lambda invocations, if the `cursor` is avialable, we start fetching data from that position; otherwise, we start from the beginning. 

Lambda invocations that are close to hitting the Lambda timeout but still need to fetch more data can instead write the current position to SQS, where is will be picked up by a fresh Lambda function. 

This looping allows Lambda to leverage SQS to queue and chunk more manageble pieces of work. 

[Diagram of lock-step]

### Always `await` Node.js Callbacks in Lambda Functions

#### Problem

This was one of the more subtle issues we ran into during the migration process. 
Theoretically, we should've been able to move our service application code wholesale into Lambda functions.

However, during testing, we observed that significant portions of functionality across services were not getting executed. This happened from seemingly unrelated events like database operations, calls to other AWS services, etc.

#### Solution

After much research and debugging, we finally discovered that if using Node.js in Lambda, you MUST `await` all of your `async` function calls! 

Node.js uses the event loop to offload asynchronous tasks to the system kernel (multi-threaded), which notifies Node.js when those tasks are complete. This is what allows Node.js to be non-blocking, even those javascript is single-threaded. 

When Lambda functions are invoked, it creates the execution context, which is a temporary runtime environment with resources defined by the function configuration. Once the Lambda function has finished executing, it freezes the execution runtime (and saves it for future invocations).

Without the `await`, Lambda will terminate before all callbacks are complete, which leads to unexpected behavior. 

Updating all application code to await all async callbacks resolved our remaining issues.

### Incorporate Deployment Automation From the Start

#### Problem

Most of the resources in AWS were already created by the time I started working on this migration.

All of these resources were hand-rolled, which means they had to be individually configured whenever changes needed to be applied. Development and production environments each had their own set of resources which also added complexity.

Although this was a relatively small project, having to manage changes manually led to significant mental overhead and a much longer development feedback loop. 

#### Solution

We used the Serverless framework as our infrastructure-as-code tool for managing Lambda application code. 

This significantly increased our deployment velocity while reducing the risk of introducing errors or regressions. 

Future projects will definitely see these tools used from the outset.

AWS CloudFormation supports many stacks for a given project, which allows encapsulation and separation of components (IAM groups/roles, network setup, application code).
