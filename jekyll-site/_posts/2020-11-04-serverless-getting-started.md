---
layout: post
title: "Beginner Serverless Workflows"
date: "2020-11-04"
categories: serverless
published: true
---

## Current Workflow

Most of the time, I only need to update code in the Lambda editor:

- Update Lambda code and deploy
- Create or use existing test messages
- Check the logs to verify the result 

If I need to verify Lambda is reading events off the SQS, I write the test messages to the SQS topics instead. This workflow is relatively fast since I'm doing my testing within AWS and verifying with the logs. 

However, there are times when I need to make more significant code changes to files that aren't in the Lambda itself. This takes much longer since you can only edit the main Lambda function. 

In cases like these, I need to:

- Update code in my local editor 
- Create a zip of all relevant source files that were updated
- Uploadd the zip to create a new version of the Lambda Layer 
- Remove the old Layer from the Lambda function
- Attacah the new Layer to the Lambda function

This is a very painful process with steps across multiple different environments. 

## Shortening the Feedback Loop

In order to shorten the feedback loop, I've been doing as much development as possible inside the main Lambda editor as possible. 

Functionality that would normally be abstracted into separate files can be either dropped into the main Lambda function or into new files in the Lambda file tree.

That way, I can quickly test changes in the underlying methods without having to go through the zip/upload/bump version process.

Once changes are relatively stable, I only have to upload the final version a single time. 

However, this can get messy if there are multiple files worth of functionality in the editor, and it's definitely possible to miss changes when going back and forth between Lambda and a local editor when checking changes into version control.

## Things to Try

Biggest pain point right now is the process of bumping Lambda Layer versions, which I'm going to try to automate with AWS CLI: 

```pseudocode
LAMBDA_ARN=?
LAYER_ARN=?
SRC_FILES={...}

zip up all src files
create a new version of the Layer with the zip
remove the old Layer version from the Lambda
attach the new Layer version to the Lambda
clean up zip files
```