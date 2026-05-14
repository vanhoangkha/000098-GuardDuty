---
title : "Credential Exfiltration"
date: 2024-01-01
weight : 2
chapter : false
pre : " <b> 6.2 </b> "
---

**Contents**
- [Kiến Trúc Tổng Quan](#kiến-trúc-tổng-quan)
- [Quá Trình Điều Tra](#quá-trình-điều-tra)
- [Câu Hỏi Ôn Tập](#câu-hỏi-ôn-tập)

#### Kiến Trúc Tổng Quan
![architecture-overview](/images/6-architecture-overview.png?featherlight=false&width=60pc)

1. Máy chủ từ xa truy cập đến *EC2 compromised instance* và đánh cắp **IAM role credential** thông qua dữ liệu **Metadata**.
2. Máy chủ này thiết lập AWS CLI Profile để tiến hành gọi API đến tài khoản AWS.
3. GuardDuty sinh ra những Findings liên quan và đồng thời gửi tới GuardDuty console và EventBridge Events.
4. EventBridge Event Rule kích hoạt SNS Topic và Lambda Function.
5. SNS Topic tiến hành gửi thông báo E-mail với chi tiết Finding.
6. Lambda Function tiến hành gán một chính sách mới nhằm thu hồi mọi Sessions đang hoạt động.

#### Quá Trình Điều Tra

---
**Truy cập GuardDuty Console**

Để tiến hành xem xét các Findings:
1. Truy cập vào GuardDuty Console ở **us-west-2**
2. Chúng ta sẽ thấy được Findings với định dạng như sau - `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`.

![guardduty-finding-unauthorized-iam-instance-credential-exfiltration](/images/6-guardduty-finding-unauthorized-iam-instance-credential-exfiltration.png?featherlight=false&width=90pc)

3. Nếu không có bất kỳ Finding nào, tiến hành nhấn nút Refresh và đợi.
4. Từ Finding - `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`, chúng ta có thể dễ dàng truy xuất một số thông tin sau:
   1. **High Severity**
   2. Thông báo rằng có người cố ý sử dụng IAM role credential ở ngoài EC2 instance

![guardduty-finding-unauthorized-iam-instance-credential-exfiltration-details](/images/6-guardduty-finding-unauthorized-iam-instance-credential-exfiltration-details.png?featherlight=false&width=90pc)

> Mỗi GuardDuty Finding sẽ được gán một mức độ nghiêm trọng cụ thể - Low/Medium/High. Các mức độ này được định nghĩa bởi AWS, chúng được dùng để phân loại và xác định  

---
**Kiểm tra EventBridge Event Rule**

1. Truy cập vào CloudWatch Console ở **us-west-2**.
2. Ở thanh điều hướng bên tay trái, dưới **Events**, chọn **Rules**. Bạn sẽ thấy có 3 quy tắc đã được thiết lập (bởi CloudFormation Template), bắt đầu với tiền tố có dạng sau `GuardDuty-Event.`.
3. Tiến hành chọn quy tắc có tên là `GuardDuty-Event-IAMUser-InstanceCredentialExfiltration`.

![eventbridge-event-iam-credential-exfiltration](/images/6-eventbridge-event-iam-credential-exfiltration.png?featherlight=false&width=90pc)

4. Ở mục **Event Pattern**, chúng ta dễ dàng thấy được nguồn dữ liệu mà Event này sẽ ghi nhận và tiến hành kích hoạt các **Target** khi có bất kỳ sự kiện nào.

![eventbridge-iam-exfiltration-event-pattern-targets](/images/6-eventbridge-iam-exfiltration-event-pattern-targets.png?featherlight=false&width=90pc)

> Bạn có thể tạo EventBridge Event Rule nhằm ghi nhận sự kiện của một loại Finding cụ thể hay bất kỳ loại Finding nào.

Sau đây là một ví dụ nhằm ghi nhận bất kỳ sự kiện nào thuộc GuardDuty Findings.
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
**Kiểm tra quá trình Remediation với Lambda Function**

Alice đã thiết lập quá trình Remediation nhằm phản ứng tự động với mối nguy hại này thông qua Lambda function. Chúng ta có thể kiểm tra đoạn mã được lập trình để hiểu thêm về quá trình này.
1. Truy cập vào Lambda Console ở **us-west-2**.
2. Ở thanh điều hướng bên tay trái, chọn **Functions** và tìm kiếm `GuardDuty-Example-Remediation-InstanceCredentialExfiltration`.

![lambda-function](/images/6-lambda-function.png?featherlight=false&width=90pc)

3. Về cơ bản, Lambda function này sẽ truy xuất thông tin về IAM Role từ Finding và tiến hành thêm IAM Policy.

![lambda-function-code](/images/6-lambda-function-code.png?featherlight=false&width=90pc)

> Những Permissions nào mà Lambda Function cần để thực hiện quá trình Remediation? Liệu có rủi ro nào có thể xảy ra với cấp độ Permissions hiện tại?

---
**Kiểm chứng quá trình Remediation**

Để tiến hành kiểm chứng liệu Finding `InstanceCredentialExfiltration` đã được giải quyết triệt để, chúng ta sẽ lần lượt tiến hành các bước sau.

- **Kiểm chứng thông qua AWS CLI**

Tiến hành thực thi câu lệnh sau:

```bash
aws dynamodb list-tables --profile badbob
```

Chúng ta sẽ nhận được kết quả trả về sẽ là `AccessDeniedException` cho câu lệnh thực thi.

![aws-cli-dynamodb-access-denied-exception](/images/6-aws-cli-dynamodb-access-denied-exception.png?featherlight=false&width=90pc)

- **Kiểm chứng thông qua AWS Console**

Tiến hành đánh giá IAM Policy đã được thêm vào IAM Role trong quá trình Remediation.
1. Truy cập vào IAM Console.
2. Ở thanh điều hướng bên tay trái, chọn **Roles** và tìm kiếm `GuardDuty-Example-EC2-Compromised`. Đây là IAM Role mà chúng ta sẽ xác định được thông qua GuardDuty Finding.

![iam-role](/images/6-iam-role.png?featherlight=false&width=90pc)

3. Bấm chọn thanh **Permissions**, nhấn vào chính sách IAM Policy `RevokeOldSessions`.

![iam-role-permissions](/images/6-iam-role-permissions.png?featherlight=false&width=90pc)

#### Câu Hỏi Ôn Tập
1. Có những rủi ro liên quan nào xuất hiện trong quá trình Remediation?
2. Liệu có những EC2 instances khác cũng sử dụng IAM Role này không?
