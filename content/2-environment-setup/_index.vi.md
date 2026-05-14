---
title : "Các bước chuẩn bị"
date: 2024-01-01
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

{{% notice note %}}
Bài thực hành sẽ được thiết lập ở **us-west-2 (Oregon)**.
{{% /notice %}}

**Contents**
- [Điều kiện cần](#điều-kiện-cần)
- [Triển khai](#triển-khai)
- [Tài nguyên AWS được chuẩn bị?](#tài-nguyên-aws-được-chuẩn-bị)

#### Điều kiện cần
- **Tài khoản AWS**: Một tài khoản dành cho môi trường **TESTING**.
- **Quyền hạn Administrator**: Bảo đảm rằng bạn đang sử dụng một **IAM user** với quyền hạn Administrator.
- **AWS CLI**: Bảo đảm rằng bạn sẵn sàng sử dụng AWS CLI cho mục đích giả lập các cuộc tấn công từ máy tính của mình.

#### Triển khai

---

**Kích hoạt Amazon GuardDuty**

1. Đăng nhập vào AWS Console, truy cập vào dịch vụ [GuardDuty](https://us-west-2.console.aws.amazon.com/guardduty/home).
2. Bắt đầu bằng cách chọn **Get Started**.

![guardduty-get-started](/images/2-guardduty-get-started.png?featherlight=false&width=90pc)

3. Đối với tài khoản mới, AWS sẽ cho chúng ta 30 ngày dùng thử, để bắt đầu sử dụng, nhấn chọn nút `Enable GuardDuty`.

![guardduty-enable](/images/2-guardduty-enable.png?featherlight=false&width=90pc)

---

**Chuẩn bị tài nguyên với AWS CloudFormation**

1. Đăng nhập vào AWS Console, truy cập vào dịch vụ [CloudFormation](https://us-west-2.console.aws.amazon.com/cloudformation/home)
2. Tiến hành tạo một Stack mới bằng việc nhấn nút `Create Stack`.
3. Ở trang `Specify template`, 
- Tải template tại [đây](https://github.com/AWS-First-Cloud-Journey/GuardDuty-Hands-On/archive/refs/heads/main.zip)
- Tiến hành tải lên Temlate có sẵn bằng cách dùng nút `Upload a template file`.
4. Ở trang `Parameters`, chúng ta sẽ nhập một số thông tin bắt buộc sau:
   1. `EmailAddress`: Tài khoản Email cá nhân để có thể nhận thông báo.

![cloudformation-stack-specify-parameters](/images/2-cloudformation-stack-specify-parameters.png?featherlight=false&width=90pc)

5. Ở trang `Specify Stack Details`, chọn nút `Next`.
6. Ở trang `Configure stack options`, chọn nút `Next`.
7. Ở trang `Capabilities`, tiến hành chấp nhận (Acknowledge) cho phép Template tạo các IAM role, cuối cùng chọn nút `Create Stack`.

![cloudformation-stack-create-complete](/images/2-cloudformation-stack-create-complete.png?featherlight=false&width=90pc)

{{% notice note %}}
Quá trình trên sẽ diễn ra trong vòng 5-10 phút cho tới khi chúng ta thấy được trạng thái của Stack là `CREATE_COMPLETE`. Sau đó, chúng ta sẽ nhận được một thông báo qua Email với chủ đề tương tự `AWS Notification - Subscription Confirmation`.
{{% /notice %}}

![sns-notification-subscription-confirmation](/images/2-sns-notification-subscription-confirmation.png?featherlight=false&width=90pc)

![sns-notification-subscription-confirmation](/images/2-sns-notification-subscription-confirmation1.png?featherlight=false&width=90pc)

Những kết quả ban đầu sẽ bắt đầu hiển thị sau 10 phút kể từ thời điểm thiết lập CloudFormation Stack hoàn thành.

#### Tài nguyên AWS được chuẩn bị?

**CloudFormation Template** sẽ chuẩn bị cho chúng ta các tài nguyên sau:

1. Dịch vụ EC2:
   - 2 instances với tên gọi là `Compromised Instance`.
   - 1 instance với tên gọi là `Malicious Instance`.
2. Dịch vụ IAM:
   - 1 **IAM Role** dành cho EC2 instance với quyền hạn truy cập đến **SSM Parameter Store** và **DynamoDB**.
3. Dịch vụ SNS:
   - 1 **SNS Topic** gửi thông báo qua E-mail.
4. Dịch vụ EventBridge:
   - 3 **EventBridge Events Rules** cho việc kích hoạt các thông báo và quá trình khắc phục.
5. Dịch vụ Lambda:
   - 2 **Lambda Functions** tiến hành khắc phục các lỗ hổng.  
6. Dịch vụ SSM:
   - 1 **SSM Parameter Store** dùng để chứa mật khẩu cho môi trường *TESTING*.
