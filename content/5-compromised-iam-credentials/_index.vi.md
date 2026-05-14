---
title : "Compromised IAM credentials"
date: 2024-01-01
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

**Tình huống 2: Compromised IAM credentials**

Bạn đã hoàn thành tình huống giả lập tấn công đầu tiên và quay trở lại với tách cà phê của mình. Tuy nhiên, bạn lại tiếp tục nhận thêm những thông báo mới về những Findings liên quan tới các dịch vụ AWS IAM. Nội dung thông báo đầu tiên chỉ ra rằng, bằng cách sử dụng **IAM credentials**, một số **API calls** đã được thực hiện từ địa chỉ IP đã được thêm vào **Threat List** (ở bài trước).

{{% notice note %}}
Không có bất kỳ **IAM credentials** cá nhân nào đã từng bị phơi bày hay lộ ra dưới bất kỳ hình thức nào.
{{% /notice %}}

**Nội dung**
- [Kiến Trúc Tổng Quan](#kiến-trúc-tổng-quan)
- [Quá Trình Điều Tra](#quá-trình-điều-tra)
- [Cấu Hỏi Ôn Tập](#cấu-hỏi-ôn-tập)

#### Kiến Trúc Tổng Quan
![architecture-overview](/images/5-architecture-overview.png?featherlight=false&width=60pc)

1. *EC2 malicious instance* này tiến hành thực hiện các **API calls**, EIP của instance này đã được thêm vào **Threat List**. Nội dung các **API calls** đã được lưu lại trong nhật ký của CloudTrail.
2. GuardDuty quan sát nhật ký của **CloudTrail Logs** cùng với **VPC Flow Logs** và **DNS Logs**, qua đó đánh giá tình hình dựa trên một số cơ sở nhất định.
3. GuardDuty tạo ra những Findings tương ứng và đồng thời gửi chi tiết đến GuardDuty Console và EventBridge Events.
4. **EventBridge Event Rule** tiến hành kích hoạt **SNS Topic**.
5. **SNS Topic** tiến hành gửi thông báo E-mail cùng với thông tin liên quan.

#### Quá Trình Điều Tra

---

**Truy cập GuardDuty Console**

Để tiến hành xem xét các Findings:
1. Truy cập vào GuardDuty Console ở **us-west-2**
2. Chúng ta sẽ thấy được các Findings với định dạng như sau. 
   1.  `Recon:IAMUser`.
   2.  `UnauthorizedAccess:IAMUser`.

![guardduty-finding-recon-iamuser](/images/5-guardduty-findings.png?featherlight=false&width=90pc)

3. Nếu không có bất kỳ Finding nào, tiến hành nhấn nút Refresh và đợi.
4. Từ Finding - `Recon:IAMUser/MaliciousIPCaller.Custom`, chúng ta có thể dễ dàng truy xuất một số thông tin sau:
   1. Chuyện gì đã xảy ra?
   2. Tài nguyên AWS nào bị ảnh hưởng?
   3. Sự kiện này xảy ra khi nào?
5. Ở dưới phần **Resource Affected**, bạn sẽ kiếm được `User Name` mà liên quan đến Finding này.

![guardduty-finding-recon-iamuser-affected-resources](/images/5-guardduty-finding-recon-iamuser-affected-resources.png?featherlight=false&width=90pc)

> Dựa trên định dạng đã được xem xét chi tiết ở phần trước, bạn có thể xác định chính xác sự cố bảo mật nào thông qua kiểu Finding?

Finding này chỉ ra rằng **IAM credential** của `User Name` trên đã có thể bị phơi bày bởi những **API calls** xuất phát từ địa chỉ IP mà trước đó đã được thêm vào **Threat List**.

> Vậy những hành động nào đã được IAM User này thực hiện?

Ở mục **Action**, chúng ta thấy được hành động `DescribeParameters` đã được thực hiện.

![guardduty-finding-recon-iamuser-action](/images/5-guardduty-finding-recon-iamuser-action.png?featherlight=false&width=90pc)

> Làm thế nào để chúng ta có thể thấy được hết các hành động còn lại, được thực hiện bởi IAM User này, trong vòng 1 tiếng trước hay 1 ngày truớc?

GuardDuty có khả năng phân tích một lượng lớn dữ liệu nhằm xác định chính xác mối nguy hiểm có mặt trong môi trường của bạn. Tuy nhiên, trong quá trình điều tra và các bước thực hiện Remediation, chúng ta cũng cần kết hợp nhiều nguồn dữ liệu khác nhau để có một cái nhìn tương quan nhất có thể.

Trong trường hợp này, nhà phân tích có thể sử dụng các thông tin chi tiết có thể tìm thấy trong nhật ký hành vi người dùng thông qua **CloudTrail**.

![cloudtrail-event-history](/images/5-cloudtrail-event-history.png?featherlight=false&width=90pc)

{{% notice note %}}
Những Findings về IAM được sinh ra bởi *EC2 malicious instance* đã thực hiện những **API calls** và EIP của instance này nằm trong **Custom Threat List**.
{{% /notice %}}

---

**Kiểm tra EventBridge Event Rule**

1. Truy cập vào CloudWatch Console ở **us-west-2**.
2. Ở thanh điều hướng bên tay trái, dưới **Events**, chọn **Rules**. Bạn sẽ thấy có 3 quy tắc đã được thiết lập (bởi CloudFormation Template), bắt đầu với tiền tố có dạng sau `GuardDuty-Event.`.
3. Tiến hành chọn quy tắc có tên là `GuardDuty-Event-IAMUser-MaliciousIPCaller`.

![eventbridge-event-iam-malicious-ip-caller](/images/5-eventbridge-event-iam-malicious-ip-caller.png?featherlight=false&width=90pc)

4. Bạn sẽ dễ dàng nhận thấy chỉ có 1 mục tiêu tại vùng **Targets** là **SNS Topic**.

![eventbridge-event-iam-malicious-ip-caller-targets](/images/5-eventbridge-event-iam-malicious-ip-caller-targets.png?featherlight=false&width=90pc)

Hoá ra, Alice chưa từng thiết lập Lambda Function để thực hiện quá trình Remediation bởi đội ngũ Security đã đưa ra quyết định rằng họ sẽ tiến hành một cách thủ công đối với Finding này.

> Sự kết hợp giữa **GuardDuty và EventBridge Events** mang lại sự uyển chuyển giúp cho chúng ta dễ dàng tạo ra một quy trình Remediation tự động hoá. Lambda function hay giải pháp đến từ phía [đối tác của AWS](https://aws.amazon.com/guardduty/resources/partners/) là những lựa chọn hàng đầu.

Đối với một số Finding nhất định, chúng ta có thể sẽ chỉ cấu hình thông báo và giải quyết vấn đề một cách thủ công, thay vì tự động hoá. Bởi khi thiết kế quá trình tự động hoá, chúng ta sẽ phải vô cùng lưu ý và đánh giá kết quả mà quá trình Remediation mang lại, bao gồm cả ưu điểm và nhược điểm.

> Ngoài ra, chúng ta có thể thiết lập **Target** đối một số tài nguyên AWS khác như **SSM Run Commands** hay **Step Function State Machine**.

---

**Giải quyết tình hình**

Bởi Alice chưa từng thiết lập quá trình Remediation đối với Finding này, chúng ta cần phải thực hiện một cách thủ công. Trong khi đội ngũ Security đang tiến hành phân tích những hành vi của IAM user này để xác định rõ hơn phạm vi lỗ hổng, chúng ta cần phải thực hiện một số bước để vô hiệu hoá **Access Key** nhằm ngăn chặn lập tức những hành động tiếp theo.
1. Truy cập vào IAM Console.
2. Ở thanh điều hướng bên tay trái, chọn **Users**.

![iam-users](/images/5-iam-users.png?width=90pc)

3. Dựa trên GuardDuty Finding và thông báo E-mail, chúng ta dễ dàng chọn ra được IAM user - `GuardDuty-Example-Compromised-Simulated`.
4. Ở user `GuardDuty-Example-Compromised-Simulated`, chúng ta chọn thanh **Security Credentials**.

![iam-users-compromised-simulated-credential](/images/5-iam-users-compromised-simulated-credential.png?featherlight=false&width=90pc)

5. Ở mục **Access Keys**, dựa trên thông tin **Access Key ID** từ Finding, chúng ta tiến hành nhấn nút `Make Inactive`.

![iam-users-compromised-simulated-credential-deactivate](/images/5-iam-users-compromised-simulated-credential-deactivate.png?featherlight=false&width=90pc)

#### Cấu Hỏi Ôn Tập
1. Nguồn dữ liệu nào đã được GuardDuty sử dụng để xác định mối nguy hại này?
2. Những Permissions nào mà IAM user được cho phép sử dụng?
3. Tại sao đội ngũ Security quyết định đi ngược lại với quá trình Remediation tự động hoá?