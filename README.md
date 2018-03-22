# Getting Started with Amazon GuardDuty

This GitHub repository walks you through a scenario covering threat detection and remediation using Amazon GuardDuty. The scenario covers an attack that simulates a few threat vectors which represents just a small sample of the threats that GuardDuty is able to detect. In addition, we look at how to view the GuardDuty findings, how to alert based on the findings, and finally how to remediate findings. The CloudFormation template builds out the assets used to simulate the attacks and the instructions are provided on how to analyze and remediate the findings.

## What is Created?
The CloudFormation template will create the following resources:
  * Three [Amazon EC2](https://aws.amazon.com/ec2/) Instances (all using a t2.micro instance type):
    * Two Instances that contain the name “*Compromised Instance*” 
    * One instance that contains the name “*Malicious Instance*”


## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.
