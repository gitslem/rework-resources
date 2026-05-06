# Cloud Automation 101: AWS Lambda, GCP Functions & Azure Logic Apps

## Overview
Serverless computing lets you run code without managing servers. Functions execute in response to events and scale automatically.

---

## Part 1: Serverless Fundamentals

### How It Works
```
Event (new file, API call, schedule)
  ↓
Trigger function
  ↓
Execute code
  ↓
Return result
  ↓
Auto-scale: 0 → 1000s simultaneously
```

### Key Concepts
- **Function**: Unit of code
- **Trigger**: Event that starts function
- **Scaling**: Automatic, infinite
- **Cost**: Pay per invocation

---

## Part 2: AWS Lambda

### Core Features
- Up to 15 min execution time
- 10GB RAM allocation
- Supports Python, Node.js, Java, Go, .NET
- Integrates with 200+ AWS services

### Common Triggers
- API Gateway: HTTP requests
- S3: File uploads
- DynamoDB: Database changes
- SNS: Messages/notifications
- CloudWatch: Scheduled events

### Pricing
- First 1M requests: Free
- $0.20 per 1M requests
- $0.0000166667 per GB-second

---

## Part 3: Google Cloud Functions

### Features
- Up to 60 min execution
- 8GB RAM
- Similar language support
- Deep GCP integration

### Best For
- GCP-native workloads
- BigQuery processing
- Pub/Sub messaging

---

## Part 4: Azure Logic Apps

### Features
- Visual workflow builder
- No-code/low-code
- 1000+ connectors
- Enterprise integration

### Compared to Functions
- **Logic Apps**: Visual, no-code, more connectors
- **Azure Functions**: Code-based, more control

---

## Part 5: Decision Framework

### Choose Lambda if
- Heavy AWS usage
- Complex integrations
- Cost-sensitive at scale

### Choose GCP if
- Using Google Cloud ecosystem
- BigData/analytics focus
- Need managed Pub/Sub

### Choose Logic Apps if
- Non-technical users
- Need visual workflow builder
- Heavy enterprise integrations

---

## Summary

**Serverless Computing:**
- Run code without managing infrastructure
- Pay per execution
- Automatic scaling
- Event-driven architecture

**Major Providers:**
- AWS Lambda: Most popular
- GCP Cloud Functions: GCP ecosystem
- Azure Logic Apps: Visual workflows

**Use Cases:**
- File processing
- API backends
- Scheduled tasks
- Data transformation
- Real-time processing

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
