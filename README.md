# EC2 CPU & Network Monitoring using EventBridge + Lambda + SNS (No CloudWatch Agent)

## Overview

This solution monitors multiple Amazon EC2 instances for:

- CPU Utilization
- Network In
- Network Out

If any configured threshold is exceeded, an email notification is sent via Amazon SNS containing the instance details and the metric that triggered the alert.

This solution uses native AWS CloudWatch metrics and **does not require the CloudWatch Agent** to be installed on the EC2 instances.

---

## Architecture

```text
                AWS CloudWatch Metrics
                        │
                        │
                EventBridge Scheduler
                  (Runs every 5 minutes)
                        │
                        ▼
                   AWS Lambda
        (Checks CPU & Network Metrics)
                        │
                        ▼
                    Amazon SNS
                        │
                        ▼
                 Email Notification
```

---

# Features

- Monitor multiple EC2 instances using a single Lambda function
- Individual thresholds per EC2 instance
- CPU Utilization monitoring
- Network In monitoring
- Network Out monitoring
- Email alerts through Amazon SNS
- No CloudWatch Agent installation required
- Easily scalable to monitor dozens of EC2 instances

---

# Prerequisites

- AWS Account
- Amazon EC2 instances
- CloudWatch metrics enabled (default for EC2)
- IAM permissions to create Lambda, EventBridge Scheduler, SNS, and IAM roles

---

# Solution Components

| Service | Purpose |
|----------|---------|
| Amazon CloudWatch | Provides CPU and Network metrics |
| Amazon EventBridge Scheduler | Invokes Lambda every 5 minutes |
| AWS Lambda | Reads CloudWatch metrics and evaluates thresholds |
| Amazon SNS | Sends email notifications |

---

# Step 1 – Create an SNS Topic

Navigate to:

```
Amazon SNS
```

Create a new topic.

Example:

```
EC2-CPU-Net-Alert
```

Create an Email Subscription.

Protocol:

```
Email
```

Enter the recipient email address and confirm the subscription from the received email.

---

# Step 2 – Create an IAM Role for Lambda

Create a new IAM Role and attach the following managed policies:

- AWSLambdaBasicExecutionRole
- CloudWatchReadOnlyAccess
- AmazonEC2ReadOnlyAccess
- AmazonSNSFullAccess

---

# Step 3 – Create the Lambda Function

Runtime:

```
Python 3.12
```

Configuration:

| Setting | Value |
|----------|-------|
| Memory | 256 MB |
| Timeout | 30 Seconds |

Deploy the monitoring code.

---

# Step 4 – Create an EventBridge Scheduler

Navigate to:

```
Amazon EventBridge
    └── Scheduler
            └── Create Schedule
```

Configuration:

| Option | Value |
|---------|-------|
| Schedule Type | Recurring |
| Frequency | Every 5 Minutes |
| Target | Lambda Function |
| Payload | Constant JSON |

---

# Step 5 – Configure the JSON Payload

Example:

```json
{
  "region": "us-east-1",
  "instances": [
    {
      "instance_id": "i-xxxxxxxxxxxxxxxx1",
      "instance_name": "Production API",
      "cpu_limit": 75,
      "network_in_limit_mb": 500,
      "network_out_limit_mb": 500
    },
    {
      "instance_id": "i-xxxxxxxxxxxxxxxx2",
      "instance_name": "FTP Server",
      "cpu_limit": 80,
      "network_in_limit_mb": 700,
      "network_out_limit_mb": 700
    }
  ],
  "sns_topic_arn": "arn:aws:sns:us-east-1:123456789012:EC2-CPU-Net-Alert"
}
```

You can configure any number of EC2 instances.

Examples:

- 5 Servers
- 10 Servers
- 15 Servers
- 50+ Servers

No Lambda code changes are required.

---

# Lambda Processing Flow

Every 5 minutes, the Lambda function performs the following steps:

```
Start

↓

Read Event JSON

↓

Loop through each configured EC2 instance

↓

Read CPUUtilization metric

↓

Read NetworkIn metric

↓

Read NetworkOut metric

↓

Compare with configured thresholds

↓

Threshold exceeded?

      Yes ─────────► Publish SNS Email

      No ──────────► Continue

↓

Next EC2 Instance

↓

Finish
```

---

# Example Scenario

Configured Servers:

| Server | CPU Threshold |
|----------|--------------|
| API01 | 75% |
| API02 | 75% |
| FTP01 | 80% |
| WEB01 | 90% |
| Others | Configured individually |

Current Status

| Server | CPU Usage | Result |
|----------|-----------|--------|
| API01 | 42% | OK |
| API02 | 85% | Alert |
| FTP01 | 39% | OK |
| WEB01 | 92% | Alert |
| Remaining Servers | Below Threshold | OK |

Lambda checks all configured instances.

SNS sends email notifications only for:

- API02
- WEB01

No emails are sent for healthy instances.

---

# Sample Email Alert

## Subject

```
🚨 EC2 High Usage Alert | Production API
```

## Body

```
🚨 EC2 HIGH USAGE ALERT

Instance Name : Production API

Instance ID   : i-xxxxxxxxxxxxxxxx

Region        : us-east-1

CPU Usage     : 86%

CPU Limit     : 75%

Network In    : 610 MB

Limit         : 500 MB

Network Out   : 225 MB

Limit         : 500 MB

Alert Time    : 2026-07-01 11:20 UTC

Reason

CPU Usage exceeded the configured threshold.
```

---

# Adding a New EC2 Instance

To monitor a new server, simply add another object to the JSON payload.

Example:

```json
{
  "instance_id": "i-xxxxxxxx",
  "instance_name": "Mail Server",
  "cpu_limit": 75,
  "network_in_limit_mb": 500,
  "network_out_limit_mb": 500
}
```

No Lambda code modifications are required.

---

# Monitoring Logic

For every configured EC2 instance:

- Read CPU Utilization
- Read Network In
- Read Network Out
- Compare metrics against configured thresholds
- Send an SNS email only if one or more thresholds are exceeded

---

# Advantages

- ✅ Single Lambda function monitors all EC2 instances
- ✅ Single EventBridge Scheduler
- ✅ No CloudWatch Agent installation required
- ✅ No software installation on EC2 instances
- ✅ Individual thresholds for each server
- ✅ Email notifications only for affected instances
- ✅ Simple JSON-based configuration
- ✅ Easily scalable to dozens of EC2 instances
- ✅ Low operational overhead
- ✅ Fully serverless solution

---
