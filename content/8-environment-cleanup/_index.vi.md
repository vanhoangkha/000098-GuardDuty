---
title : "Dọn dẹp tài nguyên"
date: 2024-01-01
weight : 8
chapter : false
pre : " <b> 8. </b> "
---
Để tiến hành dọn dẹp các tài nguyên AWS được dùng trong bài thực hành này, chúng ta lần lượt thực hiện các bước sau:

1. Xoá S3 bucket mà được tạo bởi CloudFormation Template với định dạng `guardduty-example`. Bước này bắt buộc phải thực hiện bởi CloudFormation Stack sẽ không thể bị xoá nếu S3 bucket mà tồn tại dữ liệu bên trong.
2. Xoá IAM Role mà được dùng cho EC2 compromised instance với định dạng `GuardDuty-Example-EC2-Compromised`. Bởi trong một quá trình Remediation, Lambda Function đã tiến hành thêm một IAM Policy, CloudFormation Stack sẽ không thể bị xoá nếu IAM Role xuất hiện thêm các IAM Policy khác.
3. Xoá **Custom Threat List** được sử dụng cho GuardDuty.
4. Vô hiệu hoá dịch vụ Amazon GuardDuty.
   1. Truy cập vào GuardDuty Console.
   2. Nhân vào mục **Settings**.
   3. Chọn `Disable GuardDuty` và nhấn `Save`.
5. Xoá CloudFormation Stack. Nếu bạn không thể xoá được, hãy kiểm tra lại bước 2 bước đầu tiên.