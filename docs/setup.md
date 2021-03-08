# Environment Setup

### Prerequisites

* **AWS Account**: Given that you will be simulating attacks and doing remediations, you should run this in a non-production account. After running through these scenarios, you can look at how you can implement GuardDuty and associated automated remediation workflows in a multi-account structure so you are able to aggregate findings from other accounts and use the service in a more scalable manner.
* **Admin privileges**: Ensure you are using an AWS IAM User with Admin privileges.
* **AWS CLI**: You will be using the <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html" target="_blank">AWS CLI</a> for simulating one of the attacks so be sure you have installed on your local machine.

> All of the links assume you are using the **us-west-2 (Oregon)** region.

## Deploy the environment

??? info "AWS Sponsored Event"
    1. Navigate to the <a href="https://dashboard.eventengine.run" target="_blank">Event Engine dashboard</a>
	  2. Enter your **team hash** code.
	  3. Click **AWS Console**. Please use the **us-west-2 (Oregon)** region for this workshop.
    4. Let's enable GuardDuty, in just 2 clicks! Navigate to the <a href="https://us-west-2.console.aws.amazon.com/guardduty/home?" target="_blank">GuardDuty Console</a> and click **Get Started**.
    ![Get Started](images/screenshot1.png "Get Started")
    5. On the next screen click **Enable GuardDuty**.
    ![Enable GuardDuty](images/screenshot2.png "Enable GuardDuty")
    6. Click Deploy to AWS to launch the CloudFormation stack to setup the lab environment.
      Please note we are using the US West 2 (Oregon):
      <a href="https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=GuardDuty-Hands-On&templateURL=https://sa-security-specialist-workshops-us-west-2.s3-us-west-2.amazonaws.com/guardduty-hands-on/amazon-guard-duty-revamped-v2.yml" target="_blank">![Deploy in us-west-2](./images/deploy-to-aws.png)</a>

    7. Click the **Deploy to AWS** button above.  This will automatically take you to the console to run the template.  

    8. Click **Next** on the **Specify Template** page.

    9. Under **Parameters** Enter your **EmailAddress** to receive the notifications during the lab, on the **Specify stack    details** page. Then click **Next**

    10. Scroll down and click **Next** on the **Configure stack options** page.

    11. Finally, acknowledge that the template will create IAM roles under **Capabilities** and click **Create stack**.

    12. You should see an email with subject **AWS Notification - Subscription Confirmation**. Please click on **Confirm Subscription** which will ensure you receive notifications during the lab.

    This will bring you back to the CloudFormation console. You can refresh the page to see the stack starting to create. Before moving on, make sure the stack is in a **CREATE_COMPLETE**.

??? info "Individual"


    1. Let's enable GuardDuty, in just 2 clicks!
       Please use the **us-west-2 (Oregon)** region for this workshop. Navigate to the <a href="https://us-west-2.console.aws.amazon.com/guardduty/home?" target="_blank">GuardDuty Console</a> and click **Get Started**.
    ![Get Started](images/screenshot1.png "Get Started")
    2. On the next screen click **Enable GuardDuty**.
    ![Enable GuardDuty](images/screenshot2.png "Enable GuardDuty")

    3. Click Deploy to AWS to launch the CloudFormation stack to setup the lab environment.
      Please note we are using the US West 2 (Oregon):
      <a href="https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=GuardDuty-Hands-On&templateURL=https://sa-security-specialist-workshops-us-west-2.s3-us-west-2.amazonaws.com/guardduty-hands-on/amazon-guard-duty-revamped-v2.yml" target="_blank">![Deploy in us-west-2](./images/deploy-to-aws.png)</a>

    7. Click the **Deploy to AWS** button above.  This will automatically take you to the console to run the template.  

    8. Click **Next** on the **Specify Template** page.

    9. Under **Parameters** Enter your **EmailAddress** to receive the notifications during the lab, on the **Specify stack    details** page. Then click **Next**

    10. Scroll down and click **Next** on the **Configure stack options** page.

    11. Finally, acknowledge that the template will create IAM roles under **Capabilities** and click **Create stack**.

    This will bring you back to the CloudFormation console. You can refresh the page to see the stack starting to create. Before moving on, make sure the stack is in a **CREATE_COMPLETE**.

    12. You should see an email with subject **AWS Notification - Subscription Confirmation**. Please click on **Confirm Subscription** which will ensure you receive notifications during the lab.

While we wait for our CloudFormation stack to complete, take 5 minutes to read about GuardDuty **data sources** and **findings**. Then check back to confirm the CloudFormation build has completed, move onto to **What is created?** section below.


## How GuardDuty Works
### Data Sources

From the moment you enable GuardDuty it begins analyzing all of the VPC Flow Logs, CloudTrail logs, and DNS Logs in that region. DNS Logs are generated from the default AWS DNS resolvers used for your VPCs and are not an available data source to customers.  

> If you are using a 3rd party DNS resolver or if you set up your own DNS resolvers, then GuardDuty cannot access, process, and identify threats from that data source.

GuardDuty accesses all of these <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_data-sources.html" target="_blank">data sources</a> without any of them having to be enabled; although it is a best practice to enable CloudTrail and VPC Flow Logs for your own analysis. GuardDuty is a regional service so in order for the service to monitor these data sources in other regions you will need to enable it in those regions.  You can accomplish this by following the same steps above and enabling it through the console but most customers are using the APIs to programmatically enable it in all regions and across multiple accounts.  Regardless of the number of VPCs, IAM users, or other AWS resources in your account, there is no impact to your resources because all of the processing is being done within the managed service.

> The pricing for GuardDuty is based on the quantity of AWS CloudTrail Events analyzed and the volume of Amazon VPC Flow Log and DNS Log data analyzed (per GB).  Each region in an AWS Account has a free 30-day trial to better forecast what the cost of the service will be.

![GuardDuty Enabled](images/screenshot3.png "GuardDuty Enabled")

### Findings

Now that GuardDuty is enabled it is actively monitoring the three data sources for malicious or unauthorized behavior as it relates to your EC2 instances and AWS IAM Principals.  You should be taken directly to the **Findings** tab which will show finding details as GuardDuty detects them. After deploying the scenario, you will start to see GuardDuty findings being detected.  Each finding is broken down into the following format to allow for a concise yet readable description of potential security issues.

**ThreatPurpose : ResourceTypeAffected / ThreatFamilyName . ThreatFamilyVariant ! Artifact**

> Click <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-format.html" target="_blank">here</a> for a description of each part.

The more advanced behavioral and machine learning detections require a baseline (7 - 14 days) to be established so GuardDuty is able to learn the regular behavior and identity anomalies. An example of a finding that requires a baseline would be if an EC2 instance started communicating with a remote host on an unusual port or an IAM User who has no prior history of modifying Route Tables starts making modifications.  All of the findings generated in these scenarios will be based on signatures, so the findings will be detected 10 minutes after the completion of the CloudFormation stack.  The delay is due to the amount of time it takes for the information about a threat to appear in one of the data sources and the amount of time it takes for GuardDuty to access and analyze that particular data source.  

> Click <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html" target="_blank">here</a> for a complete list of current GuardDuty finding types.

GuardDuty sends notifications based on Amazon CloudWatch Events when any change in the findings takes place. These notifications are sent within 5 minutes of the finding. All subsequent occurrences of an existing finding will have the same ID as the original finding and notifications will be sent every 6 hours after the initial notification.  This is to eliminate alert fatigue due to the same finding.

The initial findings will begin to show up in GuardDuty 10 minutes after the CloudFormation stack creation completes.

### What is created?

The CloudFormation template will create the following resources:

  * Four <a href="https://aws.amazon.com/ec2/" target="_blank">Amazon EC2</a> Instances (and supporting network infrastructure)
    * Two Instances that contain the name “*Compromised Instance*”
    * Two instance that contains the name “*Malicious Instance*”
  * <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html" target="_blank">AWS IAM Role</a> For EC2 which will have permissions to SSM Parameter Store and DynamoDB
  * One <a href="https://docs.aws.amazon.com/sns/latest/dg/GettingStarted.html" target="_blank">Amazon SNS Topic</a> so you will be able to receive notifications
  * Four <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html" target="_blank">AWS CloudWatch Event</a> rules for triggering the appropriate notification or remediation
  * Two <a href="https://aws.amazon.com/lambda/" target="_blank">AWS Lambda</a> functions that will be used for remediating findings and will have permissions to modify Security Groups and revoke active IAM Role sessions (on only the IAM Role associated with this scenario)
  * <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html" target="_blank">AWS Systems Manager Parameter Store</a> value for storing a fake database password.
  * Three <a href="https://aws.amazon.com/s3/" target="_blank">S3 buckets</a>. 

> Make sure the CloudFormation stack is in a **CREATE_COMPLETE** status before moving on.
