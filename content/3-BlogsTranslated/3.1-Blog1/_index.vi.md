---
title: "Bắt đầu với Amazon OpenSearch Service: T-shirt size tên miền của bạn cho phân tích nhật ký"
date: 2025-09-16
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


Ngày đăng: 2025‑09‑16 – Tác giả: Harsh Bansal, Aditya Challa, Raaga N.G trong [Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/category/analytics/amazon-opensearch-service/), [Intermediate (200)](https://aws.amazon.com/blogs/big-data/category/learning-levels/intermediate-200/),[Technical How-to](https://aws.amazon.com/blogs/big-data/category/post-types/technical-how-to/).

Khi triển khai một miền [Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/category/analytics/amazon-opensearch-service/), bạn cần xác định kích thước lưu trữ, loại và số lượng instance; quyết định chiến lược phân mảnh (sharding) và có sử dụng cluster manager hay không; đồng thời kích hoạt zone awareness. Nhìn chung, chúng ta xem xét dung lượng lưu trữ như là hướng dẫn để xác định số lượng instance, nhưng không phải các thông số khác. Ở bài viết này, chúng tôi đưa ra một số khuyến nghị dựa theo cách thức T‑shirt‑sizing cho khối lượng công việc phân tích nhật ký.

---

**Đặc điểm của phân tích nhật ký và khối lượng công việc streaming**

Khi sử dụng OpenSearch Service cho khối lượng công việc streaming, bạn gửi dữ liệu từ một hoặc nhiều nguồn vào OpenSearch Service để tạo chỉ mục do bạn định nghĩa

Dữ liệu nhật ký thường có đặc điểm chuỗi thời gian, do đó chiến lược đánh chỉ mục theo thời gian (chỉ mục theo ngày hoặc theo tuần) được khuyến nghị. Để quản lý nhật ký hiệu quả, bạn cần phải triển khai [index patterns theo thời gian](https://docs.opensearch.org/latest/dashboards/management/index-patterns/) và thiết lập thời gian lưu giữ. Bạn cũng cần xác định thêm việc [phân chia theo thời gian và thời gian lưu giữ](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ism.html) cho dữ liệu để quản lý vòng đời của nó trong miền của mình.

Để minh họa, hãy giả sử bạn có một nguồn dữ liệu tạo ra luồng log liên tục và bạn đã cấu hình chỉ mục cuộn hàng ngày với thời gian lưu giữ 3 ngày. Khi log tới, OpenSearch Service sẽ tạo ra một chỉ mục mỗi ngày với các tên như  stream1\_2025.05.21, stream1\_2025.05.22, … Với tiền tố stream1\_\* được gọi là một [**index pattern**](https://docs.opensearch.org/latest/dashboards/management/index-patterns/), tức là quy ước đặt tên giúp nhóm các chỉ mục liên quan lại với nhau.

Sơ đồ dưới đây cho thấy có ba primary shard cho mỗi chỉ mục hằng ngày. Các shard này được triển khai trên ba OpenSearch Service data instance, và mỗi primary shard có một replica tương ứng. (Để đơn giản, sơ đồ không minh họa rằng primary và replica shard luôn được đặt trên các instance khác nhau nhằm đảm bảo khả năng chịu lỗi)

![Sơ đồ primary shard và replica](/images/3-BlogsTranslated/Blog1/img1.jpg)

Khi OpenSearch Service xử lý các bản ghi nhật ký mới, chúng được gửi tới tất cả các primary shard liên quan và replica của chúng trong chỉ mục đang hoạt động, trong ví dụ này chỉ là chỉ mục của ngày hôm nay do cấu hình chỉ mục theo ngày.

![Phân phối bản ghi nhật ký](/images/3-BlogsTranslated/Blog1/img2.jpg)

Có một số đặc điểm quan trọng về cách Dịch vụ OpenSearch xử lý các mục nhập mới của bạn:

- **Tổng số shard** – Mỗi pattern của chỉ mục có tổng số shard bằng D × P × (1 \+ R), trong đó D là số ngày lưu giữ, P là số primary shard và R là số replica. Các shard này được phân phối trên các data node.  
- **Chỉ mục đang hoạt động** – Kỹ thuật time‑slicing nghĩa là các bản ghi log mới chỉ được ghi vào chỉ mục của ngày hiện tại.  
- **Sử dụng tài nguyên** – Khi gửi một yêu cầu [\_bulk](https://docs.opensearch.org/latest/api-reference/document-apis/bulk/) với các bản ghi log, chúng được phân phối tới tất cả các shard trong chỉ mục đang hoạt động. Ví dụ với ba primary shard và một replica cho mỗi shard chính, tổng cộng có sáu shard xử lý dữ liệu đồng thời, cần 6 vCPU để xử lý hiệu quả một yêu cầu \_bulk.

Tương tự, OpenSearch Service phân phối truy vấn trên các shard của các chỉ mục liên quan. Nếu bạn truy vấn pattern này trong cả 3 ngày, sẽ có 9 shard tham gia và cần 9 vCPU để xử lý yêu cầu.

![Truy vấn trên nhiều shard](/images/3-BlogsTranslated/Blog1/img3.jpg)

Điều này sẽ trở nên phức tạp hơn khi bạn bổ sung thêm nhiều data stream và index pattern. Với mỗi data stream hoặc index pattern bổ sung, bạn phải triển khai các shard cho từng chỉ mục hằng ngày và sử dụng vCPU để xử lý yêu cầu tương ứng với số shard được triển khai, như minh họa ở sơ đồ trước. Khi bạn gửi các yêu cầu đồng thời tới nhiều chỉ mục, mỗi shard của tất cả các chỉ mục liên quan đều phải xử lý những yêu cầu đó.

![Xử lý yêu cầu đồng thời](/images/3-BlogsTranslated/Blog1/img4.jpg)

---

**Dung lượng Cluster**

Khi số lượng index pattern và các yêu cầu đồng thời tăng lên, tài nguyên của cụm có thể nhanh chóng bị quá tải. OpenSearch Service bao gồm các hàng đợi nội bộ để đệm yêu cầu và giảm bớt nhu cầu xử lý đồng thời. Bạn có thể giám sát những hàng đợi này bằng API  [\_cat/thread\_pool](https://docs.opensearch.org/docs/latest/api-reference/cat/cat-thread-pool/), công cụ cho thấy độ sâu hàng đợi và giúp bạn hiểu khi nào cụm của mình đang tiến gần tới giới hạn dung lượng.

Một yếu tố phức tạp khác là thời gian xử lý các bản cập nhật và truy vấn phụ thuộc vào nội dung của chúng. Khi yêu cầu đến, hàng đợi sẽ lấp đầy theo tốc độ bạn gửi. Chúng được giải phóng theo tốc độ được quyết định bởi số vCPU, thời gian từng yêu cầu và thời gian xử lý. Bạn có thể xen kẽ nhiều yêu cầu hơn nếu các yêu cầu đó được xử lý trong vài phần nghìn giây so với trong một giây. Bạn có thể sử dụng API [\_nodes/stats](https://docs.opensearch.org/docs/latest/api-reference/nodes-apis/nodes-stats/) của OpenSearch để giám sát tải trung bình trên CPU. Để biết thêm về các giai đoạn truy vấn, hãy tham khảo bài  [A query, or There and Back Again](https://opensearch.org/blog/a-query-or-there-and-back-again/) trên blog OpenSearch.

Nếu bạn thấy độ sâu hàng đợi tăng lên, nghĩa là bạn đang bước vào “vùng cảnh báo”, nơi cụm vẫn xử lý được tải nhưng đã tới ngưỡng. Nếu tiếp tục, bạn có thể vượt quá các hàng đợi có sẵn và cần mở rộng thêm CPU. Nếu bạn bắt đầu thấy tải tăng, tương quan với độ sâu hàng đợi tăng, thì bạn cũng đang ở trong vùng "cảnh báo" và nên cân nhắc việc mở rộng quy mô.

---

## **Recommendations**

Để xác định kích thước một miền, bạn có thể tham khảo các bước sau:

* **Xác định dung lượng lưu trữ cần thiết** – Tổng lưu trữ \= (dữ liệu nguồn hàng ngày tính theo byte × 1,45) × (số replica \+ 1\) × số ngày lưu giữ. Hệ số 45 % bổ sung này gồm:  
  * 10 % cho việc kích thước chỉ mục lớn hơn dữ liệu gốc.  
  * 5 % cho overhead của hệ điều hành (Linux dành riêng cho khôi phục hệ thống và bảo vệ chống phân mảnh ổ đĩa).  
  * 20 % cho phần không gian dự phòng của OpenSearch trên mỗi instance (gộp segment, log và các thao tác nội bộ).

  * 10 % cho bộ đệm lưu trữ bổ sung (giảm thiểu ảnh hưởng của lỗi node và sự cố Availability Zone).  
* **Xác định số lượng shard** – Số lượng primary shard xấp xỉ bằng kích thước lưu trữ cần cho mỗi chỉ mục chia cho kích thước shard mong muốn. Làm tròn lên bội số gần nhất của số node dữ liệu để phân bố đều. Đối với phân tích nhật ký, lưu ý:  
  * Kích thước shard khuyến nghị: 30–50 GB.  
  * Mục tiêu tối ưu: 50 GB mỗi shard.  
* **Tính toán nhu cầu CPU** – Tỷ lệ đề xuất là 1,25 vCPU:1 cho mỗi shard với khối lượng dữ liệu nhỏ. Đối với khối lượng lớn hơn, tỷ lệ cao hơn được khuyến nghị. Mục tiêu sử dụng là trung bình 60 %, tối đa 80 %.  
* **Chọn loại instance phù hợp** – Dựa trên từng loại node:  
  * **Cluster manager nodes**: Dùng các instance dòng M của AWS Graviton (phù hợp với mọi khối lượng).  
  * **Data nodes** (khối lượng nhỏ đến lớn): Dùng instance dòng M hoặc R của AWS Graviton kết hợp Amazon Elastic Block Store (Amazon EBS).  
  * **Data nodes** (khối lượng rất lớn): Dùng instance dòng I với ổ NVMe SSD.

Ví dụ về việc xác định kích thước miền:

* Dung lượng log hàng ngày: 3 TB.  
* Thời gian lưu giữ: 3 tháng (90 ngày).  
* Số replica: 1\.

![Ví dụ tính toán kích thước miền](/images/3-BlogsTranslated/Blog1/img5.jpg)

Chúng ta thực hiện phép tính sau.

![Kết quả tính toán](/images/3-BlogsTranslated/Blog1/img6.jpg)

Bảng sau đây khuyến nghị loại instance, khối lượng dữ liệu nguồn, dung lượng lưu trữ cần thiết cho 7 ngày lưu giữ, và số lượng active shard dựa trên các hướng dẫn đã nêu:

| T-Shirt Size | Data (Per Day) | Storage Needed (with 7 days Retention) | Active Shards | Data Nodes | Primary Nodes |
| :---- | :---- | :---- | :---- | :---- | :---- |
| XSmall | 10 GB | 175 GB | 2 @ 50 GB | 3 \* r7g.large. search | 3 \* m7g.large. search |
| Small | 100 GB | 1.75 TB | 6 @ 50 GB | 3 \* r7g.xlarge. search | 3 \* m7g.large. search |
| Medium | 500 GB | 8.75 TB | 30 @ 50 GB | 6 \* r7g.2xlarge.search | 3 \* m7g.large. search |
| Large | 1 TB | 17.5 TB | 60 @ 50 GB | 6 \* r7g.4xlarge.search | 3 \* m7g.large. search |
| XLarge | 10 TB | 175 TB | 600 @ 50 GB | 30 \* i4g.8xlarge | 3 \* m7g.2xlarge.search |
| XXL | 80 TB | 1.4 PB | 2400 @ 50 GB | 87 \* I4g.16xlarge | 3 \* m7g.4xlarge.search |

Như với mọi khuyến nghị về sizing, các hướng dẫn này chỉ là điểm khởi đầu và được xây dựng dựa trên giả định. Khối lượng công việc (workload) của bạn sẽ khác, do đó nhu cầu thực tế của bạn cũng sẽ khác với các khuyến nghị này. Hãy đảm bảo rằng bạn triển khai, giám sát và điều chỉnh cấu hình khi cần.

Đối với T-shirt sizing workload, trường hợp extra-small bao gồm tối đa 10 GB dữ liệu mỗi ngày từ một data stream duy nhất tới một index pattern duy nhất. Trường hợp small nằm trong khoảng 10–100 GB dữ liệu mỗi ngày. Trường hợp medium nằm trong khoảng 100–500 GB dữ liệu mỗi ngày, và tiếp tục tăng theo các mức lớn hơn. Số lượng instance mặc định cho mỗi domain là 80 đối với hầu hết các dòng instance. Tham khảo thêm chi tiết trong mục “[Amazon OpenSearch Service quotas](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/limits.html) “.

Ngoài ra, hãy cân nhắc những biện pháp tốt nhất sau::

* Chọn đúng tầng lưu trữ (**Ultra Warm, Hot storage**) phù hợp với nhu cầu trong OpenSearch Service. Tham khảo mục [Choose the right storage tier for your needs in Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/choose-the-right-storage-tier-for-your-needs-in-amazon-opensearch-service/) để biết chi tiết.  
  Sử dụng dòng **OpenSearch Optimized (OR)** cho các workload quy mô lớn cần độ trễ thấp với các tác vụ nặng về indexing. Tham khảo bài [OpenSearch optimized instance (OR1) is game changing for indexing performance and cost](https://aws.amazon.com/blogs/big-data/opensearch-optimized-instance-or1-is-game-changing-for-indexing-performance-and-cost/) để biết chi tiết.  
* Cô lập việc ingest bằng cách sử dụng **OpenSearch Ingestion pipeline** cho các workload từ nhỏ đến lớn để giảm bớt gánh nặng vận hành khi quản lý pipeline ingestion.  
* Sử dụng **reserved instance** để tiết kiệm chi phí dài hạn.  
* Cân nhắc sử dụng **Availability Zone awareness** để đảm bảo tính sẵn sàng cao.

---

**Kết luận**

Bài viết này đã cung cấp các hướng dẫn toàn diện về việc xác định kích thước miền OpenSearch Service cho workload phân tích log, bao quát nhiều khía cạnh quan trọng. Những khuyến nghị này đóng vai trò là điểm khởi đầu vững chắc, nhưng mỗi workload đều có những đặc thù riêng. Để đạt hiệu năng tối ưu, hãy cân nhắc triển khai thêm các tối ưu hóa như data tiering và storage tier. Đánh giá các tùy chọn tiết kiệm chi phí như reserved instance, và mở rộng triển khai dựa trên các chỉ số hiệu năng thực tế cũng như độ sâu hàng đợi. Bằng cách tuân theo các hướng dẫn này và chủ động giám sát hệ thống, bạn có thể xây dựng một miền OpenSearch Service hoạt động hiệu quả, đáp ứng nhu cầu phân tích log trong khi vẫn duy trì tính hiệu quả và tiết kiệm chi phí.

---

| ![Harsh Bansal](/images/3-BlogsTranslated/Blog1/author.png) | Harsh Bansal [Harsh](https://www.linkedin.com/in/hbansal-analytics/) là một Analytics và AI Solutions Architect tại Amazon Web Services. Bansal cộng tác chặt chẽ với khách hàng, giúp đỡ họ  chuyển đổi lên các nền tảng đám mây và tối ưu hóa thiết lập các cụm để giúp khách hàng tăng hiệu suất và tiết kiệm chi phí. Trước khi tham gia AWS, Bansal hỗ trợ khách hàng tận dụng OpenSearch và Elasticsearch cho các yêu cầu phân tích nhật ký và tìm kiếm đa dạng. |
| :---- | :---- |
| ![Aditya Challa](/images/3-BlogsTranslated/Blog1/author2.jpg) | **Aditya Challa** Aditya là Senior Solutions Architect tại Amazon Web Services. Aditya yêu thích việc hỗ trợ khách hàng trong suốt hành trình AWS của họ vì anh ấy hiểu rằng hành trình luôn tuyệt vời hơn khi có người đồng hành. Anh ấy rất yêu thích du lịch, lịch sử, những kỳ quan kỹ thuật và học hỏi điều mới mẻ mỗi ngày. |
| ![Raaga NG](/images/3-BlogsTranslated/Blog1/author3.png) | **Raaga NG** Raaga là Solutions Architect tại Amazon Web Services. Raaga là một chuyên gia công nghệ với hơn 5 năm kinh nghiệm chuyên về Phân tích. Raaga tâm huyết với việc giúp khách hàng AWS định hướng hành trình chuyển đổi lên đám mây. |
