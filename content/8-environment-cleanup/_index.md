---
title : "Clean up resources"
date: 2024-01-01
weight : 8
chapter : false
pre : " <b> 8. </b> "
---
To perform the cleanup of the AWS resources used in this exercise, we perform the following steps in turn:

1. Delete the S3 bucket that was created by the CloudFormation Template with the format `guardduty-example`. This step is required because the CloudFormation Stack cannot be deleted if the S3 bucket has data inside.
2. Delete the IAM Role that is used for the EC2 compromised instance with the format `GuardDuty-Example-EC2-Compromised`. Because in a Remediation process, Lambda Function has added an IAM Policy, CloudFormation Stack will not be deleted if the IAM Role appears with other IAM Policy.
3. Remove **Custom Threat List** used for GuardDuty.
4. Disable the Amazon GuardDuty service.
   1. Go to GuardDuty Console.
   2. Click on **Settings**.
   3. Select `Disable GuardDuty` and click `Save`.
5. Delete the CloudFormation Stack. If you can't delete it, check the first step 2 again.