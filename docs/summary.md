# Summary

By walking through these scenarios you generated, analyzed, and remediated some or all of the following findings in your environment:

* <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#unauthorizedaccess-ec2-maliciousipcallercustom" target="_blank">UnauthorizedAccess:EC2/MaliciousIPCaller.Custom</a>
* <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html#recon-iam-maliciousipcallercustom" target="_blank">Recon:IAMUser/MaliciousIPCaller.Custom</a>
* <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html#unauthorizedaccess-iam-maliciousipcallercustom" target="_blank">UnauthorizedAccess:IAMUser/MaliciousIPCaller.Custom</a>
* <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-s3.html#discovery-s3-maliciousipcallercustom" target="_blank">Discovery:S3/MaliciousIPCaller.Custom</a>
* <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html#unauthorizedaccess-iam-instancecredentialexfiltrationoutsideaws" target="_blank">UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS</a>
* <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-s3.html#stealth-s3-serveraccessloggingdisabled" target="_blank">Stealth:S3/ServerAccessLoggingDisabled</a>
* <a href="https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-s3.html#policy-s3-accountblockpublicaccessdisabled" target="_blank">Policy:S3/AccountBlockPublicAccessDisabled</a>


Now that you understand the different components of the GuardDuty service and how it integrates with other AWS services, you can explore ways of using it across your own environments.

## Cleanup

To remove the assets created by the CloudFormation, follow these steps: 

1. Delete the S3 buckets that were created by the CloudFormation template (it will have names that begins with **guardduty-example**).  This needs to be done because data was put in the bucket and CloudFormation will not allow you to delete a bucket with data in it.
2. Delete the compromised instance IAM Roles (it will have the name **GuardDuty-Example-EC2-Compromised**). Because one of the Lambda functions added an additional policy to this Role you need to manually delete this.
3. Delete the custom Threat List within GuardDuty.  Within the GuardDuty console click **Lists** in the left navigation.  From there delete the **Example-Threat-List**.
4. Disable GuardDuty.  Within the GuardDuty console click **Settings**. Then check the box to **Disable GuardDuty** and save.
	
	> Suspending GuardDuty stops the service from monitoring so you don't incur any costs and won't receive any findings **but** it will retain your existing findings and baseline activity.

5. Delete the CloudFormation Stack. If you see any errors, it means you didn't delete the S3 Bucket or IAM role in the previous steps.   
