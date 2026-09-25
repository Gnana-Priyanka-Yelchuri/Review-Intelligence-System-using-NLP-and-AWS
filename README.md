# Review Intelligence System using NLP and AWS

An AWS-based Natural Language Processing system that analyzes customer reviews using Amazon Bedrock, identifies sentiment and common issues, generates concise summaries, stores structured results in DynamoDB, and sends email alerts for negative reviews.

---

## Project Overview

Customer reviews contain valuable information about product quality, delivery, packaging, and overall customer satisfaction. However, manually analyzing a large number of reviews is time-consuming.

This project automates the analysis of customer reviews using AWS serverless services and Amazon Bedrock.

---

## Objectives

- Automatically analyze customer reviews
- Classify reviews as Positive, Negative, or Neutral
- Identify common issues such as:
  - Delivery
  - Product Quality
  - Packaging
  - Other
- Generate a short summary for each review
- Store the analysis results in DynamoDB
- Send email alerts when negative reviews are detected
- Demonstrate a scalable serverless NLP architecture using AWS

---

## Architecture

```text
                Customer Review Dataset
                         │
                         ▼
                   Amazon S3
                  Data Storage
                         │
                         ▼
                   AWS Lambda
                Data Processing
                         │
                         ▼
              Amazon Bedrock
                 Nova Micro
                NLP Analysis
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
      Amazon DynamoDB           Amazon SNS
       Store Results          Negative Review
                                  Alert
