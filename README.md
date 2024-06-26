# Ingesting CloudWatch Logs into OpenSearch Serverless

## Overview

CloudWatch Logs provides a robust, scalable solution for collecting, monitoring, and storing log data from AWS services, applications, and systems. It offers real-time monitoring, alerts, retention policies, and search capabilities for log analysis. 

CloudWatch Logs enables highly durable storage of log data by compressing and encrypting it across multiple Availability Zones. It has flexible retention policies ranging from days to years to meet compliance requirements. Older logs can be archived to S3 or Glacier. The service also allows real-time processing to stream log data to other AWS services like Lambda and Kinesis for enabling real-time analysis and alerts. CloudWatch Logs can auto-scale to handle ingestion of terabytes of log data per day. It supports open standard log formats like JSON for easier analysis using tools like OpenSearch and QuickSight. CloudWatch Logs also has tight integration with AWS services such as EC2, Lambda, and CloudTrail for centralized logging across an AWS environment. Other capabilities include search and filter for querying log data along with viewing metrics and creating alarms.

Amazon OpenSearch Service provides a fully managed and serverless OpenSearch experience that removes the need to deploy any infrastructure. It offers automated provisioning, upgrades, and scaling to petabytes of data with millions of documents. OpenSearch Serverless has a pay-per-request pricing model with no minimum fees. It provides encryption both at rest and in transit for secure access controlled via IAM and VPC. Performance is fast with sub-second response times. OpenSearch Serverless integrates natively with services like Lambda, Kinesis, CloudWatch, and S3. It includes dashboards, alerts, SQL and machine learning capabilities powered by the OpenSearch ecosystem. High availability is built-in across Availability Zones along with point-in-time snapshots for backup.

While CloudWatch Logs excels at collecting and storing log data, OpenSearch Serverless provides more powerful search, analytics, and visualization capabilities on that log data. This project implements a serverless pipeline to get the best of both services - using CloudWatch Logs for log aggregation, and OpenSearch Serverless for log analysis.

Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.


## Key Components 

- **Amazon CloudWatch Logs** - collects and stores log data

- **Amazon Data Firehose** - buffers and delivers streaming log data 

- **Amazon S3** - provides temporary storage for log files

- **Amazon SQS** - queues notifications when new log files are ready  

- **AWS Lambda** - transforms log data into optimal JSON format for OpenSearch

- **Amazon OpenSearch Ingestion** - managed service to pull log files from S3 and ingest into OpenSearch

- **Amazon OpenSearch Serverless** - fully managed, serverless configuration for Amazon OpenSearch Service

## Architecture

The architecture diagram below illustrates how these services work together:

![Architecture diagram](architecture.png)

1. CloudWatch Logs sends log data to Amazon Data Firehose  
2. Amazon Data Firehose buffers the data, triggers Lambda to transform the log data, and delivers it to S3
3. S3 notifications are sent to SQS when new log files are added  
4. SQS triggers OpenSearch Ingestion to ingest new data
5. OpenSearch Ingestion pulls transformed logs from S3 and loads into OpenSearch

By orchestrating these components, we can build a flexible, resilient pipeline to make CloudWatch log data available for analysis in OpenSearch Serverless. The serverless approach automates the underlying data flow, providing a managed solution that scales on demand.


## Deliverables

1. The transformation Lambda function used by Kiensis Data Firehose 
2. A sample OpenSearch Ingestion configuration for ingesting data from S3
3. A set of CloudFormation templates to automate deployment of this complete solution


## Deployment Instructions

To deploy this entire solution, use the templates and instructions provided in the ![cloudformation](cloudformation) section.  
This will instantiate all components, including a sample OpenSearch Serverless Collection.  The frontend template will represent
what is needed to configure an application and it's CloudWatch Log Group to ingest data into OpenSearch.  You may optionally
modify the backend template to use an existing collection.


## Build Instructions

The following steps explain how to implement this architecture from the ground up:

### 1. Create S3 bucket

Create an S3 bucket to serve as intermediate storage for the log files. Amazon Data Firehose will deliver logs here before they are processed and loaded into OpenSearch.

### 2. Create SQS queue

Create an SQS queue that will receive notifications when new objects are added to the S3 bucket. This queue will trigger the next step in the process.

### 3. Configure S3 event notifications 

Configure the S3 bucket to send object created events to the SQS queue. This establishes the trigger to process new log files.

### 4. Deploy Lambda function

Deploy the Lambda function that will process and transform the log files into optimal JSON format for OpenSearch. 

### 5. Create IAM roles  

Create IAM roles and policies to allow access between the services. For example, Amazon Data Firehose will need access to write to S3.

### 6. Configure OpenSearch Ingestion  

Create an OpenSearch Ingestion pipeline that is triggered by the SQS queue and pulls log files from S3 to ingest into OpenSearch.

### 7. Create Amazon Data Firehose stream

Create a Amazon Data Firehose stream to receive log data from CloudWatch Logs and buffer/deliver it to the S3 bucket.

### 8. Create CloudWatch Logs subscription

Create a CloudWatch Logs subscription to send log data to the Amazon Data Firehose stream. Apply filters as needed.

### 9. Monitor pipeline 

Monitor the OpenSearch Ingestion metrics and logs to verify the pipeline is functioning. Load test and tune as needed.

This implements the architecture in an automated, scalable serverless pipeline.

## How it works

Here are the key points on how the service interactions work in this architecture:

- CloudWatch Logs collects and streams the log data. Subscription filters route specific log streams to Amazon Data Firehose.

- Amazon Data Firehose buffers the incoming log streams and delivers them to S3. This handles reliable delivery at scale.

- A Lambda function is configured in the Firehose delivery stream to transform the log records into the JSON format required by OpenSearch. 

- S3 provides durable storage for the log objects. When new objects are written, events trigger an SQS notification.

- SQS queues the notifications that new log objects are ready for processing. This decouples the streaming ingestion from the batch indexing.

- OpenSearch Ingestion is configured with a pipeline that pulls log objects from S3 when triggered by the SQS events. It parses and transforms the logs then loads them into OpenSearch.

- OpenSearch Serverless provides the destination search and analytics engine. It scales automatically to handle the load from ingestion and serves the indexed log data.

Together this creates a serverless pipeline to stream, transform, and analyze large volumes of log data. Managed integrations between AWS services handle the complexity.


## Testing

Here are some steps to test and validate this log analytics pipeline solution:

1. Generate test log data
- Use a tool like log-generator to produce sample log entries that mimic your real application logs. Variety of formats, metrics, timestamps, including JSON objects.

2. Stream sample logs to CloudWatch  
- Configure your application or log-generator to push test logs to CloudWatch Logs (e.g. a Lambda function). Send across multiple log groups/streams.

3. Check Firehose delivery stream
- Monitor the Firehose console/metrics to validate logs are being received from CloudWatch subscription. Check for errors.

4. Verify Lambda transformation
- Use CloudWatch Logs for the Lambda function to check that records are being parsed/transformed as expected.

5. Confirm objects written to S3
- Inspect contents of S3 bucket to ensure log files are being delivered by Firehose. Validate data format.

6. Test SQS notifications  
- Monitor SQS queue to confirm messages are being polled by OpenSearch Ingestion Service.

7. Query indexed data in OpenSearch
- Use the OpenSearch dashboard or query API to search and retrieve indexed log documents. Validate correct parsing and mapping of fields. 

Thorough end-to-end testing is key to ensuring a reliable, efficient log analytics pipeline. The steps above validate correctness, security, resilience and optimization of the solution.

# License

This library is licensed under the MIT-0 License. See the LICENSE file.

- [Changelog](CHANGELOG.md) of the project.
- [License](LICENSE) of the project.
- [Code of Conduct](CODE_OF_CONDUCT.md) of the project.
- [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

# Legal Disclaimer

You should consider doing your own independent assessment before using the content in this sample for production purposes. This may include (amongst other things) testing, securing, and optimizing the content provided in this sample, based on your specific quality control practices and standards.
