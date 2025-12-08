---
title: "Lambda Functions"
date:
weight: 3
chapter: false
pre: " <b> 5.7.3. </b> "
---

#### Tổng Quan

Tạo ba Lambda functions cho AI-powered assessments và flashcard generation.

#### Lambda Function 1: Writing Assessment

| Cài Đặt | Giá Trị |
|---------|-------|
| **Function name** | `ielts-writing-evaluate` |
| **Runtime** | Python 3.11 |
| **Memory** | 1024 MB |
| **Timeout** | 5 minutes |
| **Trigger** | SQS (ielts-writing-queue) |

**Sample Code:**

```python
import json
import boto3
import os

bedrock = boto3.client('bedrock-runtime')
dynamodb = boto3.resource('dynamodb')

def lambda_handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        writing_sample = body['text']
        user_id = body['userId']
        
        # Gọi Bedrock cho assessment
        response = bedrock.invoke_model(
            modelId='amazon.titan-text-express-v1',
            body=json.dumps({
                'inputText': f"""Evaluate this IELTS writing sample:
                {writing_sample}
                
                Provide scores for: Grammar, Vocabulary, Task Achievement, Coherence.
                Overall band score (0-9)."""
            })
        )
        
        result = json.loads(response['body'].read())
        
        # Lưu vào DynamoDB
        table = dynamodb.Table('ielts-writing-assessments')
        table.put_item(Item={
            'userId': user_id,
            'timestamp': str(context.aws_request_id),
            'assessment': result
        })
    
    return {'statusCode': 200}
```

{{% notice info %}}
**[PLACEHOLDER]** Screenshot: Lambda function configuration
{{% /notice %}}

#### Lambda Function 2: Speaking Assessment

| Cài Đặt | Giá Trị |
|---------|-------|
| **Function name** | `ielts-speaking-evaluate` |
| **Runtime** | Python 3.11 |
| **Memory** | 2048 MB |
| **Timeout** | 15 minutes |
| **Trigger** | SQS (ielts-speaking-queue) |

Bao gồm Amazon Transcribe integration cho speech-to-text.

#### Lambda Function 3: Flashcard Generation (RAG)

| Cài Đặt | Giá Trị |
|---------|-------|
| **Function name** | `ielts-flashcard-generate` |
| **Runtime** | Python 3.11 |
| **Memory** | 3072 MB |
| **Timeout** | 15 minutes |
| **Trigger** | SQS (ielts-flashcard-queue) |

**RAG Pipeline:**
1. Document chunking
2. Titan V2 Embeddings generation
3. Smart query generation (Gemini)
4. Context retrieval
5. Flashcard generation

#### IAM Role cho Lambda

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "dynamodb:PutItem",
                "dynamodb:GetItem",
                "s3:GetObject",
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

#### Deploy Lambda Functions

```bash
# Tạo function
aws lambda create-function \
    --function-name ielts-writing-evaluate \
    --runtime python3.11 \
    --handler lambda_function.lambda_handler \
    --role arn:aws:iam::{account}:role/ielts-lambda-role \
    --timeout 300 \
    --memory-size 1024 \
    --zip-file fileb://function.zip

# Thêm SQS trigger
aws lambda create-event-source-mapping \
    --function-name ielts-writing-evaluate \
    --event-source-arn arn:aws:sqs:ap-southeast-1:{account}:ielts-writing-queue \
    --batch-size 1
```

#### Bước Tiếp Theo

Tiến hành đến [DynamoDB](../5.7.4-DynamoDB/).

