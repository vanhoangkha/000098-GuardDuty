---
title : "Giới thiệu"
date: 2024-01-01
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---
#### Tổng quan
Với **Amazon GuardDuty**, một dịch vụ được quản lý hoàn toàn bởi AWS, bài thực hành này sẽ khái quát làm thế nào để phát hiện những mối nguy hại đến hệ thống và khắc phục chúng. Chúng ta sẽ tiến hành phân tích, đánh giá và làm thế nào để báo động cũng như khắc phục các vấn đề bảo mật dựa trên những phát hiện (*Findings*) của GuardDuty.

Nhằm chuẩn bị cho bài thực hành này, bằng cách sử dụng **CloudFormation Template** có sẵn, chúng ta sẽ tái hiện những cuộc tấn công và cách khắc phục tự động bằng việc kết hợp giữa **EventBridge Event Rules** và **Lambda Functions**.
- **Cấp độ:** 300
- **Thời lượng:** 1-2 tiếng
- **Điều kiện cần:** IAM User (Admin) và AWS CLI
- **Các chức năng của **CSF** (Cybersecurity Framework):**
  - Bảo vệ (Protect)
  - Phát hiện (Detect)
  - Phản hồi (Respond)
- **Các góc nhìn bảo mật của **CAF** (Cloud Adoption Framework):**
  - Preventative (Khả năng phòng chống)
  - Detective (Khả năng truy vết)
  - Responsive (Khả năng phản ứng)
- **Các dịch vụ AWS được sử dụng:**
  - Amazon EventBridge
  - Amazon GuardDuty
  - AWS CloudTrail
  - AWS Lambda
  - VPC Security Groups
  - Amazon SNS

#### Thiết lập
> Bài thực hành sẽ được thiết lập ở **us-west-2 (Oregon)**.

Chi tiết xem thêm ở phần [**Thiết lập môi trường**](1-environment-setup/).

#### Tình Huống
Bài thực hành sẽ bao gồm các tình huống như sau:
| Thứ tự | Tên | Đặc tả | Giải pháp |
| ------ | --- | ------ | --------- |
| 1 | [Compromised EC2 instance](3-compromised-ec2-instance/) | Phát hiện và hồi phục EC2 instance bị tấn công | Sự kết hợp giữa **Amazon GuardDuty**, **Amazon EventBridge Event Rules** và **AWS Lambda** |
| 2 | [Compromised IAM credentials](4-compromised-iam-credentials/) | Xác định một cá nhân đang chủ động gọi API đến hệ thống trên AWS | Khắc phục mối nguy hại này ngay lập tức một cách thủ công (manually) |
| 3 | [IAM role exfiltration](5-iam-role-credential-exfiltration/) | Thông qua một credential bị rò rỉ, một cá nhân đang cố gắng xâm nhập và gọi API từ một máy chủ bên ngoài | Tiến hành khắc phục với **AWS Lambda** |

#### Dọn Dẹp
Chi tiết ở phần [**Dọn dẹp môi trường**](7-environment-cleanup/).