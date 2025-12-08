---
title: "DynamoDB"
date:
weight: 4
chapter: false
pre: " <b> 5.7.4. </b> "
---

#### Overview

Create DynamoDB tables to store AI assessment results and generated flashcards.

#### Create Tables

**Writing Assessments Table:**

| Setting | Value |
|---------|-------|
| **Table name** | `ielts-writing-assessments` |
| **Partition key** | `userId` (String) |
| **Sort key** | `timestamp` (Number) |
| **Billing mode** | On-demand |

**Speaking Assessments Table:**

| Setting | Value |
|---------|-------|
| **Table name** | `ielts-speaking-assessments` |
| **Partition key** | `userId` (String) |
| **Sort key** | `timestamp` (Number) |

**Generated Flashcards Table:**

| Setting | Value |
|---------|-------|
| **Table name** | `ielts-generated-flashcards` |
| **Partition key** | `userId` (String) |
| **Sort key** | `flashcardId` (String) |
| **GSI** | `sourceDocument-index` |

{{% notice info %}}
**[PLACEHOLDER]** Screenshot: DynamoDB table configuration
{{% /notice %}}

#### AWS CLI Commands

```bash
# Create writing assessments table
aws dynamodb create-table \
    --table-name ielts-writing-assessments \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
        AttributeName=timestamp,AttributeType=N \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
        AttributeName=timestamp,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST

# Create speaking assessments table
aws dynamodb create-table \
    --table-name ielts-speaking-assessments \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
        AttributeName=timestamp,AttributeType=N \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
        AttributeName=timestamp,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST

# Create flashcards table
aws dynamodb create-table \
    --table-name ielts-generated-flashcards \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
        AttributeName=flashcardId,AttributeType=S \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
        AttributeName=flashcardId,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST
```

#### Enable Point-in-Time Recovery

```bash
aws dynamodb update-continuous-backups \
    --table-name ielts-writing-assessments \
    --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true
```

#### Next Steps

Proceed to [Bedrock Integration](../5.7.5-Bedrock-Integration/).

