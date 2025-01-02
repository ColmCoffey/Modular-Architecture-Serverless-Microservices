# Serverless Microservices Architecture

## Summary
Modern applications demand scalability, flexibility, and cost-efficiency. Serverless microservices architecture meets these needs by enabling modular, independently deployable services. This article introduces a hands-on lab for building a serverless API using AWS services.

The tutorial covers setting up API Gateway as an HTTPS endpoint, integrating it with AWS Lambda, and enabling CRUD operations on DynamoDB. Readers will:
- Define API Gateway resources and methods.
- Build and test Lambda functions connected to DynamoDB.
- Deploy APIs and invoke them via Postman or cURL.

This lab equips engineers and architects with practical skills to create scalable, serverless applications optimized for performance and cost. This lab equips engineers and architects with practical skills to create scalable, serverless applications optimized for performance and cost.

## Skills and Services Used

### Skills Demonstrated:
- Cloud Architecture Design: Creating and integrating modular microservices.
- AWS Serverless Framework: Building serverless applications with Lambda, API Gateway, and DynamoDB.
- Security and Permissions Management: Configuring IAM roles and policies for secure AWS resource access.
- RESTful API Development: Defining resources and methods in API Gateway.
- Event-Driven Programming: Handling requests and responses through Lambda functions.
- Database Management: CRUD operations with DynamoDB.
- Testing and Debugging: Using Postman and cURL for API testing.
- Deployment Automation: Streamlining API and Lambda function deployment.

### AWS Services Utilized:
- Amazon API Gateway – To create and manage APIs as HTTPS endpoints.
- AWS Lambda – For serverless computing and event-driven processing.
- Amazon DynamoDB – As a NoSQL database for scalable data storage and retrieval.
- AWS Identity and Access Management (IAM) – To define permissions and roles.
- Amazon CloudWatch Logs – For monitoring, logging, and debugging application performance.

By leveraging these skills and services, this lab provides a foundation for building scalable and cost-effective cloud-native applications.

## Problem Statement
Problem Statement: Modern applications need APIs that can handle unpredictable workloads while minimizing operational overhead and costs.

Architecture: The solution uses API Gateway as the HTTPS entry point, receiving client requests and routing them to Lambda functions. Each Lambda function executes specific business logic for CRUD operations, with permissions managed through IAM roles. These functions interact with DynamoDB tables for data persistence, providing automatic scaling and consistent performance. CloudWatch logs capture metrics and errors across all components. The architecture follows a RESTful design pattern where each API endpoint maps to specific DynamoDB operations through Lambda functions. This serverless approach eliminates the need to manage servers while ensuring the system scales automatically with demand.

<p align="center">
  <img src="https://github.com/user-attachments/assets/175883bc-c9b6-49d1-9f2b-63f77c9c9578" alt="Figure 1: AWS Serverless API Architecture">
  <br>
  <em>Figure 1: AWS Serverless API Architecture</em>
</p>


## Step-by-Step Guide

### 1. Setting Up IAM Roles and Policies
Why This Step? IAM roles and policies define secure access permissions for AWS resources, ensuring Lambda functions can interact with DynamoDB and CloudWatch.

Steps:
1. Go to the IAM console in AWS.
2. Click Roles in the sidebar and then Create Role.
3. Select AWS Service as trusted entity and choose Lambda.
4. Skip permissions for now and click Next.
5. Name the role: lambda-apigateway-role and click Create Role.
6. Search for the created role, select it, and click Add Permissions → Create Inline Policy.
7. Use the following policy JSON for DynamoDB and CloudWatch:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Scan"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

Save the policy as DynamoDB-CloudWatch-Policy.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f500ca1e-d625-42f8-824e-0e9bc947eb7e" alt="Figure 2: IAM Role Creation and Policy Attachment">
  <br>
  <em>Figure 2: IAM Role Creation and Policy Attachment</em>
</p>

### 2. Setting Up API Gateway
Why This Step? API Gateway serves as the entry point for HTTP requests and integrates with Lambda for processing logic.

Steps:
1. Open API Gateway in AWS and select Create API → REST API.
2. Name the API: DynamoDBOperations and click Create API.
3. Add a resource:
   - Select root / → Actions → Create Resource.
   - Name: DynamoDBManager → Create Resource.
4. Add a POST method:
   - Select POST, integration type: Lambda Function.
   - Enable Lambda Proxy Integration.
   - Link to LambdaFunctionOverHttps.
5. Click Save and deploy the API:
   - Actions → Deploy API → Stage: prod.
   - Copy the Invoke URL.

<p align="center">
  <img src="https://github.com/user-attachments/assets/c15edcb2-ef1f-4d8b-8424-3a1f024d5a22" alt="Figure 3: Configuring API Gateway">
  <br>
  <em>Figure 3: Configuring API Gateway</em>
</p>

### 3. Creating Lambda Function
Why This Step? The Lambda function processes incoming HTTP requests and performs CRUD operations on DynamoDB.

Steps:
1. Go to the Lambda console and click Create Function.
2. Enter function name: LambdaFunctionOverHttps.
3. Select Python 3.12 as runtime.
4. Use the previously created role: lambda-apigateway-role.
5. Replace code with:

```python
import json
import boto3

def lambda_handler(event, context):
    try:
        operation = event['operation']
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

        operations = {
            'create': lambda x: dynamo.put_item(**x),
            'read': lambda x: dynamo.get_item(**x),
            'update': lambda x: dynamo.update_item(**x),
            'delete': lambda x: dynamo.delete_item(**x),
            'list': lambda x: dynamo.scan(**x)
        }

        if operation in operations:
            return operations[operation](event.get('payload'))
        else:
            raise ValueError(f"Unrecognized operation: {operation}")
    except Exception as e:
        return {
            'statusCode': 500,
            'body': json.dumps(str(e))
        }
```

Click Deploy.


<p align="center">
  <img src="https://github.com/user-attachments/assets/f6aa7186-0a1a-4bc9-87e2-d06ebac9dec8" alt="Figure 4: Lambda Function Creation and Deployment">
  <br>
  <em>Figure 4: Lambda Function Creation and Deployment</em>
</p>

### 4. Creating DynamoDB Table
Why This Step? DynamoDB stores data used in CRUD operations.

Steps:
1. Go to DynamoDB and click Create Table.
2. Table Name: lambda-apigateway.
3. Primary Key: id (String).
4. Click Create Table.

<p align="center">
  <img src="https://github.com/user-attachments/assets/333b823c-57b6-44e0-b5fd-b722eb911a67" alt="Figure 5: DynamoDB Table Configuration">
  <br>
  <em>Figure 5: DynamoDB Table Configuration</em>
</p>

### 5. Testing API with Postman
Why This Step? Postman validates CRUD operations and helps debug API requests.

Steps:
1. Open Postman and select POST.
2. Enter the Invoke URL and headers:
   - Content-Type: application/json.
3. Use the following test cases:

Create Operation:
```json
{
  "operation": "create",
  "tableName": "lambda-apigateway",
  "payload": {
    "Item": {"id": "1234ABCD", "name": "Sample"}
  }
}
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/c15edcb2-ef1f-4d8b-8424-3a1f024d5a22" alt="Figure 6: Create Operation Response">
  <br>
  <em>Figure 6: Create Operation Response</em>
</p>

Read Operation:
```json
{
  "operation": "read",
  "tableName": "lambda-apigateway",
  "payload": {"Key": {"id": "1234ABCD"}}
}
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/f1fea988-f96b-4f62-bfbc-bc523fd42004" alt="Figure 7: Read Operation Response">
  <br>
  <em>Figure 7: Read Operation Response</em>
</p>


Update Operation:
```json
{
  "operation": "update",
  "tableName": "lambda-apigateway",
  "payload": {
    "Key": {"id": "1234ABCD"},
    "UpdateExpression": "set #name = :newName",
    "ExpressionAttributeNames": {"#name": "name"},
    "ExpressionAttributeValues": {":newName": "Updated Name"}
  }
}
```
<p align="center">
  <img src="https://github.com/user-attachments/assets/38c004ec-8ae8-4943-8178-1da6831c7fb0" alt="Figure 8: Update Operation Response">
  <br>
  <em>Figure 8: Update Operation Response</em>
</p>

Update operation response.

Delete Operation:
```json
{
  "operation": "delete",
  "tableName": "lambda-apigateway",
  "payload": {"Key": {"id": "1234ABCD"}}
}
```

<p align="center">
  <img src="https://github.com/user-attachments/assets/38c004ec-8ae8-4943-8178-1da6831c7fb0" alt="Figure 9: Delete Operation Response">
  <br>
  <em>Figure 9: Delete Operation Response</em>
</p>

### 6. Cleanup Resources
Why This Step? To avoid unnecessary costs, remove AWS resources when testing is complete.

Steps:
1. Delete the DynamoDB table.
2. Delete the Lambda function.
3. Remove the API Gateway.
4. Detach IAM policies and delete roles.

### 7. Debugging with CloudWatch Logs
Why This Step? Logs help identify errors and track Lambda execution.

Steps:
1. Open CloudWatch → Logs.
2. Select the Lambda log group.
3. Review logs for errors or debugging info.


## Overcoming Challenges

### Overcoming Challenges in Serverless Microservices Implementation
Implementing serverless microservices architecture involves several challenges during setup and deployment. Below, we address key issues specific to the AWS services used in this lab and their solutions.

#### 1. IAM Permission Errors
Challenge: Misconfigured IAM roles and policies may cause Lambda functions to fail when accessing DynamoDB or CloudWatch. 

Solution:
- Ensure IAM roles include necessary permissions for DynamoDB and CloudWatch as shown in the JSON policy.
- Test permissions by invoking Lambda functions directly from the AWS console.

#### 2. API Gateway Integration Failures
Challenge: Lambda integration with API Gateway can fail due to missing configurations or incorrect request mappings.

Solution:
- Verify that Lambda Proxy Integration is enabled for the POST method.
- Test API Gateway resources using the built-in "Test" tool before external validation.
- Check CloudWatch Logs to troubleshoot errors.

#### 3. DynamoDB Table Access Issues
Challenge: Lambda functions may fail when interacting with DynamoDB due to schema mismatches or missing attributes.

Solution:
- Ensure DynamoDB table names and key attributes match those in the Lambda function code.
- Log payloads in CloudWatch to verify input formatting.

#### 4. Lambda Timeout Errors
Challenge: Large payloads or complex operations can cause Lambda functions to time out.

Solution:
- Optimize function code and database queries for performance.
- Increase timeout settings and memory allocation in the Lambda configuration.

#### 5. Testing and Debugging Failures
Challenge: Debugging API and Lambda interactions can be challenging without proper logs.

Solution:
- Enable CloudWatch Logs and review outputs after each test.
- Use structured logging to capture key data points like request IDs.

By addressing these challenges, developers can ensure smooth deployment and reliable operation of serverless microservices built with AWS Lambda, API Gateway, and DynamoDB.

## Optimization Strategies and Further Development

### Optimization Strategies
Building on the serverless microservices architecture, several optimizations can improve performance, scalability, and cost-efficiency:

#### Cost Management:
- Use AWS Cost Explorer to monitor resource utilization and identify savings opportunities.
- Implement resource tagging to track and allocate costs more effectively across teams or projects.

#### Scalability Enhancements:
- Integrate Amazon SQS (Simple Queue Service) and SNS (Simple Notification Service) for asynchronous messaging to decouple services and improve resilience.
- Use DynamoDB Auto Scaling to handle variable workloads without manual intervention.

#### High Availability:
- Deploy APIs across multiple AWS regions and enable failover routing using AWS Global Accelerator to ensure high availability.

#### Performance Tuning:
- Enable caching in API Gateway to reduce latency and lower execution costs.
- Optimize Lambda memory allocation and execution time based on workload profiling.
- Use provisioned concurrency to minimize cold start delays for critical functions.

### Further Development
This serverless microservices architecture provides a foundation for extending cloud-native solutions into innovative domains. Below are projects I'm currently working on that leverage these principles:

#### Retrieval-Augmented Generation (RAG) with DocToc
DocToc is a chatbot system designed for advanced information retrieval and research support. It uses machine learning models and embedded search to quickly process and retrieve data from large repositories, such as research papers. The system was inspired by my experience with embedded knowledge bases and aims to streamline workflows in academic and medical research.

#### Secure Federated Learning with CXR-Secure
CXR-Secure is a privacy-preserving medical research platform. It combines machine learning and homomorphic encryption to allow research institutions to gain insights from clinical data without sharing sensitive information. This project demonstrates how decentralized machine learning models can securely collaborate to improve diagnostics, such as differentiating between COVID-19 and pneumonia cases.

#### AI-Powered Search Platforms with Groce
Groce is an AI-driven grocery shopping assistant designed to optimize cost and convenience. It leverages microservices and AI-enhanced search to improve item categorization and filtering. The platform integrates with APIs to scan supermarket databases, enabling users to find the best deals for their shopping lists.

#### IoT and Edge Computing Applications
My work also includes developing distributed systems for IoT data processing. These applications enable predictive maintenance, real-time monitoring, and smart device integration, pushing cloud-native architectures to edge environments.

If you'd like more information about these projects or have questions about their implementation, feel free to contact me at coffeycolm@gmail.com.

### Next in This Series: Event-Driven Architecture
In the next article of this series, we will explore implementing an event-driven architecture to handle asynchronous workflows and enable loosely coupled services. The tutorial will demonstrate:
- Configuring Amazon SQS and SNS for reliable messaging.
- Triggering AWS Lambda functions based on events.
- Managing distributed transactions using AWS Step Functions.

This evolution builds upon the microservices principles covered in this article, enabling architects to design even more scalable and resilient cloud-native applications.
