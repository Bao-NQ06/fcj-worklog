---
title: "Get started with Amazon OpenSearch Service: T-shirt size your domain for log analytics"
date: 2025-09-16
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

Published: 2025‑09‑16 – Authors: Harsh Bansal, Aditya Challa, Raaga N.G in [Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/category/analytics/amazon-opensearch-service/), [Intermediate (200)](https://aws.amazon.com/blogs/big-data/category/learning-levels/intermediate-200/),[Technical How-to](https://aws.amazon.com/blogs/big-data/category/post-types/technical-how-to/).

When deploying an [Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/category/analytics/amazon-opensearch-service/) domain, you need to determine storage size, instance type and quantity; decide on sharding strategy and whether to use cluster manager; and enable zone awareness. Generally, we consider storage capacity as a guide to determine the number of instances, but not other parameters. In this article, we provide some recommendations based on T‑shirt‑sizing methodology for log analytics workloads.

---

**Characteristics of log analytics and streaming workloads**

When using OpenSearch Service for streaming workloads, you send data from one or more sources into OpenSearch Service to create indices that you define

Log data typically has time-series characteristics, so time-based indexing strategies (daily or weekly indices) are recommended. To manage logs effectively, you need to implement [time-based index patterns](https://docs.opensearch.org/latest/dashboards/management/index-patterns/) and set retention periods. You also need to determine [time-based partitioning and retention](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/ism.html) for data to manage its lifecycle in your domain.

To illustrate, suppose you have a data source that generates a continuous log stream and you have configured daily rolling indices with a 3-day retention period. When logs arrive, OpenSearch Service will create one index per day with names like stream1\_2025.05.21, stream1\_2025.05.22, … The prefix stream1\_\* is called an [**index pattern**](https://docs.opensearch.org/latest/dashboards/management/index-patterns/), which is a naming convention that helps group related indices together.

The diagram below shows three primary shards for each daily index. These shards are deployed across three OpenSearch Service data instances, and each primary shard has a corresponding replica. (For simplicity, the diagram does not illustrate that primary and replica shards are always placed on different instances to ensure fault tolerance)

![Primary shard and replica diagram](/images/3-BlogsTranslated/Blog1/img1.jpg)

When OpenSearch Service processes new log records, they are sent to all relevant primary shards and their replicas in the active index, which in this example is only today's index due to the daily index configuration.

![Log record distribution](/images/3-BlogsTranslated/Blog1/img2.jpg)

There are several important characteristics about how OpenSearch Service handles your new entries:

- **Total number of shards** – Each index pattern has a total number of shards equal to D × P × (1 \+ R), where D is the number of retention days, P is the number of primary shards, and R is the number of replicas. These shards are distributed across data nodes.  
- **Active index** – The time‑slicing technique means new log records are only written to the current day's index.  
- **Resource usage** – When sending a [\_bulk](https://docs.opensearch.org/latest/api-reference/document-apis/bulk/) request with log records, they are distributed to all shards in the active index. For example, with three primary shards and one replica for each primary shard, there are a total of six shards processing data concurrently, requiring 6 vCPUs to efficiently process a \_bulk request.

Similarly, OpenSearch Service distributes queries across shards of related indices. If you query this pattern across all 3 days, 9 shards will participate and 9 vCPUs will be needed to process the request.

![Querying across multiple shards](/images/3-BlogsTranslated/Blog1/img3.jpg)

This becomes more complex when you add more data streams and index patterns. With each additional data stream or index pattern, you must deploy shards for each daily index and use vCPUs to process requests corresponding to the number of deployed shards, as illustrated in the previous diagram. When you send concurrent requests to multiple indices, each shard of all related indices must process those requests.

![Concurrent request processing](/images/3-BlogsTranslated/Blog1/img4.jpg)

---

**Cluster Capacity**

As the number of index patterns and concurrent requests increases, cluster resources can quickly become overloaded. OpenSearch Service includes internal queues to buffer requests and reduce concurrent processing demands. You can monitor these queues using the [\_cat/thread\_pool](https://docs.opensearch.org/docs/latest/api-reference/cat/cat-thread-pool/) API, a tool that shows queue depth and helps you understand when your cluster is approaching capacity limits.

Another complicating factor is that the processing time for updates and queries depends on their content. When requests arrive, queues fill at the rate you send them. They are released at a rate determined by the number of vCPUs, per-request time, and processing time. You can interleave more requests if those requests are processed in milliseconds rather than seconds. You can use the OpenSearch [\_nodes/stats](https://docs.opensearch.org/docs/latest/api-reference/nodes-apis/nodes-stats/) API to monitor average CPU load. For more information about query stages, refer to the [A query, or There and Back Again](https://opensearch.org/blog/a-query-or-there-and-back-again/) post on the OpenSearch blog.

If you see queue depth increasing, you are entering a "warning zone," where the cluster can still handle the load but has reached a threshold. If it continues, you may exceed available queues and need to scale up CPU. If you start seeing load increase, correlated with increasing queue depth, you are also in the "warning zone" and should consider scaling.

---

## **Recommendations**

To determine the size of a domain, you can follow these steps:

* **Determine required storage capacity** – Total storage \= (daily source data in bytes × 1.45) × (number of replicas \+ 1\) × retention days. This additional 45% factor includes:  
  * 10% for index size being larger than source data.  
  * 5% for operating system overhead (Linux reserved for system recovery and disk defragmentation protection).  
  * 20% for OpenSearch overhead on each instance (segment merging, logs, and internal operations).

  * 10% for additional storage buffer (minimizing the impact of node failures and Availability Zone incidents).  
* **Determine number of shards** – The number of primary shards is approximately equal to the storage size needed per index divided by the desired shard size. Round up to the nearest multiple of the number of data nodes for even distribution. For log analytics, note:  
  * Recommended shard size: 30–50 GB.  
  * Optimal target: 50 GB per shard.  
* **Calculate CPU requirements** – The recommended ratio is 1.25 vCPU:1 per shard for small data volumes. For larger volumes, a higher ratio is recommended. Target usage is average 60%, maximum 80%.  
* **Choose appropriate instance type** – Based on node type:  
  * **Cluster manager nodes**: Use AWS Graviton M-series instances (suitable for all volumes).  
  * **Data nodes** (small to large volumes): Use AWS Graviton M or R-series instances combined with Amazon Elastic Block Store (Amazon EBS).  
  * **Data nodes** (very large volumes): Use I-series instances with NVMe SSD drives.

Example of domain sizing:

* Daily log capacity: 3 TB.  
* Retention period: 3 months (90 days).  
* Number of replicas: 1\.

![Domain sizing calculation example](/images/3-BlogsTranslated/Blog1/img5.jpg)

We perform the following calculation.

![Calculation results](/images/3-BlogsTranslated/Blog1/img6.jpg)

The following table recommends instance types, source data volumes, storage capacity needed for 7 days retention, and number of active shards based on the guidelines mentioned:

| T-Shirt Size | Data (Per Day) | Storage Needed (with 7 days Retention) | Active Shards | Data Nodes | Primary Nodes |
| :---- | :---- | :---- | :---- | :---- | :---- |
| XSmall | 10 GB | 175 GB | 2 @ 50 GB | 3 \* r7g.large. search | 3 \* m7g.large. search |
| Small | 100 GB | 1.75 TB | 6 @ 50 GB | 3 \* r7g.xlarge. search | 3 \* m7g.large. search |
| Medium | 500 GB | 8.75 TB | 30 @ 50 GB | 6 \* r7g.2xlarge.search | 3 \* m7g.large. search |
| Large | 1 TB | 17.5 TB | 60 @ 50 GB | 6 \* r7g.4xlarge.search | 3 \* m7g.large. search |
| XLarge | 10 TB | 175 TB | 600 @ 50 GB | 30 \* i4g.8xlarge | 3 \* m7g.2xlarge.search |
| XXL | 80 TB | 1.4 PB | 2400 @ 50 GB | 87 \* I4g.16xlarge | 3 \* m7g.4xlarge.search |

As with all sizing recommendations, these guidelines are only a starting point and are built on assumptions. Your workload will differ, so your actual needs will also differ from these recommendations. Make sure you deploy, monitor, and adjust configuration as needed.

For T-shirt sizing workloads, the extra-small case includes up to 10 GB of data per day from a single data stream to a single index pattern. The small case ranges from 10–100 GB of data per day. The medium case ranges from 100–500 GB of data per day, and continues to increase at larger levels. The default number of instances per domain is 80 for most instance families. See more details in "[Amazon OpenSearch Service quotas](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/limits.html)".

Additionally, consider the following best practices:

* Choose the right storage tier (**Ultra Warm, Hot storage**) that fits your needs in OpenSearch Service. See [Choose the right storage tier for your needs in Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/choose-the-right-storage-tier-for-your-needs-in-amazon-opensearch-service/) for details.  
  Use the **OpenSearch Optimized (OR)** family for large-scale workloads requiring low latency with heavy indexing tasks. See [OpenSearch optimized instance (OR1) is game changing for indexing performance and cost](https://aws.amazon.com/blogs/big-data/opensearch-optimized-instance-or1-is-game-changing-for-indexing-performance-and-cost/) for details.  
* Isolate ingestion by using **OpenSearch Ingestion pipeline** for small to large workloads to reduce operational burden when managing ingestion pipelines.  
* Use **reserved instances** for long-term cost savings.  
* Consider using **Availability Zone awareness** to ensure high availability.

---

**Conclusion**

This article has provided comprehensive guidance on sizing OpenSearch Service domains for log analytics workloads, covering many important aspects. These recommendations serve as a solid starting point, but each workload has its own unique characteristics. To achieve optimal performance, consider implementing additional optimizations such as data tiering and storage tiers. Evaluate cost-saving options like reserved instances, and scale deployments based on actual performance metrics and queue depth. By following these guidelines and proactively monitoring your system, you can build an efficient OpenSearch Service domain that meets log analytics needs while maintaining efficiency and cost-effectiveness.

---

| ![Harsh Bansal](/images/3-BlogsTranslated/Blog1/author.png) | Harsh Bansal [Harsh](https://www.linkedin.com/in/hbansal-analytics/) is an Analytics and AI Solutions Architect at Amazon Web Services. Bansal works closely with customers, helping them migrate to cloud platforms and optimize cluster setups to help customers increase performance and save costs. Before joining AWS, Bansal helped customers leverage OpenSearch and Elasticsearch for diverse log analytics and search requirements. |
| :---- | :---- |
| ![Aditya Challa](/images/3-BlogsTranslated/Blog1/author2.jpg) | **Aditya Challa** Aditya is a Senior Solutions Architect at Amazon Web Services. Aditya enjoys supporting customers throughout their AWS journey because he understands that journeys are always better with companions. He loves traveling, history, technical marvels, and learning something new every day. |
| ![Raaga NG](/images/3-BlogsTranslated/Blog1/author3.png) | **Raaga NG** Raaga is a Solutions Architect at Amazon Web Services. Raaga is a technology expert with over 5 years of experience specializing in Analytics. Raaga is passionate about helping AWS customers navigate their cloud transformation journey. |
