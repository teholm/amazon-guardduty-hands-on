# Getting Started with Amazon GuardDuty

This GitHub repository walks you through a scenario covering threat detection and remediation using Amazon GuardDuty. The scenario covers an attack that simulates a few threat vectors which represents just a small sample of the threats that GuardDuty is able to detect. In addition, we look at how to view the GuardDuty findings, how to alert based on the findings, and finally how to remediate findings. The CloudFormation template builds out the assets used to simulate the attacks and the instructions are provided on how to analyze and remediate the findings.

[What is Created](#created)  
[Getting Started](#started) 

## What is Created? <a name="created"></>
The CloudFormation template will create the following resources:
  * Three [Amazon EC2](https://aws.amazon.com/ec2/) Instances (all using a t2.micro instance type):
    * Two Instances that contain the name “*Compromised Instance*” 
    * One instance that contains the name “*Malicious Instance*”
  * [AWS IAM Role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) For EC2 (which will be attached to two of the instances created)
  * One [Amazon SNS Topic](https://docs.aws.amazon.com/sns/latest/dg/GettingStarted.html) with an subscription for the e-mail address you will enter in the parameters when you run the CloudFormation template
  * Three [AWS CloudWatch Event](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html) rules:
    * Rule named “*GuardDuty-Event-EC2-MaliciousIPCaller*” with two triggers
      * Trigger to publish to the SNS Topic 
      * Trigger to run a Lambda function
    * Rule named “*GuardDuty-Event-IAMUser-InstanceCredentialExfiltration*” with two triggers
      * Trigger to publish to the SNS Topic 
      * Trigger to run a Lambda function
    * Rule named “*GuardDuty-Event-IAMUser-MaliciousIPCaller*” with one trigger
      * Trigger to publish to the SNS Topic 
  * Two [AWS Lambda](https://aws.amazon.com/lambda/) functions
    * Function named “*GuardDuty-Example-Remediation-EC2MaliciousIPCaller*” to remediate the EC2 instance compromise by removing the instance from the current security group and add it to one with no ingress or egress rules
    * Function named “*GuardDuty-Example-Remediation-InstanceCredentialExfiltration*” to remediate the IAM credential compromise by put a revoke policy on the IAM Role from where the credentials were provided.  
  * [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html) values with the IAM temporary security credentials details.

## Getting started – Just Two Clicks <a name="started"></>

The CloudFormation template works whether GuardDuty is enabled or not (you just need to indicate the state of GuardDuty in the CloudFormation parameters section). If you would like to enable GuardDuty yourself before running the CloudFormation script then you can follow the instructions below. If you don’t want to enable GuardDuty yourself you can skip to next section titled **Deploy the Solution using AWS CloudFormation**.

It's extremely easy to enable GuardDuty and get stared so it won't take very long to enable it. Also there are no pre-requisites for turning it on and nothing else you need to do. 

Follow these steps to enable GuardDuty
1. First Click: Navigate to the GuardDuty console in the region you want to run this scenario in and then click **Get Started**.

![Get Started](images/screenshot1.png "Get Started")

2. Second Click: On the next screen click **Enable GuardDuty**.

![Enable GuardDuty](images/screenshot2.png "Enable GuardDuty")

3. That is all you need to do. There are no prerequisites you need to set up, no agents to install, and no hardware to configure. From the moment you enable GuardDuty it is analyzing all of the VPC Flow Logs, CloudTrail logs, and DNS Logs in that region. There are some findings that require that a baseline is established but most findings will be available from the moment you enable GuardDuty. Also, it can take a few minutes from the time the information about a threat is entered into one of the log files and the time GuardDuty is able to display the finding. Regardless of the number of VPCs, IAM users, or other AWS resources there is no impact to your resources since all of the processing is being done outside of your account. 

![GuardDuty Enabled](images/screenshot3.png "GuardDuty Enabled")

Once GuardDuty is enabled you can easily suspend or disable the service with one click (under **Settings** -> **General**). Suspending will pause the service, which stops the billing but keeps your current findings and current baseline analysis. Disabling stops the billing and removes all the existing findings and baseline data.

## Deploy the Solution Using AWS CloudFormation

To initiate the scenario and begin generating GuardDuty findings you need to run the provided CloudFormation template. Given that we will be simulating attacks and doing remediation, it is recommended that you run this CloudFormation template in a test account. The scenario only impacts the resources that the CloudFormation stack creates but it is still a best practice to run these types of attack scenarios in test accounts with no other resources. It makes sense to create a new AWS account strictly for the purpose of security and threat testing when trying out services like GuardDuty. 

1.	Click [this link](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#cstack=sn%7EGuardDutyBlog%7Cturl%7Ehttps://s3-us-west-2.amazonaws.com/lab.gregmcconnel.net/guardduty-blog-cfn-template.yml) to run the CloudFormation template. This will automatically take you to the console to run the template.  You can also just use the template found in this repo (guardduty-blog-cfn-template.yml).
2.	You will need to enter a number of parameters at the beginning. Below is an example:

![Parameters](images/screenshot4.png "Parameters")

3.	Once you have entered your parameters click **Next**, then **Next** again (leave everything on this page at the default), check the IAM creation acknowledgement, and then click **Create**.
4.	The CloudFormation stack creation will take about 5 minutes to complete.

**Note**: The initial findings will begin to show up in GuardDuty about 15 minutes after the CloudFormation stack creation completes. One housekeeping item that needs to be done after you launch the CloudFormation template is to confirm the SNS AWS Notification Subscription. You will receive an e-mail to the e-mail address you entered in the parameters when you ran the CloudFormation script. By confirming the subscription, you will receive e-mails when GuardDuty generates findings.

## Attack scenario 1 – Compromised EC2 Instance

We are simulating an attack scenario so let’s set the scene: After an uneventful yet unnecessarily long commute to work, you have arrived at the office this morning. You have grabbed a cup of coffee, sat down in your cube, opened up your laptop and begin to go through your emails. Soon after you begin though you start receiving emails indicating that GuardDuty has detected new threats. You don’t yet know the extent of the threats but you quickly begin to investigate. Now the good news is that your coworker Alice has already set up some hooks for specific findings so that they will be automatically remediated. 

The first e-mail you receive from GuardDuty indicates that one of your EC2 instances may be compromised:

*"The EC2 instance i-xxxxxxxxx may be compromised and should be investigated.*

### Diagram of the attack and remediation

![Attack Senario 1](images/attack1.png "Attack Scenario 1")

### Browse to the GuardDuty console to investigate

To view the GuardDuty findings:

1. Navigate to the [GuardDuty console](https://console.aws.amazon.com/guardduty) and choose Current in the navigation pane on the left. 
2. Click the ![Refresh](images/refreshicon.png "Refresh") button next to words “*Current Findings*” if there is nothing displayed to refresh the display
3. A finding should show up soon with the text “*EC2 instance i-xxxxxxxx communicating with disallowed IP address.*” 
   * If you would like to see the full list of GuardDuty findings and understand the details of each finding you can view this in the [GuardDuty Documentation](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types.html). 

4. This finding means that one of your EC2 instances is communicating with an IP address that is in a custom threat list (GuardDuty comes with three threat lists and you can add custom threat lists or trusted IP lists yourself) and may indicate that this instance is compromised. Alice has set up a CloudWatch Event Rule for this type of finding. The rule will notify you via SNS and run a Lambda function to isolate instances that GuardDuty indicates may be compromised according to the particular finding type. The Lambda function removes the instance from its current security group and adds it to a security group with no ingress or egress rules. This isolates the instance so no inbound or outbound connections can be made. The instance will not be able to impact any resources in your VPC while the security team investigates.

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
