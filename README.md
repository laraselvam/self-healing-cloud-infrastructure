# Self-Healing Cloud Infrastructure

A serverless AWS project that automatically detects and reboots unhealthy EC2 instances — no manual intervention required.

## Problem

EC2 instances can become unresponsive (status check failures) due to OS-level issues, memory exhaustion, or application crashes. Manually monitoring servers 24/7 and restarting them isn't scalable. This project automates detection and recovery.

## Architecture
EC2 Instance (status check fails)
│
▼
CloudWatch Alarm (StatusCheckFailed_System)
│
▼
SNS Topic (self-healing-alerts)
│
▼
Lambda Function (autoheal-ec2)
│
▼
EC2 Instance Reboot1. 
1.**CloudWatch Alarm** continuously monitors the EC2 instance's system status check.
2. When the check fails, the alarm enters `ALARM` state and publishes a message to an **SNS topic**.
3. The SNS topic triggers a **Lambda function**, passing along the alarm details (including which instance failed).
4. The Lambda function parses the SNS message, dynamically extracts the failing instance's ID, and calls the EC2 API to reboot it.
5. An **email notification** is also sent via SNS so the incident is visible to a human, even though no manual action is needed.

## Tech Stack

- **AWS Lambda** (Python 3, boto3) — reboot logic
- **AWS CloudWatch** — health monitoring and alarming
- **AWS SNS** — event-driven trigger and email alerts
- **AWS EC2** — the monitored compute resource

## Key Design Decision: Dynamic Instance ID

The Lambda function does not hardcode which instance to reboot. Instead, it parses the incoming SNS/CloudWatch alarm payload at runtime to extract the `InstanceId` dimension. This means the same Lambda function can be reused across multiple EC2 instances/alarms without modification.

## Files

- `lambda_function.py` — the reboot logic, triggered by SNS

## Status

Core pipeline is built and tested end-to-end (Lambda deploy verified via test invocation).