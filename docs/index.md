# Getting Hands on with Amazon GuardDuty

## Overview
This repository walks you through a scenario covering threat detection and remediation using <a href="https://aws.amazon.com/guardduty/" target="_blank">Amazon GuardDuty</a>; a managed threat detection service. The scenario simulates an attack that spans a few threat vectors, representing just a small sample of the threats that GuardDuty is able to detect.

In addition, you will look at how to view and analyze GuardDuty findings, how to send alerts based on the findings, and, finally, how to remediate findings.

The <a href="https://aws.amazon.com/cloudformation/" target="_blank">AWS CloudFormation</a> template used for this scenario builds out the resources needed to simulate attacks and auto-remediate the GuardDuty findings using a combination of <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html" target="_blank">CloudWatch Event Rules</a> and <a href="https://aws.amazon.com/lambda/" target="_blank">AWS Lambda Functions</a>.  

* **Level**: 300
* **Duration**: 1-2 hours
* **<a href="https://awssecworkshops.com/getting-started/" target="_blank">Prerequisites</a>**: AWS Account, Admin IAM User, AWS CLI
* **<a href="https://www.nist.gov/cyberframework/online-learning/components-framework" target="_blank">CSF Functions</a>**: Protect, Detect, Respond
* **<a href="https://d0.awsstatic.com/whitepapers/AWS_CAF_Security_Perspective.pdf" target="_blank">CAF Components</a>**: Preventative, Detective, Responsive
* **AWS Services**: Amazon CloudWatch, Amazon GuardDuty, AWS CloudTrail, AWS Lambda, Security Groups, Amazon SNS

## Region
Please use the **us-west-2 (Oregon)** region for this workshop.

## Setup

[Overview and Environment Setup](./setup.md)

## Modules

This workshop is broken up into the three scenarios below:

1. [Compromised EC2 Instance](./scenario1/index.md)
> We will detect and remediate a compromised host using Amazon GuardDuty, Amazon CloudWatch Event Rules and AWS Lambda.

2. [Compromised IAM Credentials](./scenario2/index.md)
> We will identify a malicious host attempting to make API calls inside our AWS environment and remediate the host manually.

3. [IAM Role Exfiltration](./scenario3/index.md)
> We will identity that credentials have been exfiltrated from a compromised host and identify the API calls taken from an external host using those credentials. In addition we will review the automated remediation taken through AWS Lambda.

4. [Compromised S3 Bucket](./scenario4/index.md)
> We will identity a malicious host attempting to make S3 API calls inside our AWS environment and remediate the host manually.

## Cleanup

[Summary and Environment Cleanup](./summary.md)
