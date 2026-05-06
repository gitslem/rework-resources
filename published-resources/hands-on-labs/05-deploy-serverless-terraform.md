# Deploy Serverless Functions with Terraform
## Professional Infrastructure-as-Code Implementation Guide

---

## Table of Contents

1. [Introduction](#introduction)
2. [Serverless Architecture Fundamentals](#serverless-architecture-fundamentals)
3. [Terraform Essentials](#terraform-essentials)
4. [Getting Started](#getting-started)
5. [Deploying to AWS Lambda](#deploying-to-aws-lambda)
6. [Multi-Cloud Deployment](#multi-cloud-deployment)
7. [Advanced Infrastructure Patterns](#advanced-infrastructure-patterns)
8. [Monitoring & Observability](#monitoring--observability)
9. [Security & Best Practices](#security--best-practices)
10. [Cost Optimization](#cost-optimization)

---

## Introduction

Serverless computing has revolutionized how applications are deployed. With Terraform, you can:

- **Infrastructure as Code**: Version control your entire infrastructure
- **Multi-Cloud**: Deploy to AWS, Azure, GCP from single config
- **Repeatability**: Consistent deployments every time
- **Scalability**: Handle auto-scaling without management
- **Cost Efficiency**: Pay only for execution time
- **Rapid Iteration**: Deploy new functions in seconds

This guide covers building production-grade serverless architectures using Terraform, from simple functions to complex microservices.

### Who Should Read This Guide

- **DevOps Engineers** building cloud infrastructure
- **Cloud Architects** designing serverless systems
- **Software Engineers** deploying applications
- **Platform Engineers** creating deployment pipelines
- **Infrastructure Specialists** managing cloud resources
- **Automation Professionals** orchestrating deployments

---

## Serverless Architecture Fundamentals

### What is Serverless?

```
Traditional Server → Managed Containers → Serverless Functions
    |                    |                    |
You manage:          Platform manages:    Cloud manages:
├─ Hardware          ├─ OS                ├─ Servers
├─ OS                ├─ Runtime           ├─ OS
├─ Runtime           ├─ Auto-scaling      ├─ Runtime
├─ Scaling           └─ Load balancing    ├─ Auto-scaling
└─ Deployments                            └─ Load balancing
```

### Serverless Benefits

| Benefit | Impact |
|---------|--------|
| **No Infrastructure Management** | 80% less ops overhead |
| **Auto-Scaling** | Handles traffic spikes |
| **Pay Per Use** | 60-70% cost reduction |
| **Rapid Deployment** | Function → Production in seconds |
| **Built-in Monitoring** | Native CloudWatch/Stackdriver |
| **High Availability** | Auto-distributed across AZs |

### When to Use Serverless

**Perfect for:**
- ✅ Event-driven workloads (IoT, webhooks, queues)
- ✅ Microservices and APIs
- ✅ Scheduled tasks and cron jobs
- ✅ Real-time data processing
- ✅ Backend for mobile apps
- ✅ Lightweight web applications

**Not ideal for:**
- ❌ Long-running processes (>15 minutes)
- ❌ Complex stateful applications
- ❌ Massive continuous computation
- ❌ Real-time streaming (usually)

### Serverless Providers

| Provider | Platform | Free Tier | Cold Start |
|----------|----------|-----------|-----------|
| **AWS** | Lambda | 1M requests/month | 100-300ms |
| **Google** | Cloud Functions | 2M requests/month | 50-200ms |
| **Azure** | Functions | Free tier | 50-150ms |
| **Others** | Netlify, Vercel, etc. | Various | <100ms |

---

## Terraform Essentials

### HCL Basics

```hcl
# Variables
variable "function_name" {
  description = "Lambda function name"
  type        = string
  default     = "my-function"
}

# Resources
resource "aws_lambda_function" "example" {
  filename      = "lambda.zip"
  function_name = var.function_name
  role          = aws_iam_role.lambda_role.arn
  handler       = "index.handler"
  runtime       = "python3.9"
}

# Outputs
output "function_arn" {
  value = aws_lambda_function.example.arn
}

# Data sources
data "aws_iam_policy_document" "assume_role" {
  statement {
    actions = ["sts:AssumeRole"]
    principals {
      type        = "Service"
      identifiers = ["lambda.amazonaws.com"]
    }
  }
}
```

### Terraform Workflow

```
Write → Plan → Apply → Destroy

1. Write: Define infrastructure in HCL
2. Plan: Preview changes (terraform plan)
3. Apply: Execute changes (terraform apply)
4. Destroy: Clean up (terraform destroy)
```

### Directory Structure

```
terraform/
├── main.tf              # Main resource definitions
├── variables.tf         # Input variables
├── outputs.tf           # Output values
├── terraform.tfvars     # Variable values
├── lambda/
│   ├── function.py      # Python code
│   └── requirements.txt  # Dependencies
├── modules/
│   ├── api/
│   ├── database/
│   └── monitoring/
└── environments/
    ├── dev.tfvars
    ├── staging.tfvars
    └── prod.tfvars
```

---

## Getting Started

### Prerequisites

```bash
# Install Terraform
brew install terraform

# Verify installation
terraform --version

# Install AWS CLI
brew install awscli

# Configure AWS credentials
aws configure
```

### Project Setup

```bash
# Create project directory
mkdir serverless-terraform
cd serverless-terraform

# Initialize Terraform
terraform init

# Verify configuration
terraform validate

# Format HCL
terraform fmt
```

### AWS Credentials Configuration

```bash
# Option 1: AWS CLI
aws configure
# Enter: Access Key, Secret Key, Region, Output format

# Option 2: Environment Variables
export AWS_ACCESS_KEY_ID="xxx"
export AWS_SECRET_ACCESS_KEY="yyy"
export AWS_DEFAULT_REGION="us-east-1"

# Option 3: Terraform Variables
terraform apply \
  -var="aws_access_key=xxx" \
  -var="aws_secret_key=yyy"
```

---

## Deploying to AWS Lambda

### Simple Lambda Function

```hcl
# main.tf

provider "aws" {
  region = "us-east-1"
}

# IAM role for Lambda
resource "aws_iam_role" "lambda_role" {
  name = "lambda-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

# Attach basic execution policy
resource "aws_iam_role_policy_attachment" "lambda_basic_execution" {
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
  role       = aws_iam_role.lambda_role.name
}

# Lambda function
resource "aws_lambda_function" "hello_world" {
  filename      = "lambda.zip"
  function_name = "hello-world"
  role          = aws_iam_role.lambda_role.arn
  handler       = "index.handler"
  runtime       = "python3.9"

  environment {
    variables = {
      ENVIRONMENT = "production"
    }
  }

  timeout = 30
  memory_size = 256
}

# Output
output "lambda_arn" {
  value = aws_lambda_function.hello_world.arn
}

output "lambda_name" {
  value = aws_lambda_function.hello_world.function_name
}
```

### Lambda with API Gateway

```hcl
# Create API Gateway
resource "aws_apigatewayv2_api" "lambda_api" {
  name            = "lambda-api"
  protocol_type   = "HTTP"
  target          = aws_lambda_function.hello_world.arn
}

# Lambda permission for API Gateway
resource "aws_lambda_permission" "api_gateway" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.hello_world.function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${aws_apigatewayv2_api.lambda_api.execution_arn}/*/*"
}

# Output API endpoint
output "api_endpoint" {
  value = aws_apigatewayv2_api.lambda_api.api_endpoint
}
```

### Lambda with S3 Trigger

```hcl
# S3 bucket
resource "aws_s3_bucket" "upload_bucket" {
  bucket = "my-upload-bucket-${data.aws_caller_identity.current.account_id}"
}

# Lambda permission for S3
resource "aws_lambda_permission" "s3_trigger" {
  statement_id  = "AllowExecutionFromS3"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.hello_world.function_name
  principal     = "s3.amazonaws.com"
  source_arn    = aws_s3_bucket.upload_bucket.arn
}

# S3 event notification
resource "aws_s3_bucket_notification" "lambda_trigger" {
  bucket = aws_s3_bucket.upload_bucket.id

  lambda_function {
    lambda_function_arn = aws_lambda_function.hello_world.arn
    events              = ["s3:ObjectCreated:*"]
  }

  depends_on = [aws_lambda_permission.s3_trigger]
}
```

### Lambda with Layers

```hcl
# Create Lambda layer for dependencies
resource "aws_lambda_layer_version" "dependencies" {
  layer_name          = "python-dependencies"
  s3_bucket           = aws_s3_bucket.lambda_layers.id
  s3_key              = "dependencies.zip"
  compatible_runtimes = ["python3.9"]

  source_code_hash = filebase64sha256("dependencies.zip")
}

# Lambda function using layer
resource "aws_lambda_function" "with_dependencies" {
  filename      = "lambda.zip"
  function_name = "my-function"
  role          = aws_iam_role.lambda_role.arn
  handler       = "index.handler"
  runtime       = "python3.9"

  layers = [aws_lambda_layer_version.dependencies.arn]
}
```

---

## Multi-Cloud Deployment

### AWS Lambda Module

```hcl
# modules/aws_lambda/main.tf

variable "function_name" {
  type = string
}

variable "runtime" {
  type    = string
  default = "python3.9"
}

variable "handler" {
  type    = string
  default = "index.handler"
}

variable "zip_file" {
  type = string
}

resource "aws_iam_role" "lambda_role" {
  name = "${var.function_name}-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "basic_execution" {
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
  role       = aws_iam_role.lambda_role.name
}

resource "aws_lambda_function" "function" {
  filename      = var.zip_file
  function_name = var.function_name
  role          = aws_iam_role.lambda_role.arn
  handler       = var.handler
  runtime       = var.runtime
}

output "function_arn" {
  value = aws_lambda_function.function.arn
}

output "function_name" {
  value = aws_lambda_function.function.function_name
}
```

### Google Cloud Functions Module

```hcl
# modules/gcp_function/main.tf

variable "function_name" {
  type = string
}

variable "runtime" {
  type    = string
  default = "python39"
}

variable "source_code" {
  type = string
}

resource "google_cloudfunctions_function" "function" {
  name        = var.function_name
  runtime     = var.runtime
  source_repository {
    url = var.source_code
  }

  entry_point = "hello_world"

  environment_variables = {
    ENVIRONMENT = "production"
  }
}

output "function_url" {
  value = google_cloudfunctions_function.function.https_trigger_url
}
```

### Azure Functions Module

```hcl
# modules/azure_function/main.tf

variable "function_name" {
  type = string
}

variable "resource_group_name" {
  type = string
}

resource "azurerm_storage_account" "function_storage" {
  name                     = "${replace(var.function_name, "-", "")}storage"
  resource_group_name      = var.resource_group_name
  location                 = "eastus"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_app_service_plan" "function_plan" {
  name                = "${var.function_name}-plan"
  location            = "eastus"
  resource_group_name = var.resource_group_name
  kind                = "FunctionApp"
  reserved            = true

  sku {
    tier = "Dynamic"
    size = "Y1"
  }
}

resource "azurerm_function_app" "function" {
  name                       = var.function_name
  location                   = "eastus"
  resource_group_name        = var.resource_group_name
  app_service_plan_id        = azurerm_app_service_plan.function_plan.id
  storage_account_name       = azurerm_storage_account.function_storage.name
  storage_account_access_key = azurerm_storage_account.function_storage.primary_access_key
  os_type                    = "linux"
  runtime_stack              = "python|3.9"
  version                    = "~4"
}
```

### Multi-Cloud Main Configuration

```hcl
# main.tf - Deploy same function to multiple clouds

provider "aws" {
  region = var.aws_region
}

provider "google" {
  project = var.gcp_project
  region  = var.gcp_region
}

provider "azurerm" {
  features {}
}

# AWS Lambda
module "aws_function" {
  source = "./modules/aws_lambda"

  function_name = var.function_name
  runtime       = "python3.9"
  handler       = "index.handler"
  zip_file      = "lambda.zip"
}

# Google Cloud Function
module "gcp_function" {
  source = "./modules/gcp_function"

  function_name = var.function_name
  runtime       = "python39"
  source_code   = var.gcp_source_repo
}

# Azure Function
module "azure_function" {
  source = "./modules/azure_function"

  function_name       = var.function_name
  resource_group_name = azurerm_resource_group.main.name
}

# Outputs
output "endpoints" {
  value = {
    aws   = module.aws_function.function_arn
    gcp   = module.gcp_function.function_url
    azure = module.azure_function.function_url
  }
}
```

---

## Advanced Infrastructure Patterns

### Pattern 1: Microservices Architecture

```hcl
# Microservices with shared database and message queue

# RDS Database
resource "aws_db_instance" "main" {
  identifier            = "microservices-db"
  allocated_storage    = 20
  engine               = "postgres"
  engine_version       = "13.7"
  instance_class       = "db.t3.micro"
  db_name              = "microservices"
  username             = "admin"
  password             = random_password.db_password.result
  skip_final_snapshot  = true
}

# SQS Queue
resource "aws_sqs_queue" "events" {
  name                       = "microservices-events"
  visibility_timeout_seconds = 300
  message_retention_seconds  = 86400
}

# User Service Lambda
module "user_service" {
  source = "./modules/aws_lambda"

  function_name = "user-service"
  zip_file      = "services/user-service.zip"

  environment_variables = {
    DB_HOST = aws_db_instance.main.endpoint
    DB_NAME = aws_db_instance.main.name
  }
}

# Order Service Lambda
module "order_service" {
  source = "./modules/aws_lambda"

  function_name = "order-service"
  zip_file      = "services/order-service.zip"

  environment_variables = {
    DB_HOST     = aws_db_instance.main.endpoint
    QUEUE_URL   = aws_sqs_queue.events.url
  }
}

# API Gateway
resource "aws_apigatewayv2_api" "api" {
  name          = "microservices-api"
  protocol_type = "HTTP"
}

# API Routes
resource "aws_apigatewayv2_integration" "user_integration" {
  api_id           = aws_apigatewayv2_api.api.id
  integration_type = "AWS_PROXY"
  integration_method = "POST"
  integration_uri  = module.user_service.function_arn
}

resource "aws_apigatewayv2_route" "user_route" {
  api_id    = aws_apigatewayv2_api.api.id
  route_key = "GET /users/{id}"
  target    = "integrations/${aws_apigatewayv2_integration.user_integration.id}"
}
```

### Pattern 2: Event-Driven Architecture

```hcl
# Event sourcing with DynamoDB and EventBridge

# DynamoDB Event Store
resource "aws_dynamodb_table" "events" {
  name           = "event-store"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "aggregate_id"
  range_key      = "timestamp"

  attribute {
    name = "aggregate_id"
    type = "S"
  }

  attribute {
    name = "timestamp"
    type = "N"
  }
}

# EventBridge Rule
resource "aws_cloudwatch_event_rule" "order_events" {
  name        = "order-events"
  description = "Capture order events"

  event_pattern = jsonencode({
    source      = ["order.service"]
    detail-type = ["Order Placed"]
  })
}

# Event Processor Lambda
module "event_processor" {
  source = "./modules/aws_lambda"

  function_name = "event-processor"
  zip_file      = "event-processor.zip"

  environment_variables = {
    TABLE_NAME = aws_dynamodb_table.events.name
  }
}

# Target Lambda
resource "aws_cloudwatch_event_target" "lambda" {
  rule      = aws_cloudwatch_event_rule.order_events.name
  target_id = "OrderEventProcessor"
  arn       = module.event_processor.function_arn
}

# Lambda permission
resource "aws_lambda_permission" "eventbridge" {
  statement_id  = "AllowExecutionFromEventBridge"
  action        = "lambda:InvokeFunction"
  function_name = module.event_processor.function_name
  principal     = "events.amazonaws.com"
  source_arn    = aws_cloudwatch_event_rule.order_events.arn
}
```

### Pattern 3: CI/CD Deployment Pipeline

```hcl
# CodePipeline for automated deployment

resource "aws_s3_bucket" "artifact_store" {
  bucket = "pipeline-artifacts-${data.aws_caller_identity.current.account_id}"
}

resource "aws_codepipeline" "pipeline" {
  name     = "serverless-pipeline"
  role_arn = aws_iam_role.codepipeline_role.arn

  artifact_store {
    location = aws_s3_bucket.artifact_store.bucket
    type     = "S3"
  }

  stage {
    name = "Source"

    action {
      name             = "SourceAction"
      category         = "Source"
      owner            = "GitHub"
      provider         = "GitHub"
      version          = "1"
      output_artifacts = ["source_output"]

      configuration = {
        Owner  = var.github_owner
        Repo   = var.github_repo
        Branch = "main"
      }
    }
  }

  stage {
    name = "Build"

    action {
      name             = "BuildAction"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      version          = "1"
      input_artifacts  = ["source_output"]
      output_artifacts = ["build_output"]

      configuration = {
        ProjectName = aws_codebuild_project.build.name
      }
    }
  }

  stage {
    name = "Deploy"

    action {
      name            = "DeployAction"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "CloudFormation"
      version         = "1"
      input_artifacts = ["build_output"]

      configuration = {
        ActionMode     = "CREATE_UPDATE"
        StackName      = "serverless-stack"
        FileName       = "packaged.yaml"
        Capabilities   = "CAPABILITY_IAM,CAPABILITY_AUTO_EXPAND"
      }
    }
  }
}
```

---

## Monitoring & Observability

### CloudWatch Integration

```hcl
# CloudWatch Log Group
resource "aws_cloudwatch_log_group" "lambda_logs" {
  name              = "/aws/lambda/${aws_lambda_function.hello_world.function_name}"
  retention_in_days = 14
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "lambda_errors" {
  alarm_name          = "lambda-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  period              = 300
  statistic           = "Sum"
  threshold           = 1
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    FunctionName = aws_lambda_function.hello_world.function_name
  }
}

# Duration alarm
resource "aws_cloudwatch_metric_alarm" "lambda_duration" {
  alarm_name          = "lambda-high-duration"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "Duration"
  namespace           = "AWS/Lambda"
  period              = 300
  statistic           = "Average"
  threshold           = 10000  # 10 seconds
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    FunctionName = aws_lambda_function.hello_world.function_name
  }
}

# Custom dashboard
resource "aws_cloudwatch_dashboard" "lambda_dashboard" {
  dashboard_name = "lambda-monitoring"

  dashboard_body = jsonencode({
    widgets = [
      {
        type = "metric"
        properties = {
          metrics = [
            ["AWS/Lambda", "Invocations", { stat = "Sum" }],
            [".", "Errors", { stat = "Sum" }],
            [".", "Duration", { stat = "Average" }],
          ]
          period = 300
          stat   = "Average"
          region = var.aws_region
          title  = "Lambda Metrics"
        }
      }
    ]
  })
}
```

### X-Ray Tracing

```hcl
# Enable X-Ray tracing
resource "aws_lambda_function" "traced" {
  # ... other configuration ...
  
  tracing_config {
    mode = "Active"
  }
}

# X-Ray IAM policy
resource "aws_iam_role_policy" "xray_write_access" {
  name   = "xray-write-access"
  role   = aws_iam_role.lambda_role.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "xray:PutTraceSegments",
        "xray:PutTelemetryRecords"
      ]
      Resource = "*"
    }]
  })
}
```

---

## Security & Best Practices

### IAM Role with Least Privilege

```hcl
# Minimal IAM role
resource "aws_iam_role" "lambda_role" {
  name = "lambda-minimal-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

# Specific permissions only
resource "aws_iam_role_policy" "specific_s3_access" {
  name   = "s3-specific-access"
  role   = aws_iam_role.lambda_role.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "s3:GetObject",
        "s3:PutObject"
      ]
      Resource = "${aws_s3_bucket.data.arn}/input/*"
    }]
  })
}
```

### Secrets Management

```hcl
# Store secrets in Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "lambda/database/password"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = random_password.db_password.result
}

# Grant Lambda access
resource "aws_iam_role_policy" "secrets_access" {
  name   = "secrets-access"
  role   = aws_iam_role.lambda_role.id
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "secretsmanager:GetSecretValue"
      ]
      Resource = aws_secretsmanager_secret.db_password.arn
    }]
  })
}
```

### VPC Configuration

```hcl
# Lambda in VPC for database access
resource "aws_lambda_function" "vpc_lambda" {
  # ... other configuration ...

  vpc_config {
    subnet_ids         = aws_subnet.private[*].id
    security_group_ids = [aws_security_group.lambda.id]
  }
}

# Security group
resource "aws_security_group" "lambda" {
  name        = "lambda-sg"
  description = "Security group for Lambda"
  vpc_id      = aws_vpc.main.id

  egress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## Cost Optimization

### Reserved Concurrency

```hcl
# Reserve concurrency to control costs
resource "aws_lambda_provisioned_concurrency_config" "example" {
  function_name                     = aws_lambda_function.hello_world.function_name
  provisioned_concurrent_executions = 10
  qualifier                         = aws_lambda_alias.live.name
}
```

### Cost Monitoring

```hcl
# Budget alert
resource "aws_budgets_budget" "lambda_budget" {
  name              = "lambda-monthly-budget"
  budget_type       = "COST"
  limit_amount      = "100"
  limit_unit        = "USD"
  time_period_start = "2024-01-01_00:00:00Z"
  time_period_end   = "2087-06-15_00:00:00Z"
  time_unit         = "MONTHLY"

  cost_filters = {
    "SERVICE" = ["AWS Lambda"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    notification_type          = "ACTUAL"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_channel_arns  = [aws_sns_topic.billing_alerts.arn]
  }
}
```

### Memory Optimization

```hcl
# Right-size memory allocation
resource "aws_lambda_function" "optimized" {
  # ... other configuration ...
  
  memory_size = 256  # 128, 256, 512, 1024, etc.
  timeout     = 30   # seconds

  # Note: CPU scales with memory
  # 128 MB = 0.083 vCPU
  # 256 MB = 0.167 vCPU
  # 1024 MB = 0.67 vCPU
}
```

---

## Best Practices Summary

**Do's ✅**
- ✅ Use environment variables for configuration
- ✅ Implement proper logging and monitoring
- ✅ Use IAM least privilege principle
- ✅ Test locally with SAM or LocalStack
- ✅ Version your infrastructure
- ✅ Use modules for reusability
- ✅ Implement secrets management
- ✅ Set appropriate timeouts and memory

**Don'ts ❌**
- ❌ Hard-code secrets or API keys
- ❌ Use overly permissive IAM policies
- ❌ Ignore monitoring and logging
- ❌ Deploy to production without testing
- ❌ Forget to set function timeouts
- ❌ Ignore cold start implications
- ❌ Use state drift (always use Terraform)
- ❌ Deploy without version control

---

## Real-World Example: Complete Serverless Application

```hcl
# Complete todo app with API, database, and monitoring

# Provider
provider "aws" {
  region = var.aws_region
}

# Variables
variable "app_name" {
  default = "serverless-todo-app"
}

variable "environment" {
  default = "production"
}

# DynamoDB
resource "aws_dynamodb_table" "todos" {
  name           = "${var.app_name}-todos"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "userId"
  range_key      = "todoId"

  attribute {
    name = "userId"
    type = "S"
  }

  attribute {
    name = "todoId"
    type = "S"
  }
}

# Lambda functions
module "create_todo" {
  source = "./modules/aws_lambda"
  function_name = "${var.app_name}-create-todo"
  zip_file = "functions/create-todo.zip"

  environment_variables = {
    TABLE_NAME = aws_dynamodb_table.todos.name
  }
}

module "list_todos" {
  source = "./modules/aws_lambda"
  function_name = "${var.app_name}-list-todos"
  zip_file = "functions/list-todos.zip"

  environment_variables = {
    TABLE_NAME = aws_dynamodb_table.todos.name
  }
}

module "delete_todo" {
  source = "./modules/aws_lambda"
  function_name = "${var.app_name}-delete-todo"
  zip_file = "functions/delete-todo.zip"

  environment_variables = {
    TABLE_NAME = aws_dynamodb_table.todos.name
  }
}

# API Gateway
resource "aws_apigatewayv2_api" "api" {
  name          = "${var.app_name}-api"
  protocol_type = "HTTP"
  cors_configuration {
    allow_origins = ["*"]
    allow_methods = ["GET", "POST", "PUT", "DELETE"]
    allow_headers = ["*"]
  }
}

# Routes
resource "aws_apigatewayv2_route" "create" {
  api_id    = aws_apigatewayv2_api.api.id
  route_key = "POST /todos"
  target    = "integrations/${aws_apigatewayv2_integration.create_integration.id}"
}

# Integration and permissions...
# (Similar pattern for list and delete routes)

# Monitoring
resource "aws_cloudwatch_metric_alarm" "api_errors" {
  alarm_name          = "${var.app_name}-api-errors"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "4XXError"
  namespace           = "AWS/ApiGateway"
  period              = 300
  statistic           = "Sum"
  threshold           = 10
  alarm_actions       = [aws_sns_topic.alerts.arn]
}

# Output
output "api_endpoint" {
  value = aws_apigatewayv2_api.api.api_endpoint
}
```

---

## Conclusion

Terraform enables you to:
✅ Define serverless infrastructure as code
✅ Deploy across multiple clouds consistently
✅ Manage infrastructure versions
✅ Automate deployments
✅ Scale from single function to enterprise
✅ Maintain compliance and security

**Next Steps:**
1. Install Terraform
2. Set up AWS credentials
3. Deploy first Lambda function
4. Add API Gateway
5. Implement monitoring
6. Optimize for production

---

## Resources

- **Terraform Docs**: https://www.terraform.io/docs
- **AWS Provider**: https://registry.terraform.io/providers/hashicorp/aws
- **AWS Lambda Guide**: https://docs.aws.amazon.com/lambda/
- **Terraform Registry**: https://registry.terraform.io/
- **AWS SAM**: https://aws.amazon.com/serverless/sam/

---

## Next Steps for Enterprise

1. **State Management**: Use S3 backend with locking
2. **Modules**: Create reusable infrastructure modules
3. **Environments**: Separate dev, staging, production
4. **Testing**: Implement infrastructure testing with Terratest
5. **CI/CD**: Automate Terraform with GitHub Actions or GitLab CI
6. **Cost Management**: Monitor and optimize spending
7. **Security**: Implement scanning and compliance checks

---

## About Rework Digital

This guide was created by **Rework Digital** - Resources Department for automation professionals building innovative solutions.

**Resources Department Contact:** resource@reworkdigital.io  
**Follow us on GitHub:** https://github.com/Reworkdigital-io

---

*Last Updated: 2026-04-10*  
*Version: 1.0*
