---
title: "DynamoDB"
date:
weight: 4
chapter: false
pre: " <b> 5.7.4. </b> "
---

#### Tổng Quan

Tạo DynamoDB tables để lưu AI assessment results và generated flashcards.

#### Tạo Tables

**Writing Assessments Table:**

| Cài Đặt | Giá Trị |
|---------|-------|
| **Table name** | `ielts-writing-assessments` |
| **Partition key** | `userId` (String) |
| **Sort key** | `timestamp` (Number) |
| **Billing mode** | On-demand |

**Speaking Assessments Table:**

| Cài Đặt | Giá Trị |
|---------|-------|
| **Table name** | `ielts-speaking-assessments` |
| **Partition key** | `userId` (String) |
| **Sort key** | `timestamp` (Number) |

**Generated Flashcards Table:**

| Cài Đặt | Giá Trị |
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
# Tạo writing assessments table
aws dynamodb create-table \
    --table-name ielts-writing-assessments \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
        AttributeName=timestamp,AttributeType=N \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
        AttributeName=timestamp,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST

# Tạo speaking assessments table
aws dynamodb create-table \
    --table-name ielts-speaking-assessments \
    --attribute-definitions \
        AttributeName=userId,AttributeType=S \
        AttributeName=timestamp,AttributeType=N \
    --key-schema \
        AttributeName=userId,KeyType=HASH \
        AttributeName=timestamp,KeyType=RANGE \
    --billing-mode PAY_PER_REQUEST

# Tạo flashcards table
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

#### Bước Tiếp Theo

Tiến hành đến [Bedrock Integration](../5.7.5-Bedrock-Integration/).

