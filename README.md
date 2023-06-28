# Eventing and M3ssaging Showdown
![Eventing Architecture](./04%20-%20Eventing%20and%20Messaging%20Showdown%20-%20Page%201.png)
## Project Overview
The purpose of this project is to explore and create the various event streams and messaging queues offered by AWS to simulate a real life event-based architecture, having an input Lambda function publish to the various technologies, which then features data manipulation either by another lambda function or by Data Firehose, and the results then stored in an S3 bucket.
### Project Outcome
By the end of this project is to gain a basic understanding of how various eventing and messaging technologies work in AWS and understand the various use cases for each.
### Project Outputs
- Terraform Scripts which contains all AWS resources provisioned
- Python script for all relevant lambda functions
- Notes on each service including what to use and when
### Out of Scope
- Configuring dead letter queues
- Apache Self hosted Kafka
- Kinesis Video Streams

## Project Description

For all of these one-off projects, everything is encapsulated in a brand new AWS account which is then destroyed upon project completion, I would then access the account in Terraform using the `OrganizationAccountAccessRole` in order to prevent any extra configuration in the AWS CLI.

In this project, I spun up an API Gateway instance that, based on the call received by curl, route the call to a lambda function that would interface with DynamoDB and return the results back to the user.

In order to add a layer of protection to the API, I configured a Cognito user pool to generate a JWT token upon a successful authentication call, which is passed in the header of the API call.

I then have Cloudwatch Logs enabled for debugging purposes.

### Pricing
This project did not cost me anything to create and test as all my usage fell under the free tier and was not intensive enough to breach any thresholds.

## Learning from the Project

### Composition of Generator function
The generator function is designed to return a simple json object with the following:
```json
{
	"time": datetime,
	"id": randint(1,1000),
	"service": "eventbridge" | "sqs" | "data stream" | "data firehose",
	"random fact": factoid
}
```

I'm using the facts api from API Ninjas to add a random fact just to add an extra element to the JSON.

There is a separate function for each route which I am then calling within the main `lambda_handler` function just to keep the code as clean as possible.

### Using SSM
One of the interesting challenges in creating my generator function I found was referencing the endpoint for the SQS queue when that value would not be apparent until the TF files had been applied and created.

A neat solution that I found for this was to store the SQS queue as a value within the SSM Parameter Store, and then within my Lambda function, call SSM to retrieve the url. 

This does present an interesting architectural challenge in that that extra call to SSM will increase latency slightly and adds an extra dependency to the function to work, so this may not be the best idea in a production environment but is fine for this use case. An alternative would be to use either the `list_queues` or `get_queue_url` function to call SQS directly and retrieve the URL that way. I chose to use SSM for this instance to gain experience and understand where SSM Parameter Store could be utilised.
### Batching Events and Messages
Many of the services that I tested included the ability to batch messages in order to make effective use of function invocations. Whilst this was not something that I tested in this project it's useful to see how, for AWS estates that make heavy use of Lambda that it's a great way of being able to cut costs (just batching the messages so that each Lambda function processes 2 messages will effectively halve your costs...).

There is a fine balance to be struck when looking at reducing cost by batching as having a lambda function polling for too long could cause a performance bottleneck further down the chain. This can be measured as a KPI and potentially build in some kind of automation so that the amount of messages that Lambda processes per invocation increases with demand.

### Kinesis Firehose
Kinesis Data Firehose was an interesting one to learn about because from all of the marketing material it is described as an ETL service for streaming data however I did not appreciate that it uses Lambda as part of the transformation layer which requires setting up all the necessary permissions and resources.

The built-in blueprint however is really useful and I used that pretty much untouched for my data transformations, this is something I want to cover later to understand how far you can go with these transformation.

### How to configure an Event Backbone in AWS
When I first learned of event-based architectures and event backbones my first assumption was that this was a methodology that would have been serviced by Kinesis, by its ability to stream data at massive scale, this was driven by my inexperience with Kinesis Data Shards which I believed were routing tunnels, rather than an ability to provide more throughput.

In the course of my research, I realised how powerful EventBridge is at routing events and messages which can then feed SQS topics, Lambda functions, even Kinesis streams. I originally didn't give EventBridge much credit as my only usage of it in the past was to trigger state changes in ECS based on cron jobs. EventBridge is definitely a service that I want to pay a closer attention to in the future as it has a lot of potential.

## Resources
- [b] https://medium.com/@zhaoyi0113/how-to-choose-between-eventbridge-and-sqs-in-event-driven-architecture-a9a51efca9c4
- [b] https://aws.amazon.com/blogs/architecture/a-serverless-solution-for-invoking-aws-lambda-at-a-sub-minute-frequency/
- [b] https://stackoverflow.com/questions/40741282/cannot-use-requests-module-on-aws-lambda
- [b] https://api-ninjas.com/api/facts
- [b] https://docs.aws.amazon.com/lambda/latest/dg/with-kinesis-example.html
- [b] https://docs.aws.amazon.com/step-functions/latest/dg/getting-started-with-sfn.html
- [b] https://beabetterdev.com/2021/09/10/aws-sqs-vs-sns-vs-eventbridge/