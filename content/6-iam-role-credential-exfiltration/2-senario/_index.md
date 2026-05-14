---
title : "Credential Exfiltration"
date: 2024-01-01
weight : 2
chapter : false
pre : " <b> 6.2 </b> "
---

**Contents**
- [Architecture Overview](#architecture-overview)
- [Investigation process](#investigation-process)
- [Review questions](#review-questions)

#### Architecture Overview
![architecture-overview](/images/6-architecture-overview.png?featherlight=false&width=60pc)

1. Remote server accesses *EC2 compromised instance* and steals **IAM role credential** through **Metadata** data.
2. This server sets up the AWS CLI Profile to make API calls to the AWS account.
3. GuardDuty generates related Findings and sends them simultaneously to the GuardDuty console and EventBridge Events.
4. EventBridge Event Rule activates SNS Topic and Lambda Function.
5. SNS Topic proceeds to send an E-mail notification with Finding details.
6. Lambda Function assigns a new policy to revoke all active Sessions.

#### Investigation process

---
**Go to GuardDuty Console**

To conduct a review of the Findings:
1. Access GuardDuty Console at **us-west-2**
2. We will see Findings in the following format - `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`.

![guardduty-finding-unauthorized-iam-instance-credential-exfiltration](/images/6-guardduty-finding-unauthorized-iam-instance-credential-exfiltration.png?featherlight=false&width=90pc)

3. If there isn't any Finding, proceed to press the Refresh button and wait.
4. From Finding - `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`, we can easily retrieve some of the following information:
   1. **High Severity**
   2. Notice that someone intentionally used the IAM role credential outside the EC2 instance

![guardduty-finding-unauthorized-iam-instance-credential-exfiltration-details](/images/6-guardduty-finding-unauthorized-iam-instance-credential-exfiltration-details.png?featherlight=false&width=90pc)

> Each GuardDuty Finding will be assigned a specific severity - Low/Medium/High. These levels are defined by AWS, they are used to classify and define

---
**Check EventBridge Event Rule**

1. Access the CloudWatch Console at **us-west-2**.
2. In the left-hand navigation bar, under **Events**, select **Rules**. You will see 3 rules have been set up (by CloudFormation Template), starting with a prefix of the following from `GuardDuty-Event.`.
3. Proceed to select the rule named `GuardDuty-Event-IAMUser-InstanceCredentialExfiltration`.

![eventbridge-event-iam-credential-exfiltration](/images/6-eventbridge-event-iam-credential-exfiltration.png?featherlight=false&width=90pc)

4. In the **Event Pattern** section, we can easily see the data source that this Event will record and proceed to trigger **Target** when there are any events.

![eventbridge-iam-exfiltration-event-pattern-targets](/images/6-eventbridge-iam-exfiltration-event-pattern-targets.png?featherlight=false&width=90pc)

> You can create an EventBridge Event Rule to record events of a particular type of Finding or any type of Finding.

The following is an example to document any GuardDuty Findings events.
```
{
  "detail-type": [
    "GuardDuty Finding"
  ],
  "source": [
    "aws.guardduty"
  ]
}
```

---
**Check the Remediation process with Lambda Function**

Alice has set up the Remediation process to automatically respond to this threat through the Lambda function. We can check the programmed code to understand more about this process.
1. Access the Lambda Console at **us-west-2**.
2. In the left hand navigation bar, select **Functions** and search for `GuardDuty-Example-Remediation-InstanceCredentialExfiltration`.

![lambda-function](/images/6-lambda-function.png?featherlight=false&width=90pc)

3. Basically, this Lambda function will retrieve information about IAM Role from Finding and proceed to add IAM Policy.

![lambda-function-code](/images/6-lambda-function-code.png?featherlight=false&width=90pc)

> What Permissions does the Lambda Function need to perform the Remediation process? Is there a possible risk with the current Permissions level?

---

**Verify the Remediation Process**

To verify that Finding `InstanceCredentialExfiltration` has been completely resolved, we will proceed with the following steps in turn.

- **Verify via AWS CLI**

Execute the following command:

```bash
aws dynamodb list-tables --profile badbob
```

We will get `AccessDeniedException` return for the command to execute.

![aws-cli-dynamodb-access-denied-exception](/images/6-aws-cli-dynamodb-access-denied-exception.png?featherlight=false&width=90pc)

- **Verify via AWS Console**

Conduct an evaluation of the IAM Policy that was added to the IAM Role during the Remediation process.
1. Access the IAM Console.
2. In the left hand navigation bar, select **Roles** and search for `GuardDuty-Example-EC2-Compromised`. This is the IAM Role that we will identify through GuardDuty Finding.

![iam-role](/images/6-iam-role.png?featherlight=false&width=90pc)

3. Click the **Permissions** bar, click on the IAM Policy `RevokeOldSessions`.

![iam-role-permissions](/images/6-iam-role-permissions.png?featherlight=false&width=90pc)

#### Review questions
1. What are the risks involved in the Remediation process?
2. Are there other EC2 instances that also use this IAM Role?