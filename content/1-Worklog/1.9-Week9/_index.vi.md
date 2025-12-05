---
title: "Worklog Tuần 9"
date: 2024-11-04
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9: 

* Hoàn tất quá trình chuyển đổi sang mô hình phát triển **AWS SAM (Serverless Application Model)**.
* **Tái cấu trúc (Refactor)** và triển khai lại các chức năng CRUD cơ bản theo cấu trúc SAM.
* Giải quyết các vấn đề liên quan đến môi trường để đạt được trạng thái **Deploy thành công** lên AWS.
* Tích hợp **Docker** để chuẩn hóa môi trường `sam build`.

---

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| 2 | - **Nghiên cứu chuyên sâu về AWS SAM:** Tìm hiểu cấu trúc `template.yaml`, cách thức hoạt động của các tài nguyên Serverless (Lambda, API Gateway) trong mô hình SAM. <br> - Lập kế hoạch chi tiết cho việc chuyển đổi mã nguồn đã có sang cấu trúc SAM. | 04/11/2024 | 04/11/2024 | Tài liệu AWS SAM, AWS Study Group |
| 3 | - **Tái cấu trúc mã nguồn:** Bắt đầu viết lại các chức năng CRUD cơ bản (tạo/cập nhật bài viết) theo khuôn mẫu của SAM (Handlers và Event Triggers). <br> - **Tích hợp Docker:** Cài đặt và cấu hình Docker để đảm bảo phiên bản Python phù hợp cho quá trình `sam build`. | 05/11/2024 | 06/11/2024 | Tài liệu Docker, SAM CLI |
| 4 | - **Gỡ lỗi và kiểm thử Local:** Thực hiện `sam local invoke` và `sam local start-api`. <br> - **Gặp sự cố nghiêm trọng** trong môi trường Local (Lỗi dependency, xung đột môi trường, vấn đề kết nối DynamoDB local). | 06/11/2024 | 07/11/2024 | Báo cáo lỗi SAM CLI, Stack Overflow |
| 5 | - **Ra quyết định chiến lược:** Team Backend quyết định chuyển sang chiến lược **deploy-then-test** (triển khai rồi kiểm thử) lên môi trường AWS thật để vượt qua các rào cản gỡ lỗi Local, chấp nhận rủi ro cao. <br> - Tập trung khắc phục các lỗi cấu hình trong `template.yaml` để chuẩn bị cho `sam deploy`. | 07/11/2024 | 08/11/2024 | |
| 6 | - **Triển khai thành công:** Thực hiện `sam deploy --guided` và cuối cùng đã triển khai dự án lên môi trường AWS. <br> - **Xác thực cơ bản:** Kiểm tra các API Endpoint đã được tạo và hoạt động, chứng minh chức năng CRUD đã online. | 08/11/2024 | 08/11/2024 | Log triển khai AWS CloudFormation |

---

### Kết quả đạt được tuần 9: 

* **Hoàn thành việc chuyển đổi** công nghệ sang mô hình phát triển **AWS SAM** cho toàn bộ dự án.
* **Tái cấu trúc thành công** các chức năng CRUD cơ bản vào cấu trúc Serverless của SAM.
* Đã giải quyết được vấn đề môi trường bằng cách sử dụng **Docker** để đảm bảo quá trình `sam build` diễn ra chính xác với phiên bản Python yêu cầu.
* **Đạt được cột mốc quan trọng:** Deploy thành công dự án lên môi trường AWS, vượt qua các trục trặc gỡ lỗi Local.
* **Dự án Travel-Guided** đã có phiên bản API hoạt động trên môi trường Cloud thực tế (mặc dù vẫn cần kiểm thử sâu hơn).
* ...