# Summary

By walking through these scenarios you generated, analyzed, and remediated all of the following threats in your environment:

* [UnauthorizedAccess:EC2/MaliciousIPCaller.Custom](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_unauthorized.html#unauthorized8)
* [Recon:IAMUser/MaliciousIPCaller.Custom](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_recon.html#recon2)
* [UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_unauthorized.html#unauthorized2)
* [UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_unauthorized.html#unauthorized11)

Now that you understand the different components of the GuardDuty service and how it integrates with other AWS services, you can explore ways of using it across your own environments.

## Cleanup

To remove the assets created by the CloudFormation, follow these steps: 

1. Delete the S3 bucket that was created by the CloudFormation template (it will have a name that begins with **guardduty-example**).  This needs to be done because data was put in the bucket and CloudFormation will not allow you to delete a bucket with data in it.
2. Delete the compromised instance IAM Role (it will have the name **GuardDuty-Example-EC2-Compromised**). Because one of the Lambda functions added an additional policy to this Role you need to manually delete this.
3. Delete the custom Threat List within GuardDuty.  Within the GuardDuty console click **Lists** in the left navigation.  From there delete the **Example-Threat-List**.
4. Disable GuardDuty (if you didn't have it enabled already).  Within the GuardDuty console click **Settings**. Then check the box to **Disable GuardDuty** and save.
	
	> Suspending GuardDuty stops the service from monitoring so you don't incur any costs and won't receive any findings **but** it will retain your existing findings and baseline activity.

5. Delete the CloudFormation Stack. If you see any errors, it means you didn't delete the S3 Bucket or IAM role in the previous steps.   