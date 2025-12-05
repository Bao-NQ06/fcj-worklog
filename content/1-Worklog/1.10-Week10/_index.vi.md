---
title: "Worklog Tuần 10"
date: 2024-11-11
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10: 

* **Ổn định môi trường triển khai** AWS SAM/Serverless.
* **Tập trung gỡ lỗi** các vấn đề cốt lõi: CORS, lỗi vòng lặp (loop) trong `template.yaml`.
* **Tích hợp Frontend/Backend** để có thể kiểm thử chức năng hiển thị và thao tác cơ bản trên giao diện người dùng.
* **Hoàn thiện** các chức năng **Read** và **Delete** cơ bản.
* **Tham gia chuỗi sự kiện AWS Cloud Mastery Series** để nhận hướng dẫn và giải đáp thắc mắc về dự án.

---

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| :--- | :--- | :--- | :--- | :--- |
| 2 | - **Gỡ lỗi CORS:** Phân tích cấu hình CORS trong **API Gateway** và các headers phản hồi của **Lambda** để cho phép Frontend truy cập. <br> - **Khắc phục lỗi template loop:** Kiểm tra và tối ưu hóa file `template.yaml` để tránh lỗi vòng lặp khi triển khai (`sam deploy`). | 11/11/2024 | 11/11/2024 | Tài liệu API Gateway/CORS |
| 3 | - Tiếp tục củng cố chức năng **Read** (Hiển thị bài viết): Đảm bảo dữ liệu từ DynamoDB được truy vấn và trả về đúng định dạng JSON cho Frontend. | 12/11/2024 | 12/11/2024 | |
| 4 | - **Tích hợp Frontend:** Bắt đầu kết hợp mã nguồn Frontend vào dự án và kiểm thử các API Endpoint đã deploy. <br> - **Thành công hiển thị** danh sách bài viết trên giao diện. | 13/11/2024 | 13/11/2024 | Giao diện Frontend mẫu |
| 5 | - Triển khai và kiểm thử chức năng **Delete** (Xóa bài viết). <br> - **Gặp lỗi:** Phát hiện vấn đề về **xác thực (Authorization)** và **Sub ID** khi thực hiện chức năng Delete. | 14/11/2024 | 14/11/2024 | |
| 6 | - **Tham gia sự kiện AWS Cloud Mastery Series:** <br>&emsp; + Nhận hướng dẫn và giải đáp các thắc mắc về Serverless, Rekognition. <br> - Phân tích lỗi **Update/Rekognition:** Bắt đầu áp dụng hướng dẫn từ Mentor để giải quyết vấn đề xác thực và lỗi liên quan đến Rekognition. | 15/11/2024 | 15/11/2024 | Mentor, AWS Cloud Mastery Series |

---

### Kết quả đạt được tuần 10: 

* **Khắc phục thành công** lỗi CORS và ổn định quá trình triển khai SAM.
* **Tham gia chuỗi sự kiện AWS Cloud Mastery Series** và thu thập được các thông tin cần thiết để giải quyết các lỗi lớn của dự án.
* **Hoàn thành tích hợp Frontend** và Backend, đạt được giao diện người dùng đầu tiên để kiểm thử.
* **Đã triển khai thành công chức năng Read** (Hiển thị bài viết) và **Delete** (Xóa bài viết) hoạt động trên giao diện web.
* **Xác định và có hướng giải quyết** cho các điểm nghẽn quan trọng:
    * Lỗi xác thực: Lambda không lấy được/xử lý không đúng **Sub ID** từ token Cognito, ảnh hưởng đến các thao tác cần quyền.
    * Lỗi chức năng **Update**: Phụ thuộc vào luồng xử lý ảnh liên quan đến **Rekognition** (đã có hướng dẫn từ Mentor).
* Dự án đã chuyển sang giai đoạn kiểm thử người dùng cơ bản.
