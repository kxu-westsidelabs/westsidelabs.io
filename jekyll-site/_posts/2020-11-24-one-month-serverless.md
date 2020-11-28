---
layout: post
title: "One Month of Serverless Framework"
date: 2020-11-24
categories: serverless
published: true
---



For the last month, I've been using the [Serverless](https://www.serverless.com/)  framework for the last month to deploy [ConversationsPlus](https://conversationsplus.com/) development and production environments to AWS.

I originally thought that adding Serverless to my workflow was overkill because the app only uses a few Lambda functions, but I've seen an incredible difference in both deploy frequency and confidence. 

The main benefits of Serverless for me has been:

- Reducing mental overhead during the deploy process
- Tightening the feedback loop during development and debugging

Migrating turned out to be a lot less time-intensive than I initially expected - Serverless makes incremental migrations very easy.


## Reducing Mental Burden of Deploys
Before Serverless, deploying application code changes consisted of: (todo: link to previous article)

- Update code in my local editor 
- Create a zip of all relevant source files that were updated
- Upload the zip to create a new version of the Lambda Layer 
- Remove the old Layer from the Lambda function
- Attach the new Layer to the Lambda function

I had to click through all the tabs for Lambda, Lambda Layers, Cloudwatch, and SQS, among others - needless to say it was incredibly time-consuming. 

With Serverless, I can deploy changes to one (or all) Lambda functions directly from the command line. 

Although deploys still take a bit because of all the AWS resources that get provisioned, I don't have to context switch away from the terminal which helps me stay focused.


## Tightening the Development Feedback Loop

One of the main pain points I had with my previous deployment workflow was the lack of visibility into Lambda errors. Changes to Lambda functions were packaged and uploaded in zip files, which were not accessible in the AWS Lambda editor. 

If I ran into any errors during Lambda execution, I would need to either re-upload the source files, or create a new file in the editor and copy over the source. 

This is no longer an issue with Serverless, which deploys Lambda code using S3. All source dependies are automatically available in the AWS editor file tree, which makes it easy to add logging to make changes to handler code or dependencies deeper in the call stack. 

![img](https://lh5.googleusercontent.com/0xzftjX0p2Lu6ZIcUl2OJm417TjxZUce5Lb0xoPdF8RQ45ayMC5a7bJY620EPepgOxUHAWQB76SIpCaqzfOWza9kbStiSdT7RMHhBIS-ZYwd6liQUJTLrSVMHwQY7QJW4OdYDC1n)![img](https://lh5.googleusercontent.com/8ox4BN6QBS2_uqC8CBc2rIqMOQ3ENfR3488Gd90hOaRQGOTyIIeyDIKDq_0vbLMqLHAqjHZVDd58bKXGl6-T3cgz9vuYKoXeyh1TC3iCi2mNHlwA83Za0zmqK6dVGau1bw-SN9At)

## Additional Benefits of Serverless

#### Configs live in a single place - serverless.yml

Makes it easy to come back and review infra. Don't have to do detective work on AWS to figure out what is attached to what

#### Extensive Plugin support	

Many of the more popular plugins (dotenv, iam roles per function), almost feel like native AWS functionality

#### Manage multiple environments

YML is not meant for serious conditional logic, but you can have basic if branching by leveraging object properties. Use this to populate values based on environment and respective dotenv configs

## Migrating to Serverless

Serverless makes it easy to incrementally migrate pieces of your app. 

When you first start building an app, infrastructure changes frequently as you:

- Tweak data models to support new features
- Add message queues, caches, file/blob storage

Once infrastructure is set, it changes much less frequently than the application code. 

Serverless streamlines the deployment of Lambdas, which contain the app code that needs to be updated frequently. 

Other pieces of your infrastructure don't need to live on Serverless since they can updated manually if needed.