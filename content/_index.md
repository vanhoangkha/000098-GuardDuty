---
title : "Getting Hands on with Amazon GuardDuty"
date : "2026-02-18"
weight : 1
chapter : false
---

# Getting Hands on with Amazon GuardDuty

#### Overview

**Amazon GuardDuty** is an intelligent threat detection service that continuously monitors your AWS accounts, workloads, and data for malicious activity. This workshop covers threat detection, analysis, and automated remediation.

**What's New in 2026:**
- **GuardDuty Extended Threat Detection** - AI/ML-powered attack sequence detection
- **GuardDuty Malware Protection for S3** - Scans objects uploaded to S3
- **GuardDuty Runtime Monitoring** - Real-time threat detection for EKS, ECS, EC2
- **GuardDuty Attack Sequence Findings** - Correlates multiple signals into attack narratives

#### Workshop Details

| Attribute | Value |
|-----------|-------|
| **Level** | 300 (Advanced) |
| **Duration** | 2-3 hours |
| **Cost** | ~$5-10 (cleanup after) |
| **Region** | us-west-2 (Oregon) |

#### AWS Services Used

| Service | Purpose |
|---------|---------|
| Amazon GuardDuty | Threat detection |
| Amazon EventBridge | Event routing |
| AWS Lambda | Automated remediation |
| AWS Security Hub | Centralized findings |
| Amazon SNS | Notifications |
| AWS CloudTrail | API logging |
| Amazon Detective | Investigation |

#### Scenarios Covered

| # | Scenario | Detection | Remediation |
|---|----------|-----------|-------------|
| 1 | Compromised EC2 Instance | GuardDuty + VPC Flow Logs | Auto-isolate with Lambda |
| 2 | Compromised IAM Credentials | GuardDuty + CloudTrail | Disable keys + notify |
| 3 | IAM Role Credential Exfiltration | GuardDuty anomaly detection | Revoke sessions + alert |
| 4 | **[NEW]** Malware in S3 | GuardDuty Malware Protection | Quarantine + scan |
| 5 | **[NEW]** EKS Runtime Threats | GuardDuty Runtime Monitoring | Pod isolation |

#### Prerequisites

- AWS Account with Admin access
- AWS CLI v2 configured
- Basic knowledge of IAM, VPC, Lambda

#### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     AWS Account                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   VPC       │    │ CloudTrail  │    │    S3       │     │
│  │  Flow Logs  │    │   Logs      │    │   Events    │     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘     │
│         │                  │                  │             │
│         └────────────┬─────┴─────────────────┘             │
│                      ▼                                      │
│              ┌───────────────┐                              │
│              │  GuardDuty    │                              │
│              │  (ML/AI)      │                              │
│              └───────┬───────┘                              │
│                      │ Findings                             │
│                      ▼                                      │
│              ┌───────────────┐                              │
│              │ EventBridge   │                              │
│              └───────┬───────┘                              │
│         ┌────────────┼────────────┐                        │
│         ▼            ▼            ▼                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                   │
│  │  Lambda  │ │   SNS    │ │ Security │                   │
│  │ Remediate│ │  Alert   │ │   Hub    │                   │
│  └──────────┘ └──────────┘ └──────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

#### Content

1. [Introduction](1-intro/)
2. [Environment Setup](2-environment-setup/)
3. [How GuardDuty Works](3-how-it-works/)
4. [Compromised EC2 Instance](4-compromised-ec2-instance/)
5. [Compromised IAM Credentials](5-compromised-iam-credentials/)
6. [IAM Role Credential Exfiltration](6-iam-role-credential-exfiltration/)
7. [Summary](7-summary/)
8. [Environment Cleanup](8-environment-cleanup/)

#### References

- [Amazon GuardDuty User Guide](https://docs.aws.amazon.com/guardduty/latest/ug/)
- [GuardDuty Finding Types](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)
- [AWS Security Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
- [GuardDuty Malware Protection](https://docs.aws.amazon.com/guardduty/latest/ug/malware-protection.html)
- [GuardDuty Runtime Monitoring](https://docs.aws.amazon.com/guardduty/latest/ug/runtime-monitoring.html)

#### Cost Optimization

- Enable GuardDuty only in regions you use
- Use S3 protection selectively for sensitive buckets
- Review and tune suppression rules to reduce noise

#### License

MIT License - AWS First Cloud Journey
