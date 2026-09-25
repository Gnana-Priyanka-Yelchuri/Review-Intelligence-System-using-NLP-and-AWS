# Review Intelligence System using NLP and AWS

An automated pipeline that takes raw Amazon product reviews, cleans and
samples them, then uses a serverless AWS pipeline (S3 → Lambda → Bedrock →
DynamoDB → SNS) to classify sentiment, detect issue categories, generate
summaries, and send real-time email alerts for negative reviews.

![Architecture diagram](screenshots/architecture-diagram.png)

## Why

Manually reading through thousands of customer reviews to spot problems
(bad packaging, late delivery, quality issues) doesn't scale. This project
automates that: it turns unstructured review text into structured signals
(sentiment, issue category, summary) and proactively alerts on anything
negative, without needing to train or host a custom ML model.

## Pipeline

1. **Data cleaning** (`notebooks/datacleaning.ipynb`) — Starting from the
   [Amazon Fine Food Reviews](https://www.kaggle.com/datasets/snap/amazon-fine-food-reviews)
   dataset (~568K reviews), keep only the relevant columns, rename them,
   drop nulls, and take a stratified random sample of 200 reviews per
   rating (1-5 stars) for a balanced 1,000-review dataset.
2. **Amazon S3** — The cleaned `reviews_1000.csv` is uploaded to an S3
   bucket as the pipeline's input storage.
3. **AWS Lambda** (`lambda/lambda_function.py`) — Triggered function reads
   the CSV from S3, randomly samples 20 reviews per run (to control cost
   and avoid bias toward any one review type), and for each review:
   - Calls **Amazon Bedrock** (Nova Micro model) with a structured prompt
     to classify `sentiment`, `issue_category`, and generate a `summary`.
   - Writes the structured result to **Amazon DynamoDB**.
   - If the sentiment is negative, publishes an alert via **Amazon SNS**,
     which emails stakeholders in real time.

```
Amazon S3  →  AWS Lambda  →  Amazon Bedrock (NLP)  →  Amazon DynamoDB
                   │
                   └──(if negative)──→  Amazon SNS  →  Email alert
```

## Repo contents

```
notebooks/
  datacleaning.ipynb        # cleans + samples the raw review dataset
lambda/
  lambda_function.py        # core Lambda handler: S3 → Bedrock → DynamoDB → SNS
screenshots/
  architecture-diagram.png
  cleaned-dataset-output.png
  lambda-function-console.png
  iam-permissions.png
  dynamodb-tables.png
  dynamodb-items.png
  sns-topic.png
  negative-review-email-alert.png
```

## Setup

You'll need:
- An AWS account with access to S3, Lambda, Bedrock (Nova Micro model
  enabled), DynamoDB, and SNS.
- An S3 bucket with the cleaned CSV uploaded to it.
- A DynamoDB table (see `dynamodb-tables.png` / `dynamodb-items.png` for
  the schema used here).
- An SNS topic with an email subscription confirmed (see `sns-topic.png`).
- A Lambda execution role with `AmazonS3ReadOnlyAccess`,
  `AmazonBedrockFullAccess`, `AmazonDynamoDBFullAccess`, and
  `AmazonSNSFullAccess` attached (see `iam-permissions.png`).

Then, in `lambda/lambda_function.py`, fill in your own values for:

```python
BUCKET_NAME = 'your-bucket-name'
FILE_KEY = 'raw-data/reviews_1000.csv'
TABLE_NAME = 'review-analysis'
TOPIC_ARN = 'arn:aws:sns:us-east-1:<YOUR_AWS_ACCOUNT_ID>:review-alerts'
```

> **Note:** the ARN placeholder above is intentional — don't publish your
> real AWS Account ID in a public repo. Keep it in Lambda's environment
> variables or a config file that's gitignored instead of hardcoding it.

## Results

- Reliable extraction and processing of review data from S3.
- Reviews correctly classified as Positive / Negative / Neutral, with
  issue categories (Delivery, Quality, Packaging, Other) and short
  summaries.
- Structured results persisted in DynamoDB for further analysis.
- Real-time email alerts triggered for negative reviews via SNS.

![Negative review email alert](screenshots/negative-review-email-alert.png)

## Future improvements

- Replace batch sampling with real-time processing via API Gateway, so
  individual reviews can be submitted and analyzed on demand.
- Add a small dashboard (e.g. QuickSight or a simple web frontend) over
  the DynamoDB table for browsing results.
- Expand beyond a 1,000-review sample to the full dataset with proper
  batching/throttling against Bedrock's rate limits.

## Tech stack

Python, pandas, AWS Lambda, Amazon S3, Amazon Bedrock (Nova Micro),
Amazon DynamoDB, Amazon SNS, boto3
