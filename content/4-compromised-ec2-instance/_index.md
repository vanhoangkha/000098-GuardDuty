---
title : "Compromised EC2 Instance"
date: 2024-01-01
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

**Scenario 1: Compromised EC2 Instance**

As usual, you arrive at the office early on Monday morning, prepare a cup of coffee, sit down and open your laptop, starting a series of routine tasks like sending E-mails and writing plans. Suddenly, you start receiving a stream of E-mails with content related to the latest detection of threats, which you have never seen before, however you start to learn and investigate. right away. The good news is that your colleague Alice has already set up automatic responses for those Findings so they are resolved instantly.

The first E-mail you received with your EC2 instance may have been compromised as follows:

```text
GuardDuty Finding | ID: 1xx: The EC2 instance i-xxxxxxxxx may be compromised and should be evaluated
```

The content of the second E-mail immediately after that you received with the situation resolved immediately is as follows:

```text
GuardDuty Remediation | ID: 1xx: GuardDuty discovered an EC2 instance (Instance ID: i-xxx) that is communicating outbound with an IP Address on a threat list that you uploaded. All security groups have been removed and it has been isolated. Please follow up with any additional remediation actions.
```

**Content**
- [Architecture overview](#architecture-overview)
- [Investigation process](#investigation-process)
- [Review questions](#review-questions)

#### Architecture overview
![architecture-overview](/images/4-architecture-overview.png?featherlight=false&width=60pc)

1. An *EC2 compromised instance* sends pings to the EIP address of a malicious EC2 instance. That EIP address has been added to **Custom Threat List**.
2. GuardDuty conducts monitoring of VPC Flow Logs (including CloudTrail and DNS Logs) and situation analysis based on Machine Learning, **Custom Threat List**, and others.
3. GuardDuty generates a Finding and displays it on the GuardDuty Console and sends the event to EventBridge Events.
4. Based on this event, the EventBridge Event Rule reacts and simultaneously activates the SNS Topic and Lambda Function respectively.
5. SNS Topic will send an E-mail with Finding details to you.
6. Lambda Function will isolate *EC2 compromised instance*.

When Alice sets up an E-mail notification for this event, she adds only certain information about that Finding and configures the Lambda Function to automatically isolate the *EC2 compromised instance*. Even though Finding has been resolved, you still decide to dig into more detail about these current settings and configurations of Alice.

#### Investigation process

---

**Go to GuardDuty Console**

While you can see these Findings from the GuardDuty Console, most customers want to bring them together, from AWS Regions and AWS Accounts to a centralized secure data management system (**SIEM**) ) to conduct analysis and perform the Remediations process. The most common approach is to configure GuardDuty under an `Admin/Member` model and use a combination of EventBridge Event Rules and Lambda Functions to push these Findings to **SIEM** or a ** Centralized Logging Framework**. There are also many solutions from AWS partners that help customers make data push and consolidation tasks as easy as possible.
1. Access GuardDuty Console at **us-west-2**
2. We should see a Finding with the following format `UnauthorizedAccess:EC2/MaliciousIPCaller.Custom`.

![findings-ec2-malicous-ip-caller](/images/4-findings-ec2-malicous-ip-caller.png?featherlight=false&width=90pc)

3. If there isn't any Finding, proceed to press the Refresh button and wait.

> Findings can be retrieved from the GuardDuty console for 90 days.

> Based on the format reviewed in detail in the previous section, what security incident can you pinpoint through the Finding style?
1. In your environment, this type of Finding indicates that an EC2 instance is communicating to an IP address (added to **Threat Lists**).
2. Select **Lists** in the navigation bar (left-hand side) to see the **Threat List** that Alice added previously - `Example-Threat-List`.

![guardduty-lists](/images/4-guardduty-lists.png?featherlight=false&width=90pc)

> GuardDuty uses Threat Intelligence systems provided by the AWS Security team and 3rd parties such as *ProofPoint* and *CrowdStike*. You can extend GuardDuty's visibility by manually configuring the Trusted IP Lists (**Trusted IP Lists**) and the Threat Lists (**Threat Lists**). If you have set up GuardDuty under the Admin/Member structure, from the GuardDuty Admin account you can manage the lists above and let the Members accounts inherit. By default, Members accounts will not be able to edit these lists.

{{% notice note %}}
In this emulation scenario, *EC2 compromised instance* only accesses **EIP** of another EC2 instance in the same VPC to internalize the emulation process and data processing that only occurs in the same environment. your school. The CloudFormation template will automatically create a threat list (**Threat Lists**) and assign this **EIP** address to it.
{{% /notice %}}

---

**Check EventBridge Event Rule**

Alice uses the EventBridge Event Rules to notify you of the Findings along with the steps of the Remediations process. Shall we conduct a more detailed survey to understand what Alice has set up and how does this work?
1. Access the EventBridge Console at **us-west-2**.
2. In the left-hand navigation bar, under **Events**, select **Rules**. You will see 3 rules have been set up (by CloudFormation Template), starting with a prefix of the form `GuardDuty-Event`.

![eventbridge-events-rules](/images/4-eventbridge-events-rules.png?featherlight=false&width=90pc)

3. Proceed to select the rule named `GuardDuty-Event-EC2-MaliciousIPCaller`.

![eventbridge-event-ec2-malicious-ip-caller](/images/4-eventbridge-event-ec2-malicious-ip-caller.png?featherlight=false&width=90pc)

4. You will easily notice that there are 2 targets in the **Targets** area.
   1. **Lambda Function**
   2. **SNS Topic**: Proceed to send E-mail notifications to you based on data provided by EventBridge Event Rule. Instead of the entire JSON data being used, by using **Input Transformer**, Alice customized the message content.

![eventbridge-event-ec2-malicious-ip-caller-targets](/images/4-eventbridge-event-ec2-malicious-ip-caller-targets.png?featherlight=false&width=90pc)

---

**Check Remediation process based on Lambda Function**

The Lambda Function is the key that holds the logic to perform the steps of the Remediations process for Findings. Alice has set up a Lambda Function to remove and replace the **Security Group** of the *EC2 compromised instance* with a **Security Group** that does not contain any `Ingress/Egress` rules. This will help isolate the *EC2 compromised instance* from the current network.

To test the Remediation process:
1. From the `GuardDuty-Event-EC2-MaliciousIPCaller` rule, in the **Targets** area, in the **Type** section is Lambda Function, we search for the corresponding **Resource Name**.

![eventbridge-event-ec2-malicious-ip-caller-targets-lambda](/images/4-eventbridge-event-ec2-malicious-ip-caller-targets-lambda.png?featherlight=false&width=90pc)

2. At the Lambda Function console, search for **Resource Name** following the previous step.

![lambda-Remediation-EC2MaliciousIPCaller](/images/4-lambda-Remediation-EC2MaliciousIPCaller.png?featherlight=false&width=90pc)

3. We can look at some items
   1. Configuration
      1. In the **Designer** bar, we will easily see the relationship with the EventBridge Event Rule.
      2. In the **Function code** section, the coding logic will be executed here.
   2. Permissions
   3. Monitoring

![lambda-Remediation-EC2MaliciousIPCaller-overview](/images/4-lambda-Remediation-EC2MaliciousIPCaller-overview.png?featherlight=false&width=90pc)

---

**Confirm Remediation was successful**

To ensure the results of the Remediation process, we need to see if the EC2 instance is isolated or not. At this point, you have received an E-mail with some important information.
1. Access the EC2 console at **us-west-2**.
2. Select `Instances (Running)`, they will see 3 EC2 instances with prefixes starting with the following format `GuardDuty-Example`.

![ec2-running](/images/4-ec2-running.png?width=90pc)

3. Based on the instance ID from GuardDuty Finding or the E-mail message, we choose the corresponding EC2 instance - `GuardDuty-Example: Compromised Instance: Scenario 1`.

![guardduty-finding-MaliciousIPCaller-target](/images/4-guardduty-finding-MaliciousIPCaller-target.png?featherlight=false&width=90pc)

![ec2-compromised-scenario-1](/images/4-ec2-compromised-scenario-1.png?featherlight=false&width=90pc)

4. After the Remediation process is completed, we will check **Security Group** of this *EC2 compromised instance*, which will have the same name format as `ForensicSecurityGroup`.
5. `ForensicSecurityGroup` will not have any `Ingress/Egress` rules containing IP addresses in `Example-Threat-List`.

![ec2-compromised-scenario-1-security-group](/images/4-ec2-compromised-scenario-1-security-group.png?featherlight=false&width=90pc)

#### Review questions
1. What data source was used by GuardDuty to identify this threat?
2. Will isolation affect applications running inside this EC2 instance?
3. How can we add more detailed information to E-mail notifications?