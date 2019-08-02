# Getting Hands on with Amazon GuardDuty

This repository walks you through a scenario covering threat detection and remediation using [Amazon GuardDuty](https://aws.amazon.com/guardduty/); a managed threat detection service. The scenario simulates an attack that spans a few threat vectors, representing just a small sample of the threats that GuardDuty is able to detect. 

In addition, you will look at how to view and analyze GuardDuty findings, how to send alerts based on the findings, and, finally, how to remediate findings. 

The [AWS CloudFormation](https://aws.amazon.com/cloudformation/) template used for this scenario builds out the resources needed to simulate attacks and auto-remediate the GuardDuty findings using a combination of [CloudWatch Event Rules](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) and [AWS Lambda Functions](https://aws.amazon.com/lambda/).  


## Region
Please use the **us-west-2 (Oregon)** region for this workshop.

## Setup

[Overview and Environment Setup](./setup.md)

## Modules

This workshop is broken up into the three scenarios below:
 
1. [Compromised EC2 Instance](./scenario1/index.md)
2. [Compromised IAM Credentials](./scenario2/index.md)
3. [IAM Role Exfiltration](./scenario3/index.md)

## Cleanup

[Summary and Environment Cleanup](./summary.md)
