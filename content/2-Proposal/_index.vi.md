---
title: "Bản đề xuất"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Team Super Beast Warrior(SBW)

# Batch-based Clickstream Analytics Platform

### 1. Tóm tắt điều hành

Dự án này nhằm mục tiêu thiết kế và triển khai một **Batch-based Clickstream Analytics Platform** cho một trang web thương mại điện tử chuyên về máy tính và phụ kiện, sử dụng AWS Cloud Services. Hệ thống thu thập dữ liệu tương tác của người dùng (như clicks, searches, và page visits) từ website và lưu trữ chúng trong **Amazon S3** dưới dạng raw logs. Cứ mỗi giờ, **Amazon EventBridge** sẽ kích hoạt **AWS Lambda** functions để xử lý và chuyển đổi dữ liệu trước khi tải chúng vào **data warehouse** được triển khai trên **Amazon EC2**.

Dữ liệu đã được xử lý sẽ được trực quan hóa thông qua **R Shiny dashboards**, cung cấp cho chủ cửa hàng các thông tin chi tiết về kinh doanh như mô hình hành vi khách hàng, mức độ phổ biến của sản phẩm, và xu hướng tương tác trên website.

Kiến trúc này tập trung vào **batch analytics**, **ETL pipelines**, và **business intelligence**, đồng thời đảm bảo **bảo mật (security)**, khả năng **mở rộng (scalability)**, và **hiệu quả chi phí (cost efficiency)** thông qua việc tận dụng AWS managed services.

### 2. Tuyên bố vấn đề

### Vấn đề hiện tài là gì?

Các E-commerce websites tạo ra một lượng lớn **clickstream data** — bao gồm product views, cart actions, và search activities — chứa đựng nhiều business insights có giá trị.

Tuy nhiên, các cửa hàng **nhỏ và vừa (SMEs)** thường thiếu hạ tầng (infrastructure) và chuyên môn kỹ thuật (expertise) để thu thập, xử lý và phân tích dữ liệu này một cách hiệu quả.

Do đó, họ gặp khó khăn trong việc:

- Hiểu hành vi mua hàng của khách hàng
- Xác định sản phẩm hoạt động hiệu quả nhất
- Tối ưu hóa chiến dịch marketing và hiệu suất website
- Đưa ra quyết định về tồn kho và định giá dựa trên dữ liệu

### Giải pháp

Dự án này giới thiệu một **AWS-based batch clickstream analytics** hệ thống có khả năng tự động thu thập dữ liệu tương tác của người dùng từ website mỗi giờ, xử lý dữ liệu thông qua serverless functions, và lưu trữ vào central data warehouse trên Amazon EC2.

Kết quả sau khi xử lý được trực quan hóa bằng **R Shiny dashboards**, giúp chủ cửa hàng thu được actionable insights về hành vi khách hàng (customer behavior) và nâng cao hiệu suất kinh doanh tổng thể (overall business performance).

### Lợi ích và hoàn vốn đầu tư

- **Data-driven decision making**: Khai thác customer preferences, popular products, và shopping trends để hỗ trợ ra quyết định dựa trên dữ liệu.
- **Scalable and modular design**: Thiết kế có khả năng mở rộng và mô-đun hóa, dễ dàng mở rộng để xử lý nhiều users hơn hoặc tích hợp thêm data sources mới.
- **Cost-efficient batch processing**: Giảm chi phí tính toán liên tục (continuous compute costs) bằng cách vận hành theo lịch trình định kỳ hàng giờ (scheduled, hourly basis).
- **Business insight enablement**: Trao quyền cho store owners tối ưu hóa sales strategies và cải thiện revenue thông qua evidence-based analytics.

### 3. Kiến trúc giải pháp

Nền tảng áp dụng kiến trúc AWS Serverless để quản lý dữ liệu từ 5 trạm dựa trên Raspberry Pi, có thể mở rộng lên 15 trạm. Dữ liệu được tiếp nhận qua AWS IoT Core, lưu trữ trong S3 data lake và xử lý bởi AWS Glue Crawlers và ETL jobs để chuyển đổi và tải vào một S3 bucket khác cho mục đích phân tích. Lambda và API Gateway xử lý bổ sung, trong khi Amplify với Next.js cung cấp bảng điều khiển được bảo mật bởi Cognito.(cần bổ sung)

![Architecture](/static/images/2-Proposal/AWS_Architecture.jpg)

### Dịch vụ AWS sử dụng

- **Amazon Cognito**: Xử lý user authentication và authorization cho cả administrators và website customers, đảm bảo truy cập an toàn vào e-commerce platform.
- **Amazon S3**: Đóng vai trò là lớp lưu trữ dữ liệu tập trung (centralized data storage layer) — lưu trữ static website front-end và raw clickstream logs được thu thập từ user interactions. Ngoài ra, S3 còn tạm thời lưu trữ các batch files trước khi chúng được xử lý và chuyển đến data warehouse.
- **Amazon CloudFront**: Phân phối nội dung website tĩnh (static website content) trên toàn cầu với độ trễ thấp (low latency), cải thiện trải nghiệm người dùng và lưu cache tài nguyên gần với khách hàng.
- **Amazon API Gateway**: Đóng vai trò là điểm vào chính (main entry point) cho các API calls từ website, cho phép gửi dữ liệu một cách an toàn (như clickstream hoặc browsing activity) vào hệ thống AWS.
- **AWS Lambda**: Thực thi serverless functions để tiền xử lý (preprocess) và tổ chức (organize) dữ liệu clickstream được tải lên S3. Lambda cũng xử lý các tác vụ chuyển đổi dữ liệu theo lịch trình (scheduled data transformation jobs) được EventBridge kích hoạt, trước khi tải vào data warehouse.
- **Amazon EventBridge**: Lên lịch và điều phối (schedules and orchestrates) các batch workflows — ví dụ, kích hoạt Lambda functions mỗi giờ để xử lý và di chuyển clickstream data từ S3 vào EC2 data warehouse.
- **Amazon EC2 (Data Warehouse)**: Đóng vai trò là môi trường data warehouse, chạy PostgreSQL hoặc một relational database khác để phục vụ batch analytics, trend analysis, và business reporting.
- **R Shiny (on EC2)**: Lưu trữ các interactive dashboards giúp trực quan hóa các insights đã được xử lý theo batch, hỗ trợ doanh nghiệp khám phá hành vi khách hàng (customer behavior), sản phẩm phổ biến (popular products), và cơ hội bán hàng (sales opportunities).
- **AWS IAM**: Quản lý quyền truy cập (access permissions) và chính sách (policies) nhằm đảm bảo rằng chỉ những người dùng và thành phần AWS được ủy quyền mới có thể tương tác với data và services.
- **Amazon CloudWatch**: Thu thập và giám sát (collects and monitors) metrics, logs, và trạng thái công việc theo lịch trình (scheduled job statuses) từ Lambda và EC2, giúp duy trì độ tin cậy của hệ thống (system reliability) và hiển thị hiệu suất (performance visibility).
- **Amazon SNS**: Gửi thông báo hoặc cảnh báo (notifications or alerts) khi batch jobs hoàn tất, thất bại, hoặc gặp lỗi, đảm bảo nhận biết tình trạng vận hành kịp thời (timely operational awareness).

_Thiết kế thành phần_

- _Thiết bị biên_: Raspberry Pi thu thập và lọc dữ liệu cảm biến, gửi tới IoT Core.
- _Tiếp nhận dữ liệu_: AWS IoT Core nhận tin nhắn MQTT từ thiết bị biên.
- _Lưu trữ dữ liệu_: Dữ liệu thô lưu trong S3 data lake; dữ liệu đã xử lý lưu ở một S3 bucket khác.
- _Xử lý dữ liệu_: AWS Glue Crawlers lập chỉ mục dữ liệu; ETL jobs chuyển đổi để phân tích.
- _Giao diện web_: AWS Amplify lưu trữ ứng dụng Next.js cho bảng điều khiển và phân tích thời gian thực.
- _Quản lý người dùng_: Amazon Cognito giới hạn 5 tài khoản hoạt động.

### 4. Triển khai kỹ thuật

#### End-to-end data flow

1. Auth (Cognito).

- Trình duyệt (Browser) thực hiện xác thực (authenticate) với Amazon Cognito thông qua Hosted UI hoặc JS SDK.
- ID token (JWT) được lưu trong bộ nhớ (stored in memory); SDK sẽ tự động đính kèm (attach) header `Authorization: Bearer <JWT>` cho các API calls.

2. Static web (CloudFront + S3).

- SPA/assets được host trên S3; CloudFront nằm phía trước với OAC, gzip/brotli, HTTP/2, và WAF managed rules.
- Trang web sẽ tải một analytics SDK nhỏ gọn có nhiệm vụ thu thập các sự kiện (events) và gửi chúng đến API Gateway (phía dưới).

3. Event ingest (API Gateway).

- `POST /v1/events` (HTTP API). CORS được giới hạn (locked) theo site origin; JWT authorizer sẽ xác thực (validate) Cognito token (hoặc API key đối với anonymous flows).
- Các requests sau đó được chuyển tiếp (forwarded) đến Lambda để xử lý.

4. Security & Ops.

- IAM được cấu hình theo nguyên tắc least-privilege cho từng component.
- CloudWatch theo dõi logs, metrics, và alarms liên quan đến API 5xx, Lambda errors, throttles, và Shiny health.
- SNS sẽ gửi thông báo (notify) khi có alarms hoặc khi DLQ tăng trưởng bất thường (DLQ growth).

5. Processing & storage (Lambda → DynamoDB(datalake) → PostgreSQL on EC2(data warehouse)→ Shiny).

- Lambda thực hiện xác thực (validate) và làm giàu dữ liệu (enrich) các events, sau đó ghi vào DynamoDB (các bảng session/event).
- Một ETL job nhỏ chạy trên EC2 sẽ định kỳ (periodically) nén và tổng hợp (compact/aggregate) dữ liệu từ DynamoDB vào curated store (Postgres hoặc DuckDB) trên EC2 data-warehouse node.
- R Shiny Server (trên EC2) sẽ đọc các curated tables và render dashboards dành cho admins.

#### Data Contracts & Governance

**Event JSON (ingest)**

```yaml
{
  "event_id": "uuid-v4",
  "ts": "2025-10-18T12:34:56.789Z",
  "event_type": "view|click|search|add_to_cart|checkout|purchase",
  "session_id": "uuid-v4",
  "user_id": "cognito-sub-or-null",
  "anonymous_id": "stable-anon-id",
  "page_url": "https://site/p/123",
  "referrer": "https://google.com",
  "device": { "type": "mobile|desktop|tablet" },
  "geo": { "country": "VN", "city": null },
  "ecom":
    {
      "product_id": "sku-123",
      "category": "Shoes",
      "currency": "USD",
      "price": 79.99,
      "qty": 1,
    },
  "props": { "search_query": "running shoes" },
}
```

- **PII**: không bao giờ gửi name/email/phone; mọi optional identifier (nếu có) sẽ được hash trong Lambda.
- **Behaviour**: tạo anonymous_id một lần, duy trì session_id (tự động roll sau 30 phút không hoạt động); gửi dữ liệu bằng navigator.sendBeacon với fetch retry fallback; tùy chọn offline buffer thông qua IndexedDB.

**S3 raw layout & retention**

- **Bucket**: `s3://clickstream-raw/`
- **Object format**: NDJSON, tùy chọn GZIP.
- **Partitioning**: `year=YYYY/month=MM/day=DD/hour=HH/ → events-<uuid>.ndjson.gz`
- **Optional manifest per batch**: bao gồm processed watermark, object list, record counts, và hash.
- **Lifecycle**: raw → (30 days Standard/IA) → (365+ days Glacier/Flex).
- **Idempotency**: duy trì một compact staging table trong PostgreSQL (hoặc một small S3 key-value manifest) để theo dõi (track) last processed object/batch và ngăn chặn việc tải trùng (prevent double-load).

#### Frontend SDK (Static site on S3 + CloudFront)

**Instrumentation**

- Một JS snippet nhỏ được tải trên toàn bộ website (site-wide) với thuộc tính defer.
- Đoạn mã này tạo `anonymous_id` một lần và lưu `session_id` trong `localStorage`; session sẽ tự động roll sau 30 phút không hoạt động.
- Các events được gửi qua navigator.sendBeacon; nếu thất bại, fallback sang `fetch` với retry & jitter.

**Auth context**

- Nếu người dùng đăng nhập bằng Cognito, hãy bao gồm trường `user_id = idToken.sub` để kích hoạt (enable) các logged-in funnels.

**Offline durability**

- Optional Service Worker queue: khi offline, tạm lưu (buffer) các events trong IndexedDB và gửi lại (flush) khi kết nối được khôi phục (on reconnect).

#### Ingestion API (API Gateway → Lambda)

**API Gateway (HTTP API)**

- Route: `POST /v1/events`.
- JWT authorizer: sử dụng Cognito user pool. Đối với các anonymous pre-login events, dùng API key usage-plan với rate limits nghiêm ngặt (strict rate limits).
- WAF: sử dụng AWS Managed Core + Bot Control; chặn (block) các non-site origins thông qua strict CORS.

**Lambda (Node.js or Python)**

- Validate: kiểm tra dữ liệu dựa trên JSON Schema (sử dụng ajv/pydantic).
- Idempotency: kiểm tra `event_id` gần đây trong bảng DynamoDB nhỏ hoặc in-memory cache có TTL để tránh xử lý trùng lặp.
- Enrichment: trích xuất `date/hour`, phân tích UA (User-Agent), và suy luận country từ `CloudFront-Viewer-Country` (nếu có).
- Persist: thực hiện `PutItem` vào bảng `events`, và `UpdateItem` để cập nhật `sessions.last_seen` cùng `page_count`.
- Failure path: publish các sự kiện lỗi lên SQS DLQ; gửi alarm qua SNS nếu DLQ depth > 0.

#### Batch Buffer (S3)

- Purpose: cung cấp bộ đệm (buffer) rẻ tiền, bền vững cho batch analytics.
- Write pattern: ghi các object nhỏ cho từng request hoặc micro-batches (ví dụ: 1–5 MB mỗi object) với GZIP. Optional compactor có thể gộp thành các file ≥64MB để tăng hiệu quả đọc (efficient reads).
- Read pattern: ETL Lambda chỉ scan các partition/object mới kể từ last watermark.
- Schema-on-read: ETL áp dụng schema khi đọc, đồng thời xử lý dữ liệu đến muộn (late-arriving data) bằng cách reprocess một cửa sổ trượt nhỏ (ví dụ: 2 giờ gần nhất) để chỉnh sửa sessions.

#### EC2 “data warehouse” node

Purpose: chạy ETL và host curated analytical store mà Shiny sẽ truy vấn. Có hai lựa chọn:

- Postgres trên EC2 (khuyến nghị nếu team thích SQL/window functions)
  - Instance: t3.small / t4g.small; gp3 50–100GB
  - Schema: `fact_events`, `fact_sessions`, `dim_date`, `dim_product`
  - Security: đặt trong VPC private subnet; truy cập thông qua ALB / SSM Session Manager; tự động snapshot hàng ngày vào S3
- ETL (Lambda, batch qua EventBridge cron)
  - Trigger: chạy theo chu kỳ rate(5 minutes) hoặc cron(...) tùy thuộc vào cost và freshness (mức độ cập nhật dữ liệu).
  - Các bước: liệt kê các S3 objects mới → đọc dữ liệu → validate/dedupe (kiểm tra hợp lệ và loại bỏ trùng lặp) → transform (làm phẳng cấu trúc nested JSON, ép kiểu dữ liệu, thêm ingest_date, session_window_start/end) → upsert vào Postgres bằng COPY vào temp tables rồi merge, hoặc thực hiện batched INSERT ... ON CONFLICT.
  - Networking: Lambda được gắn với VPC private subnets để có thể truy cập EC2 Postgres security group.

#### R Shiny Server on EC2 (admin analytics)

**Server**

- EC2 (t3.small/t4g.small) với các thành phần: R 4.4+, Shiny Server (open-source), Nginx reverse proxy, TLS thông qua ACM/ALB hoặc Let’s Encrypt.
- IAM instance profile (không dùng static keys). Security group cho phép HTTPS từ office/VPN hoặc Cognito-gated admin site.

**App (packages)**

- Các package R sử dụng: `shiny`, `shinydashboard/bslib`, `plotly`, `DT`, `dplyr`, `DBI` + `RPostgres` hoặc `duckdb`, `lubridate`.
- Nếu truy vấn DynamoDB trực tiếp cho các small cards, có thể sử dụng `paws.dynamodb` (tùy chọn).

**Dashboards**

- **Traffic & Engagement**: DAU/MAU, sessions, avg pages, bounce proxy.
- **Funnels**: view → add_to_cart → checkout → purchase với tỷ lệ chuyển đổi từng stage (stage conversion) và drop-off.
- **Product Performance**: views, CTR, ATC rate, revenue theo product/category.
- **Acquisition**: referrer, campaign, device, country.
- **Reliability**: Lambda error rate, DLQ depth, ETL lag, data freshness.

**Caching**

- Query results được cache trong quá trình xử lý (in-process) thông qua reactive values hoặc materialized bởi ETL; các cache keys được tạo dựa trên date range và filters.

#### Security baseline

**IAM**

- Ingest Lambda: quyền s3:PutObject tới raw bucket (giới hạn theo prefix), s3:ListBucket trên các prefix cần thiết.
- ETL Lambda: quyền s3:GetObject/ListBucket trên các raw prefixes; có quyền lấy secrets từ SSM Parameter Store; không có quyền truy cập rộng vào S3.
- EC2 roles: chỉ read/write tới DB/volumes riêng của nó; tùy chọn read S3 cho backups.
- Shiny EC2: không ghi vào S3 raw; chỉ read-only tới Postgres khi cần.

**Network**

- Đặt EC2 trong private subnets; truy cập công khai qua ALB (HTTPS 443).
- Lambda thực hiện ETL được join vào VPC để kết nối tới Postgres; Security Group (SG) áp dụng least-privilege (chỉ mở Postgres port từ ETL SG).
- Không mở rộng 0.0.0.0/0 tới các DB ports.

**Data**

- Mã hóa: EBS bằng KMS, S3 server-side encryption, RDS/PG TLS, secrets lưu trong SSM Parameter Store.
- Dữ liệu nhạy cảm: không có PII trong events; retention: raw S3 90–365 ngày (lifecycle), curated Postgres theo chính sách kinh doanh.

#### Observability & alerting

**CloudWatch metrics/alarms**

- Theo dõi: API Gateway 5xx/latency, Lambda errors/throttles, DLQ depth, DynamoDB throttles, ETL job exit code/lag, Shiny health check.

**SNS topics**: gửi email/SMS/Slack webhook cho on-call team.
**Structured logs**: JSON logs từ Lambda & ETL (bao gồm request_id, event_type, status, ms, error_code).
**Watermark tracking**: custom metric “DW Freshness (minutes since last successful upsert)”.

#### Cost Controls (stay near Free/low tier)

- HTTP API được sử dụng (chi phí thấp hơn), Lambda memory tối thiểu (256–512MB), nén requests.
- Batch thay vì realtime: dùng S3 làm buffer để loại bỏ chi phí ghi/đọc DynamoDB.
- S3 lifecycle: Standard → Standard-IA/Intelligent-Tiering → Glacier cho dữ liệu raw cũ; bật GZIP để giảm chi phí lưu trữ và truyền tải.
- Điều chỉnh cadence ETL (ví dụ: 15–60 phút) và chỉ xử lý các object mới; gộp các file nhỏ thành file lớn để giảm read I/O.
- Single small EC2 cho Shiny + DW ban đầu; sau đó scale vertically hoặc tách riêng khi cần.
- AWS Budgets với SNS alerts cho chi phí thực tế và dự báo.

#### Deliverables

- Analytics SDK (TypeScript): hỗ trợ sessionization, beacon, và optional offline queue.
- API/Lambda (ingest): xử lý validation, enrichment, idempotency hints, và DLQ.
- S3 raw bucket spec: prefixing/partitioning, compression, lifecycle, kèm optional compactor.
- ETL Lambda (batch): kết hợp với EventBridge cron, watermarking, và upsert strategy vào PostgreSQL
- PostgreSQL schema: fact_events, fact_sessions, các dims, cùng indexes và vacuum/maintenance plan.
- R Shiny dashboard app: gồm 5 modules, triển khai với Nginx/ALB TLS setup.
- Runbook: bao gồm alarms, on-call, backups, disaster recovery, freshness SLO, và cost guardrails.

### 5. Lộ trình & Mốc triển khai

### Dự án theo tiến độ

#### Tháng 1 – Học tập & Chuẩn bị

Nghiên cứu một loạt các AWS services bao gồm compute, storage, analytics, và security. Hiểu các khái niệm chính về kiến trúc đám mây (cloud architecture), data pipelines, và serverless computing. Tổ chức các cuộc họp nhóm để đồng bộ mục tiêu dự án và phân công nhiệm vụ.

#### Tháng 2 – Thiết kế kiến trúc & Prototyping

Thiết kế kiến trúc tổng thể của dự án và xác định luồng dữ liệu giữa các thành phần. Thiết lập các AWS resources ban đầu như S3, Lambda, API Gateway, EventBridge, và EC2. Thử nghiệm với các công cụ mã nguồn mở để trực quan hóa và báo cáo. Kiểm tra sample code và xác thực pipeline thu thập và xử lý dữ liệu.

#### Tháng 3 – Triển khai & Kiểm thử

Triển khai toàn bộ kiến trúc dựa trên thiết kế đã được phê duyệt. Tích hợp tất cả các dịch vụ AWS và đảm bảo độ tin cậy của hệ thống. Thực hiện kiểm thử hiệu năng và chức năng. Hoàn thiện tài liệu và chuẩn bị dự án cho buổi trình bày.

### 6. Ước tính ngân sách

Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)  
Hoặc tải [tệp ước tính ngân sách](../attachments/budget_estimation.pdf).

### Chi phí hạ tầng

- AWS Services

  - **Amazon Cognito (User Pools)**: 0.10 USD/tháng(1 monthly active user (MAU), 1 MAU đăng nhập qua SAML hoặc OIDC federation)

  - **Amazon S3**

    - S3 Standard: 0.17 USD/tháng (6 GB, 1,000 PUT requests, 1,000 GET requests, 6 GB Data returned, 6 GB Data scanned)
    - Data Transfer: 0.00 USD/tháng (Outbound 6 TB, Inbound 6 TB)

  - **Amazon CloudFront (United States)**: 0.64 USD/tháng(6 GB Data transfer out to internet, 6 GB Data transfer out to origin, 10,000 HTTPS requests)

  - **Amazon API Gateway (HTTP APIs)**: 0.01 USD/tháng(10,000 HTTP API requests units)

  - **Amazon Lambda (Service settings)**: 0.00 USD/tháng(1,000,000 requests, 512 MB memory)

  - **Amazon CloudWatch (APIs)**: 0.03 USD/tháng(100 metrics GetMetricData, 1,000 metrics GetMetricWidgetImage, 1,000 API requests)

  - **Amazon SNS (Service settings)**: 0.02 USD/tháng(1,000,000 requests, 100,000 HTTP/HTTPS Notifications, 1,000 EMAIL/EMAIL-JSON Notifications, 100,000,000 QS Notifications, 100,000,000 Lambda deliveries, 100,000 Kinesis Data Firehose notifications)

  - **Amazon EC2 (EC2 specifications)**: 1.68 USD/tháng(1 instance, 730 Compute Savings Plans)

  - **Amazon EventBridge**: 0.00 USD/tháng(1,000,000 events (AWS management events - EventBridge Event Bus Ingestion))

Tổng cộng: 2.65 USD/tháng, 31.8 USD/12 tháng

### 7. Đánh giá rủi ro

| Risk                                                                                                                             | Likelihood | Impact | Mitigation Strategy                                                                                                                                                                                                      |
| -------------------------------------------------------------------------------------------------------------------------------- | ---------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Chi phí cao vượt quá ngân sách ước tính                                                                                          | Medium     | High   | Theo dõi chặt chẽ và tính toán tất cả các chi phí tiềm năng trên AWS. Giới hạn việc sử dụng các dịch vụ AWS có chi phí cao và thay thế bằng các giải pháp đơn giản, tiết kiệm chi phí nhưng cung cấp chức năng tương tự. |
| Các vấn đề tiềm ẩn trong việc truyền dữ liệu hoặc tích hợp dịch vụ giữa các thành phần AWS                                       | Medium     | Medium | Thực hiện step-by-step validation trước khi triển khai chính thức. Thử nghiệm sớm, sử dụng các managed AWS services, và liên tục giám sát hiệu suất thông qua Amazon CloudWatch.                                         |
| Rủi ro trong thu thập hoặc xử lý dữ liệu (ví dụ: tương tác người dùng quá mức, mạng không ổn định, thiếu hoặc trùng lặp sự kiện) | High       | Medium | Áp dụng data validation, temporary buffering, và schema enforcement để đảm bảo tính nhất quán. Sử dụng structured logging và alarms để phát hiện và xử lý lỗi khi ingest dữ liệu.                                        |
| Người dùng ít hoặc không sử dụng analytics dashboard                                                                             | Low        | High   | Tổ chức các buổi internal training và tận dụng các communication channels hiện có để nâng cao nhận thức. Khuyến khích việc sử dụng bằng cách trình bày các practical benefits và actionable insights của hệ thống.       |

### 8. Kết quả kỳ vọng

#### Hiểu Hành Vi và Hành Trình Khách Hàng

Hệ thống ghi lại toàn bộ hành trình khách hàng — bao gồm các trang mà người dùng truy cập, sản phẩm mà họ xem, thời gian họ ở lại, và điểm họ rời khỏi trang web.

Bằng cách phân tích session duration, bounce rate, và navigation paths, doanh nghiệp có thể đánh giá mức độ tương tác của người dùng và trải nghiệm tổng thể.

Điều này cung cấp nền tảng dữ liệu đáng tin cậy để cải thiện giao diện website, tối ưu bố cục trang, và nâng cao overall customer satisfaction.

#### Xác Định Sản Phẩm Phổ Biến và Xu Hướng Người Tiêu Dùng

Dựa trên clickstream data được thu thập và xử lý trên AWS, hệ thống xác định các sản phẩm được xem nhiều nhất và mua nhiều nhất.

Các sản phẩm ít được chú ý cũng được theo dõi, cho phép doanh nghiệp đánh giá hiệu quả của product listings, điều chỉnh giá cả hoặc hình ảnh sản phẩm, và lập kế hoạch tồn kho hiệu quả hơn.

Hơn nữa, hệ thống hỗ trợ phát hiện shopping trends theo khoảng thời gian, khu vực, hoặc loại thiết bị — giúp đưa ra quyết định kinh doanh kịp thời và dựa trên dữ liệu.

#### Tối Ưu Chiến Lược Marketing và Bán Hàng

Dữ liệu hành vi của khách hàng được transform (chuyển đổi) thành business insights (thông tin kinh doanh chuyên sâu) và được trình bày thông qua R Shiny dashboards.

Với các kết quả phân tích này, doanh nghiệp có thể:

- Xác định chính xác target customer segments cho các nỗ lực marketing
- Tùy chỉnh các chiến dịch quảng cáo và khuyến mãi cho các nhóm sản phẩm hoặc đối tượng khách hàng cụ thể
- Đánh giá hiệu quả của các sáng kiến marketing thông qua các chỉ số tương tác và chuyển đổi có thể đo lường

Kết quả là, marketing and sales strategies trở nên dựa trên bằng chứng và chính xác hơn, hỗ trợ ra quyết định tốt hơn và cải thiện hiệu suất kinh doanh.
