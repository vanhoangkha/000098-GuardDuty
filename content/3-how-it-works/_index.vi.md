---
title : "Về GuardDuty"
date: 2024-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

**Nội dung**
- [Nguồn Dữ liệu](#nguồn-dữ-liệu)
- [Findings](#findings)

#### Nguồn Dữ liệu
Kể từ khi được kích hoạt trong một AWS Region, GuardDuty tiến hành phân tích mọi dữ liệu đến từ:
1. VPC Flow Logs
2. CloudTrail Logs
3. DNS Logs
   - Nhật ký lưu trữ của DNS xuất phát từ các DNS resolvers (sở hữu bởi AWS) dành cho các VPC và không thể truy xuất trực tiếp từ phía người dùng.
   - Nếu như DNS resolver được cấu hình độc lập bởi chính bạn hoặc từ phía bên thứ 3, GuardDuty sẽ không thể truy xuất, xử lý và xác định các mối nguy hại từ nguồn dữ liệu này.

GuardDuty có thể truy xuất đến mọi nguồn dữ liệu (được đề cập ở trên) cho dù chúng chưa được kích hoạt từ trước.

{{% notice info %}}
AWS khuyên bạn nên kích hoạt đồng thời CloudTrail Logs và VPC Flow Logs để có cái nhìn tổng quan và chi tiết nhất khi tiến hành phân tích dữ liệu.
{{% /notice %}}

GuardDuty là một dịch vụ mang tính chất **Regional**, thế nên để có thể theo dõi dữ liệu ở một AWS Region thì bạn phải kích hoạt ở AWS Region đó.

{{% notice info %}}
Bạn có thể kích hoạt thông qua AWS Console hoặc sử dụng APIs. Đa số người dùng sẽ kích hoạt ở mọi AWS Regions và AWS Accounts một cách đồng thời thông qua APIs.
{{% /notice %}}

Cho dù bạn có số lượng các tài nguyên AWS ít hay nhiều (ví dụ như VPCs hay IAM users), GuardDuty sẽ không gây bất kỳ ảnh hưởng nào đến bất kỳ tài nguyên nào bởi các quy trình xử lý sẽ chỉ được thực hiện nội bộ bên trong dịch vụ GuardDuty.

{{% notice info %}}
GuardDuty là một dịch vụ được quản lý hoàn toàn bởi AWS.
{{% /notice %}}

Cách tính giá của GuardDuty sẽ dựa trên
- Số lượng CloudTrail events được phân tích
- Khối lượng của VPC floư logs (theo **GB**)
- Khối lượng của DNS log (theo **GB**)

{{% notice info %}}
Mỗi tài khoản AWS sẽ có 30 ngày thử nghiệm ở mỗi AWS Region, điều này sẽ giúp GuardDuty dễ dàng dự đoán chi phí phát sinh.
{{% /notice %}}

#### Findings
GuardDuty sẽ chủ động quan sát và theo dõi các dấu hiệu bất thường xuất phát
- Từ 3 nguồn dữ liệu (được đề cập ở trên)
- Từ các EC2 instances
- Từ các tài nguyên AWS IAM

Bạn sẽ dễ dàng truy xuất chi tiết các Findings được phát hiện bởi GuardDuty ở thanh **Findings**. Mỗi Finding sẽ được chia nhỏ thành nhiều thông tin theo định dạng mà cho phép chúng ta dễ dàng đọc hiểu và xử lý các nguy cơ về bảo mật. 

```
ThreatPurpose : ResourceTypeAffected / ThreatFamilyName . ThreatFamilyVariant ! Artifact
```

{{% notice tip %}}
Tìm hiểu chi tiết hơn về các thành phần của định dạng trên [tại đây](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-format.html).
{{% /notice %}}

Đối với những trường hợp và hành vi có mức độ khó đoán cao, để mà có thể học chúng và phân tích những điểm bất thường, bộ máy Machine Learning của GuardDuty sẽ cần một khoảng thời gian cơ sở từ 7 đến 14 ngày.

**Ví dụ: **
1. Một EC2 instance bắt đầu giao tiếp với một máy chủ từ xa thông qua một Port bất thường.
2. Một IAM user bắt đầu thay đổi Route Tables trong khi trước đó chưa từng thực hiện tác vụ này.

Đối các truờng hợp trên, mọi tác vụ tìm kiếm đều sẽ được dựa trên các Signatures, vì vậy chúng sẽ được phát hiện trong vòng 10 phút kể từ khi CloudFormation stack được hoàn thành. Trong quá trình phát hiện mối nguy hại, sự chậm trễ sẽ là ít hay nhiều dựa trên tần suất xuất hiện của các thông tin liên quan và tổng thời gian mà GuardDuty có thể truy xuất và phân tích các thông tin ấy tại các nguồn dữ liệu cụ thể.

{{% notice tip %}}
Tìm hiểu chi tiết hơn về danh sách của toàn bộ các loạt GuardDuty Findings [tại đây](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)
{{% /notice %}}

Khi có bất kỳ sự thay đổi nào tại các Findings, dựa trên các thiết lập đến từ Amazon CloudWatch Events, GuardDuty sẽ gửi thông báo đến bạn ngay tức thì trong vòng 5 phút. Mọi sự thay đổi liên quan từ các lần kế tiếp sẽ có cùng ID với Finding gốc và thông báo sẽ được gửi mỗi 6 tiếng tính từ lần gửi đầu tiên, điều này nhằm đối phó với lượng thông báo đến ồ ạt trong cùng một Finding đến người dùng.
