\# AWS Lambda + API Gateway Serverless REST API

![AWS](https://img.shields.io/badge/AWS-Lambda%20%7C%20API%20Gateway-orange)
![Language](https://img.shields.io/badge/Python-3.x-blue)
![Region](https://img.shields.io/badge/Region-ap--southeast--2-lightgrey)




\## 📌 Overview

This project demonstrates a fully functional \*\*serverless REST API\*\* built on \*\*AWS Lambda\*\* (Python) and \*\*Amazon API Gateway\*\*, managed entirely via \*\*AWS CLI\*\*.  

It showcases how to design, deploy, and secure a cloud-native application without using the AWS Management Console.

## 🔎 Quick Look: Lambda Handler

[![View Lambda Code](https://img.shields.io/badge/View-Lambda%20Code-blue)](./lambda_function.py)

```python
import json

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"message": "Hello from Zain's Cloud API!"})
    }


\## 🚀 Features

\- \*\*Serverless architecture\*\* — no servers to manage.

\- \*\*IAM Role with least-privilege\*\* permissions for Lambda execution.

\- \*\*REST API endpoint\*\* exposed through Amazon API Gateway.

\- \*\*JSON response\*\* returned from Lambda to the client.

\- \*\*Deployed \& tested entirely via AWS CLI\*\*.



\## 🛠️ Tech Stack

\- \*\*AWS Lambda\*\* (Python 3.x)

\- \*\*Amazon API Gateway\*\*

\- \*\*AWS IAM\*\*

\- \*\*AWS CLI\*\*

\- \*\*Python\*\*



\## 📂 Project Structure



zain-lambda-api/

│── lambda\_function.py # Python Lambda handler

│── lambda.zip # Deployment package

│── trust-policy.json # IAM trust policy for Lambda

│── output.json # Deployment output







\## 📋 Steps to Deploy

1\. \*\*Create IAM Role\*\* with trust policy for Lambda.

2\. \*\*Attach AWSLambdaBasicExecutionRole\*\* to the IAM Role.

3\. \*\*Package Lambda function\*\* into `lambda.zip`.

4\. \*\*Deploy Lambda\*\* via AWS CLI.

5\. \*\*Create API Gateway\*\* and integrate it with Lambda.

6\. \*\*Deploy API\*\* and get a public URL.



\## 📡 Example API Response

\*\*Endpoint:\*\*  

`https://<your-api-id>.execute-api.ap-southeast-2.amazonaws.com/prod`



\*\*Sample Response:\*\*

```json

{

&nbsp; "message": "Hello from Zain's Cloud API!"

}

🎯 Learning Outcomes
Hands-on experience deploying serverless applications on AWS.

Understanding IAM role creation and permission policies.

API Gateway integration with Lambda.

Using AWS CLI for automation and deployments.

🧑‍💻 Author
Zain Ahmed Hussain

AWS Cloud Practitioner Certified

AWS Solutions Architect Associate Certified

----------------------------------------------------------------------------------

## 🧪 How to Run This Project (AWS CLI, PowerShell, Sydney region)

> Prereqs:  
> • AWS account + configured AWS CLI (`aws configure`)  
> • Region: `ap-southeast-2` (Sydney)  
> • PowerShell on Windows  
> • Python 3.x installed locally (only for authoring)

### 1) Create the Lambda role (IAM)
Create `trust-policy.json` in this repo (already included in my repo — add it if missing):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Principal": { "Service": "lambda.amazonaws.com" }, "Action": "sts:AssumeRole" }
  ]
}

Create role + attach the basic execution policy:


aws iam create-role --role-name ZainLambdaExecutionRole --assume-role-policy-document file://trust-policy.json
aws iam attach-role-policy --role-name ZainLambdaExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
$roleArn = (aws iam get-role --role-name ZainLambdaExecutionRole | ConvertFrom-Json).Role.Arn
$roleArn

2) Package and create the Lambda function
If you change the code, re-zip before updating/deploying:

Compress-Archive -Path .\lambda_function.py -DestinationPath .\lambda.zip -Force

Create the function:

aws lambda create-function `
  --function-name ZainHelloLambda `
  --runtime python3.12 `
  --role $roleArn `
  --handler lambda_function.lambda_handler `
  --zip-file fileb://lambda.zip


Quick local invoke test:

aws lambda invoke --function-name ZainHelloLambda output.json
type .\output.json


3) Create API Gateway (HTTP API) and integrate with Lambda
Create API and capture ApiId:

$apiId = (aws apigatewayv2 create-api --name ZainHelloAPI --protocol-type HTTP | ConvertFrom-Json).ApiId
$apiId


Create Lambda integration and capture IntegrationId:


$accountId = aws sts get-caller-identity --query Account --output text
$integrationId = (aws apigatewayv2 create-integration `
  --api-id $apiId `
  --integration-type AWS_PROXY `
  --integration-uri ("arn:aws:lambda:ap-southeast-2:{0}:function:ZainHelloLambda" -f $accountId) `
  --payload-format-version 2.0 | ConvertFrom-Json).IntegrationId
$integrationId

Create the route and grant invoke permission:

aws apigatewayv2 create-route --api-id $apiId --route-key "GET /hello" --target integrations/$integrationId

aws lambda add-permission `
  --function-name ZainHelloLambda `
  --statement-id apigw-perm `
  --action lambda:InvokeFunction `
  --principal apigateway.amazonaws.com `
  --source-arn ("arn:aws:execute-api:ap-southeast-2:{0}:{1}/*/GET/hello" -f $accountId, $apiId)

Create the prod stage (auto deploy):

aws apigatewayv2 create-stage --api-id $apiId --stage-name prod --auto-deploy

4) Invoke the public endpoint

$invokeUrl = "https://$apiId.execute-api.ap-southeast-2.amazonaws.com/prod/hello"
$invokeUrl


Open $invokeUrl in your browser


Invoke-WebRequest -Uri $invokeUrl | Select-Object -Expand Content

Expected response: 

{"message": "Hello from Zain's Cloud API!"}
