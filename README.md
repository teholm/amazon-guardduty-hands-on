# Getting Started with Amazon GuardDuty

This GitHub repository walks you through a scenario covering threat detection and remediation using Amazon GuardDuty. The scenario covers an attack that simulates a few threat vectors which represents just a small sample of the threats that GuardDuty is able to detect. In addition, we look at how to view the GuardDuty findings, how to alert based on the findings, and finally how to remediate findings. The CloudFormation template builds out the assets used to simulate the attacks and the instructions are provided on how to analyze and remediate the findings.

### Table of Contents

* [What is Created](#created)  
* [Getting Started](#started) 
* [Deploy the Scenario](#deploy) 
* [Attack scenario 1 – Compromised EC2 Instance](#attack1) 
* [Attack Scenario 2 – Compromised IAM Credentials](#attack2)
* [Attack Scenario 3 – IAM Role Credential Exfiltration](#attack3)

## What is Created? <a name="created"/>
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

## Getting started – Just Two Clicks <a name="started"/>

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

## Deploy the Senario Using AWS CloudFormation <a name="deploy"/>

To initiate the scenario and begin generating GuardDuty findings you need to run the provided CloudFormation template. Given that we will be simulating attacks and doing remediation, it is recommended that you run this CloudFormation template in a test account. The scenario only impacts the resources that the CloudFormation stack creates but it is still a best practice to run these types of attack scenarios in test accounts with no other resources. It makes sense to create a new AWS account strictly for the purpose of security and threat testing when trying out services like GuardDuty. 

1.	Click [this link](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#cstack=sn%7EGuardDutyBlog%7Cturl%7Ehttps://s3-us-west-2.amazonaws.com/lab.gregmcconnel.net/guardduty-blog-cfn-template.yml) to run the CloudFormation template. This will automatically take you to the console to run the template.  You can also just use the template found in this repo (guardduty-blog-cfn-template.yml).
2.	You will need to enter a number of parameters at the beginning. Below is an example:

![Parameters](images/screenshot4.png "Parameters")

3.	Once you have entered your parameters click **Next**, then **Next** again (leave everything on this page at the default), check the IAM creation acknowledgement, and then click **Create**.
4.	The CloudFormation stack creation will take about 5 minutes to complete.

**Note**: The initial findings will begin to show up in GuardDuty about 15 minutes after the CloudFormation stack creation completes. One housekeeping item that needs to be done after you launch the CloudFormation template is to confirm the SNS AWS Notification Subscription. You will receive an e-mail to the e-mail address you entered in the parameters when you ran the CloudFormation script. By confirming the subscription, you will receive e-mails when GuardDuty generates findings.

## Attack scenario 1 – Compromised EC2 Instance <a name="attack1"/>

We are simulating an attack scenario so let’s set the scene: After an uneventful yet unnecessarily long commute to work, you have arrived at the office this morning. You have grabbed a cup of coffee, sat down in your cube, opened up your laptop and begin to go through your emails. Soon after you begin though you start receiving emails indicating that GuardDuty has detected new threats. You don’t yet know the extent of the threats but you quickly begin to investigate. Now the good news is that your coworker Alice has already set up some hooks for specific findings so that they will be automatically remediated. 

The first e-mail you receive from GuardDuty indicates that one of your EC2 instances may be compromised:

*"The EC2 instance i-xxxxxxxxx may be compromised and should be investigated"*

### Diagram of the Attack, Detection, and Remediation

![Attack Senario 1](images/attack1.png "Attack Scenario 1")

### Browse to the GuardDuty console to investigate

When using CloudWatch Events to send a finding from GuardDuty to SNS and then to your e-mail address, the default behavior is to send the entire JSON containing all of the threat data. It is also possible to put a transformation on the data before it is sent to SNS to produce a more user-friendly e-mail. That is what was done in this case so in order to get the complete information about the finding you need to check the GuardDuty console. You browse to the GuardDuty console to investigate further.

To view the GuardDuty findings:

1. Navigate to the [GuardDuty console](https://console.aws.amazon.com/guardduty) and choose Current in the navigation pane on the left. 
2. Click the ![Refresh](images/refreshicon.png "Refresh") button next to words “*Current Findings*” if there is nothing displayed to refresh the display
3. A finding should show up soon with the text “*EC2 instance i-xxxxxxxx communicating with disallowed IP address.*” 
   * If you would like to see the full list of GuardDuty findings and understand the details of each finding you can view this in the [GuardDuty Documentation](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types.html). 

![GuardDuty Finding](images/screenshot5.png "GuardDuty Finding")

4. This finding means that one of your EC2 instances is communicating with an IP address that is in a custom threat list (GuardDuty comes with three threat lists and you can add [custom threat lists or trusted IP lists](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_upload_lists.html) yourself) and may indicate that this instance is compromised. Alice has set up a CloudWatch Event Rule for this type of finding. The rule will notify you via SNS and run a Lambda function to isolate instances that GuardDuty indicates may be compromised according to the particular finding type. The Lambda function removes the instance from its current security group and adds it to a security group with no ingress or egress rules. This isolates the instance so no inbound or outbound connections can be made. The instance will not be able to impact any resources in your VPC while the security team investigates.

**Scenario Notes**: 
1.	The EC2 instance indicated by this finding is actually just connecting to an EIP on another instance. The EIP is in a custom threat list. 
2.	An option more suited for a production environment would be to notify the security team and allow them to investigate before any action is taken. A timer could be set to allow investigation before an automated action is taken.  This way a determination as to what action to take could be made based on the results of the investigation of the reported threat.

### View the CloudWatch Event rule for the compromised EC2 instance

Although you can view the GuardDuty findings in the console, most customers will want to receive a notification of new findings and possibly have all of the findings delivered to a central SIEM for analysis and remediation. The recommended process for GuardDuty findings management is to use CloudWatch Events. From there you can trigger SNS to send email, Lambda to take various actions or any number of other steps. This is also the most common pattern for sending the findings to your SIEM. GuardDuty has many partners to help with the centralized management of all your findings from different regions and AWS accounts.  You can use a Master/Member setup for accounts and add up to 1000 members to a central master account. All of the findings for the member accounts will roll up the master account GuardDuty console and the master account CloudWatch Events. Alice used CloudWatch Events to send the e-mail you received about the findings and also to take remediations steps. We will examine the CloudWatch Events console to understand what Alice configured and to make sure the remediation was triggered. 

1.	Navigate to the [CloudWatch console](https://us-east-2.console.aws.amazon.com/cloudwatch/home?) and click on **Rules** under the **Events** section of the left. 
2.	You will see three Rules in the list that were created by the CloudFormation template. All of these begin with the phrase “*GuardDuty-Event*” (you may see other rules depending on if any other rules were created for other purposes in this account)

![CloudWatch Event Rules](images/screenshot6.png "CloudWatch Event Rules")

3.	The rule that Alice configured for this “*compromised instance*” scenario is called “GuardDuty-Event-EC2-MaliciousIPCaller”. **Click on that rule**. 
4.	Under the Targets section you will see two entries, one for a Lambda function and one for an SNS Topic.

![CloudWatch Event Rule](images/screenshot7.png "CloudWatch Event Rule")

5.	The SNS topic entry will send an email to the e-mail address you entered in the parameters for the CloudFormation template. In the Input section you can see that an **input transformer** has been configured. This is to make the e-mail that is sent a little friendlier then a dump of the full JSON from the GuardDuty finding.
6.	The Lambda function is what handle this remediation for this finding. The Lambda function will remove the “compromised instance” from its current security group and add it to one with no ingress or egress rules so that the instance is isolated from the network. If you click on the **Resource name** for the Lambda function, this will take you into the Lambda console for that function.

![Lambda Function](images/screenshot8.png "Lambda Function")

7.	You can scroll down to view the code of this function. You can also click on the Monitoring tab and view the invocation of the function. You should see one invocation Count and no Invocation Errors. 
8.	In order to see that the remediation was applied to the instance, browse to the [EC2 console](https://us-east-2.console.aws.amazon.com/ec2/v2) and click on **Running Instances**. You should see three instances with names that begin with “GuardDuty-Example.” There are two instances named “GuardDuty-Example: Compromised Instance”. Click on each of these and find the one that is in the Security Group with a name that starts with “*GuardDutyBlog-ForensicSecurityGroup-*“. 

![EC2 Instances](images/screenshot9.png "EC2 Instances")

9.	Initially all three of the instances launched by the CloudFormation template were in the Security Group that starts with the name “*GuardDutyBlog-TargetSecurityGroup-*”. The Lambda function removed this one instance from the TargetSecurityGroup and added it to the ForensicsSecurityGroup in order to isolate the instance. 
10.	If you check back in your email you should see another email that came right after that first email regarding the compromised instance. This second email indicates that the remediation was completed. 

*"GuardDuty discovered an EC2 instance (Instance ID: i-xxxxxxxx) that is communicating outbound with an IP Address on a threat list that you uploaded.  All security groups have been removed and it has been isolated. Please follow up with any additional remediation actions"*

## Attack Scenario 2 – Compromised IAM Credentials <a name="attack2"/>

You have completed the examination of this first attack, confirmed it was properly remediated and then sat back to took your first sip of coffee for the day when you notice additional e-mails about new findings. The first of the new findings indicates that an API call was made using IAM credentials from your AWS account from a malicious IP address. Similar to before with the compromised instance you can browse to the GuardDuty console to view the finding.

**Scenario Note**: None of you IAM credentials have actually been compromised or exposed in any way. The finding is the result of an EC2 instance using an IAM Role for EC2 and with an EIP that is in the custom threat list making API calls on your behalf. 

### Browse to the GuardDuty Console to investigate

Again you decide to browse to the GuardDuty console to further investigate.

To view the findings:
1. Navigate to the [GuardDuty console](https://console.aws.amazon.com/guardduty) and choose Current in the navigation pane on the left. 
2.	Click the  ![Refresh](images/refreshicon.png "Refresh") icon to refresh the GuardDuty console. A finding should be visible right after the prior EC2 one with the text starting with either “*API GetParameters*” or “*Reconnaissance API*” (although the e-mail you received was only reporting that first Recon threat which  refers specifically to the threat “*UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom*.” If you click on one the finding then you can bring up a window showing the full details of this finding. You can also select the finding and click the **Actions** menu, click **Export** and extract the full JSON from the finding. More findings will begin to show up in the GuardDuty console. Below is a screenshot of one example showing a partial view of the finding details. 

![GuardDuty Finding](images/screenshot10.png "GuardDuty Finding")

3.	This finding indicates that the IAM credentials (the finding details show the Access key ID) are possibly compromised since API calls using those credentials are being made from an IP address on a threat list. The security team would be notified and proper steps to address this would be follow. This would usually lead to investigation of the IAM Credentials and most likely disabling them if they belong to am IAM user or Revoking the Session if they belong to an IAM Role. 
4.	Similar to the compromised EC2 instance finding, Alice set up a CloudWatch Event Rule for this finding which we should examine now.

**Scenario Note**: All of these findings showing up after the initial “*UnauthorizedAccess:EC2/MaliciousIPCaller.Custom*” are regarding IAM access key compromises. These IAM findings are being generated by the “malicious EC2” instance making API calls. These API calls generate findings because the EIP of that instance is in a custom threat list.

### Diagram of the Attack and Detection

![Attack 2](images/attack2.png "Attack2")

### View the CloudWatch Event rule for the compromised IAM credential

1.	Navigate to the [CloudWatch console](https://us-east-2.console.aws.amazon.com/cloudwatch/home?) and click on **Rules** under the **Events** section of the left. 
2.	You will see three Rules in the list that were created by the CloudFormation template each beginning with the phrase “*GuardDuty-Event*” (you may see other rules depending on if any other rules were created for other purposes.) 
3.	The rule that Alice configured for this “compromised access key” scenario is called “*GuardDuty-Event-EC2-MaliciousIPCaller*”. Click on that rule. 
4.	Under the Targets section you will see one for an SNS Topic. The SNS topic entry was responsible for sending an email to the e-mail address you entered in the parameters for the CloudFormation template. You can see this SNS Topic Trigger also has a transformation filter to modify the email to make it more user friendly.
5.	Alice did not set up a Lambda function to remediate this threat but there are many options that can be employed for dealing with this potential compromise. There could be an investigative phase before and action is taken including disabling the IAM Credential or the revoking the IAM Role session. This is also where a third-party solution could be examined. In many of the customer use cases we have seen CloudWatch Events are used to send findings to an existing customer’s SIEM. Many [Amazon GuardDuty partners](https://aws.amazon.com/guardduty/resources/partners/) already have solutions for GuardDuty to make ingestion simple, help with analysis of findings and also to help set up remediations. Some of these remediation options use Lambda so instead of a CloudWatch Event Rule triggering Lambda, the solution itself could trigger a workflow which would include Lambda to take action.

## Attack scenario 3 – IAM Role Credential Exfiltration <a name="attack3"/>

We will look at one final threat that Alice anticipated. For the previous attack we mentioned that the EC2 Instance was making API calls using the temp credentials from an IAM Role for EC2. Alice wanted to monitor for and remediate the situation where IAM temporary security credentials are copied (stolen) from an EC2 instance and used from a system external to the AWS network. She has already created the CloudWatch Events rule and Lambda function for this remediation. This vector will require you to take some manual steps in order to trigger it though (which of course will mildly reduce the illusion that this is an actual attack scenario.) The manual action you take will result in GuardDuty generating a High Severity finding and is an interesting threat because it indicates that an unusual and potentially serious event is occurring. 

In order to produce this finding you will need to copy the IAM temporary security credentials from the EC2 instance (or from the Parameters in AWS Systems Manager where the CloudFormation template has copied the IAM temporary security credentials  information) and make API calls from your laptop (the calls need to come from outside the AWS network). 

### Retrieve the IAM temporary security credentials from the EC2 instance

In order to simulate this last and final attack you will need to retrieve the IAM temporary security credentials  generated by the IAM Role for EC2. You will need to copy these credentials to your laptop and then set up a local AWS CLI profile. You can retrieve the credentials by either [connecting to the instance via SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html) and querying the [instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html) or you can find the credentials in the Parameter store of AWS Systems Manager. On startup the EC2 instance with the credentials exported them to the Parameter Store to allow you to easily retrieve these without connecting to the instance. Below are instructions for retrieving the credentials from the Parameter Store:

Finding the Credentials in the Parameter Store:
1. Open the [EC2 console](https://console.aws.amazon.com/ec2)
2. In the navigation pane on the left, scroll all the way to the bottom and click on **Parameter Store**
3. You should see three parameters: *gd_access_key_id*, *gd_secret_key*, and *gd_session_token*. As shown below, click on the parameters and in the **Description** tab and then click **Show** to decrypt and view the secret.

![Parameters](images/screenshot11.png "Parameters")

### Creating a new AWS CLI profile on your laptop to use the IAM temporary credentials 

Now that you have retrieved the IAM temporary security credentials you will need to copy them to your computer (this can be your laptop or any other computer not on the AWS network) and add them to an AWS CLI profile. There are a number of ways to do this but below are some commands to help get you started:

From a command prompt run the following commands:
```
aws configure set profile.attacker.region <region>
aws configure set profile.attacker.aws_access_key_id <gd_access_key>
aws configure set profile.attacker.aws_secret_access_key <gd_secret_key>
aws configure set profile.attacker.aws_session_token <gd_session_token>

```

If you view your local aws credentials file you should now see an [attacker] profile with the stolen IAM temporary credentials.

### Run commands using the IAM temporary credentials

Now that you have your named profile you can use it to make API calls. Use the commands below to query different services to see what you have access to (don't be surprised if you see some access denied responses):

**Let's see if we have any IAM permissions:**
```
aws iam get-user --profile attacker

aws iam create-user --user-name Chuck --profile attacker

```

**What about DynamoDB:**
```
aws dynamodb list-tables --profile attacker

aws dynamodb describe-table --table-name GuardDuty-BlogPost --profile attacker
```

**Can we query the data?**
```
aws dynamodb scan --table-name GuardDuty-Example-Customer-DB --profile attacker

aws dynamodb put-item --table-name GuardDuty-Example-Customer-DB --item '{"name":{"S":"Joshua Tree"},"state":{"S":"Michigan"},"website":{"S":"https://www.nps.gov/yell/index.htm"}}' --profile attacker

aws dynamodb scan --table-name GuardDuty-Example-Customer-DB --profile attacker

aws dynamodb delete-table --table-name GuardDuty-Example-Customer-DB --profile attacker

aws dynamodb list-tables --profile attacker
```

**Do you have access to Systems Manager Parameter Store:**
```
aws ssm describe-parameters --profile attacker

aws ssm get-parameters --names "gd_prod_dbpwd_sample" --profile attacker

aws ssm get-parameters --names "gd_prod_dbpwd_sample" --with-decryption --profile attacker

aws ssm delete-parameter --name "gd_prod_dbpwd_sample" --profile attacker
```

Feel free to make other calls to see what you have access to with those credentials.

### View the Lambda function designed to remediate this finding

You should eventually see alerts sent to your email and findings show up in the GuardDuty console for UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration. The alert for this threat will have a high severity. In addition, Alice set up a remediation for this threat that we will now examine.

Go to the Lambda console and review the function named GuardDuty-Example-Remediation-InstanceCredentialExfiltration.

To verify that the InstanceCredentialExfiltration finding was remediated you can run one of the CLI commands you ran earlier (e.g. aws dynamodb list-tables --profile attacker). You should see a response that states that there is an explicit deny for that action. This is because the remediation Lambda Function attaches a policy to the EC2 IAM Role that revokes all active sessions. You can view this policy within IAM under the Role.



## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
