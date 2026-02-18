---
title : "Introduction"
date : "2026-02-18"
weight : 1
chapter : false
pre : " <b> 1. </b> "
---

## What is Amazon GuardDuty?

**Amazon GuardDuty** is an intelligent threat detection service that uses machine learning, anomaly detection, and integrated threat intelligence to identify and prioritize potential threats.

### Key Features (2026)

| Feature | Description |
|---------|-------------|
| **Threat Detection** | Analyzes CloudTrail, VPC Flow Logs, DNS logs |
| **Malware Protection** | Scans EBS volumes and S3 objects |
| **Runtime Monitoring** | Real-time monitoring for EKS, ECS, EC2 |
| **Extended Threat Detection** | AI-powered attack sequence correlation |
| **Multi-Account Support** | Centralized management via Organizations |

### Data Sources

```
┌─────────────────────────────────────────────────┐
│              GuardDuty Data Sources             │
├─────────────────────────────────────────────────┤
│  ✓ AWS CloudTrail Events                        │
│  ✓ AWS CloudTrail Management Events             │
│  ✓ VPC Flow Logs                                │
│  ✓ DNS Logs                                     │
│  ✓ S3 Data Events (optional)                    │
│  ✓ EKS Audit Logs (optional)                    │
│  ✓ RDS Login Activity (optional)                │
│  ✓ Lambda Network Activity (optional)           │
│  ✓ EBS Malware Scanning                         │
│  ✓ Runtime Monitoring (EKS/ECS/EC2)             │
└─────────────────────────────────────────────────┘
```

### Finding Categories

| Category | Examples |
|----------|----------|
| **Reconnaissance** | Port scanning, API enumeration |
| **Instance Compromise** | Cryptocurrency mining, malware C2 |
| **Account Compromise** | Credential theft, privilege escalation |
| **Bucket Compromise** | Public access, data exfiltration |
| **Kubernetes Threats** | Privileged containers, suspicious exec |
| **Malware** | Trojans, ransomware, cryptominers |

### Severity Levels

| Level | Range | Action Required |
|-------|-------|-----------------|
| **Critical** | 9.0-10.0 | Immediate response |
| **High** | 7.0-8.9 | Priority investigation |
| **Medium** | 4.0-6.9 | Review within 24h |
| **Low** | 1.0-3.9 | Informational |

### Pricing Model (2026)

| Data Source | Pricing |
|-------------|---------|
| CloudTrail Management Events | Per 1M events |
| VPC Flow Logs | Per GB analyzed |
| DNS Logs | Per 1M queries |
| S3 Data Events | Per 1M events |
| EKS Audit Logs | Per 1M events |
| Malware Protection | Per GB scanned |
| Runtime Monitoring | Per vCPU-hour |

> **Tip:** Use the [GuardDuty pricing calculator](https://calculator.aws/#/addService/GuardDuty) to estimate costs.

### Best Practices

1. **Enable in all regions** - Threats can originate anywhere
2. **Use Organizations** - Centralized management
3. **Integrate with Security Hub** - Unified view
4. **Automate remediation** - EventBridge + Lambda
5. **Review findings regularly** - Tune suppression rules
6. **Enable all protection plans** - S3, EKS, Malware, Runtime

### References

- [GuardDuty Documentation](https://docs.aws.amazon.com/guardduty/)
- [GuardDuty Pricing](https://aws.amazon.com/guardduty/pricing/)
- [AWS Security Blog - GuardDuty](https://aws.amazon.com/blogs/security/tag/amazon-guardduty/)
