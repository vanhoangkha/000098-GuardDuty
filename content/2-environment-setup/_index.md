---
title : "Preparation"
date: 2024-01-01
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

#### Preparation

{{% notice note %}}
The exercise will be set at **us-west-2 (Oregon)**.
{{% /notice %}}

**Contents**
- [Preparation](#preparation)
- [Prerequisites](#prerequisites)
- [Implementation](#implementation)
- [AWS Resources Prepared?](#aws-resources-prepared)

#### Prerequisites
- **AWS Account**: An account for the **TESTING** environment.
- **Administrator rights**: Make sure you are using an **IAM user** with Administrator privileges.
- **AWS CLI**: Make sure you are ready to use the AWS CLI to simulate attacks from your computer.

#### Implementation

---

**Activate Amazon GuardDuty**

1. Sign in to AWS Console, and access the [GuardDuty] service (https://us-west-2.console.aws.amazon.com/guardduty/home).
2. Start by selecting **Get Started**.

![guardduty-get-started](/images/2-guardduty-get-started.png?featherlight=false&width=90pc)

3. For new accounts, AWS will give us a 30-day trial, to start using, click the `Enable GuardDuty` button.

![guardduty-enable](/images/2-guardduty-enable.png?featherlight=false&width=90pc)

---

**Prepare resources with AWS CloudFormation**

1. Sign in to AWS Console, access the [CloudFormation] service (https://us-west-2.console.aws.amazon.com/cloudformation/home)
2. Proceed to create a new Stack by pressing the `Create Stack` button.
3. On the `Specify template` page,
- Download the template at [here](https://github.com/AWS-First-Cloud-Journey/GuardDuty-Hands-On/archive/refs/heads/main.zip)
- Proceed to upload an existing Template by using the `Upload a template file` button.
4. On the `Parameters` page, we will enter some required information as follows:
   1. `EmailAddress`: Personal Email account to be able to receive notifications.

![cloudformation-stack-specify-parameters](/images/2-cloudformation-stack-specify-parameters.png?featherlight=false&width=90pc)

5. On the `Specify Stack Details` page, select the `Next` button.
6. On the `Configure stack options` page, select the `Next` button.
7. On the `Capabilities` page, proceed to accept (Acknowledge) to allow the Template to create IAM roles, and finally select the `Create Stack` button.

![cloudformation-stack-create-complete](/images/2-cloudformation-stack-create-complete.png?featherlight=false&width=90pc)

{{% notice note %}}
The above process will take 5-10 minutes until we see the status of the Stack as `CREATE_COMPLETE`. We will receive an email notification with the same subject as `AWS Notification - Subscription Confirmation`.
{{% /notice %}}

![sns-notification-subscription-confirmation](/images/2-sns-notification-subscription-confirmation.png?featherlight=false&width=90pc)

![sns-notification-subscription-confirmation](/images/2-sns-notification-subscription-confirmation1.png?featherlight=false&width=90pc)

Initial results will show 10 minutes after the CloudFormation Stack setup is completed.

#### AWS Resources Prepared?

**CloudFormation Template** will prepare us with the following resources:

1. EC2 Service:
   - 2 instances named `Compromised Instance`.
   - 1 instance named `Malicious Instance`.
2. IAM service:
   - 1 **IAM Role** for EC2 instance with access to **SSM Parameter Store** and **DynamoDB**.
3. SNS service:
   - 1 **SNS Topic** send notifications via E-mail.
4. EventBridge Service:
   - 3 **EventBridge Events Rules** for triggering notifications and remediation process.
5. Lambda Service:
   - 2 **Lambda Functions** to fix the vulnerabilities.
6. SSM service:
   - 1 **SSM Parameter Store** is used to store the password for the *TESTING* environment.