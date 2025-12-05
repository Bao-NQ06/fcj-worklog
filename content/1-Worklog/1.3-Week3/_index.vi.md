---
title: "Worklog Tuần 3"
date: 2025-09-21
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Khắc phục tình trạng tài khoản và tạo lại tài khoản mới nếu cần.
* Tìm hiểu và thiết lập Hybrid DNS với Route 53 Resolver.
* Tìm hiểu và thiết lập VPC-PEERING.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
|---|---|---|---|---|
| 2 | - Tạo lại thẻ Visa và tạo lại tài khoản AWS, cũng như setup các bước ban đầu. | 21/09/2025 | 23/09/2025 | |
| 3 | - Thực hiện Lab 10 để tìm hiểu về Route 53 và thiết lập hệ thống hybrid DNS.<br>- Khởi chạy máy chủ ảo để triển khai và thiết lập DNS. | 24/09/2025 | 25/09/2025 | [Tham khảo tại đây](https://youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&si=NtlkPHvTydrkH4rK) |
| 4 | - Thiết lập kết nối VPC-PEERING để các VPC trong cùng AWS Cloud giao tiếp với nhau.<br>- Tạo các tài nguyên cần thiết cho VPC-PEERING.<br>- Dọn dẹp tài nguyên sau khi hoàn thành. | 25/09/2025 | 26/09/2025 | [Tham khảo tại đây](https://youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i&si=NtlkPHvTydrkH4rK) |
| 5 | - Tham gia họp nhóm để bàn về dự án. | 28/09/2025 | 28/09/2025 | |


### Kết quả đạt được tuần 3:

* Tạo lại thành công tài khoản AWS mới để tiếp tục học tập, Tiếp tục được đồng hành cùng FCJ team.

* Hiểu rõ về Route 53 và cách thiết lập các quy tắc (Rules):
    * Tìm hiểu và tạo Outbound Endpoint.
    * Tìm hiểu và tạo Route 53 Resolver.
    * Tìm hiểu và tạo Inbound Endpoints.
    * Thực hành và kết nối thành công với RD Gateway Server.

* Nắm được cách các VPC trong cùng một AWS Clould kết nối với nhau mà không cần đi qua Internet thông qua VPC Peering.

* Biết cách tạo tài nguyên qua CloudFormation từ template có sẵn.

* Hiểu và kích hoạt thành công tính năng Cross-Zone and Cross-Region DNS Resolution trong VPC-PEERING, cho phép một EC2 instance phân giải DNS của một EC2 instance ở VPC khác ra địa chỉ IP private.
    * Nhận biết được rằng nếu không kích hoạt tính năng này, DNS truy vấn sẽ trả về IP public và lưu lượng truy cập sẽ đi qua Internet.
* Biết được ngôn ngữ mà nhóm sẽ sử dụng và cũng như đưa dealine để mọi người trong group tự tìm hiểu thêm về ngôn ngữ