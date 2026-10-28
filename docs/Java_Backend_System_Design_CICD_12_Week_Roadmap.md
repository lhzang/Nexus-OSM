# Lộ trình Java Backend, System Design, CI/CD và High Availability

## Kế hoạch 12 tuần --- 2 giờ/ngày --- Học, code, kiểm thử và viết documentation

> **Mục tiêu:** xây dựng một Order Management System (OMS) đủ thực tế để
> luyện Java/Spring Boot, MySQL, Redis, multithreading, Kafka,
> WebSocket, MinIO, Docker, CI/CD, System Design, HAProxy, Keepalived,
> MySQL replication và Kubernetes.
>
> **Nhịp học:** 6 ngày/tuần × 2 giờ/ngày = 12 giờ/tuần; 12 tuần ≈ 144
> giờ. Ngày thứ 7 nghỉ hoặc dùng làm ngày bù, không cộng thêm vào kế
> hoạch mặc định.
>
> **Nguyên tắc:** không chỉ xem video/đọc tài liệu. Mỗi buổi phải tạo ra
> ít nhất một đầu ra kiểm chứng được: code, test, sơ đồ, benchmark, ADR
> hoặc ghi chú giải thích.

------------------------------------------------------------------------

## 0. Cách sử dụng lộ trình

### Cấu trúc cố định cho mỗi buổi 120 phút

  -----------------------------------------------------------------------
  Thời gian               Việc cần làm            Quy tắc
  ----------------------- ----------------------- -----------------------
  0--10 phút              Ôn chủ động             Đóng tài liệu, tự viết
                                                  lại 3--5 ý của buổi
                                                  trước

  10--35 phút             Học lý thuyết           Chỉ học nội dung phục
                                                  vụ mục tiêu của buổi

  35--90 phút             Code/thực hành          Tự viết, chạy, quan
                                                  sát; tránh chỉ
                                                  copy-paste

  90--105 phút            Test/debug/đo đạc       Có bằng chứng là code
                                                  hoạt động

  105--120 phút           Documentation           Cập nhật README, nhật
                                                  ký học, sơ đồ hoặc ADR
  -----------------------------------------------------------------------

Nếu buổi đó cần nhiều thời gian setup, có thể dùng 10 phút ôn tập + 20
phút học + 70 phút thực hành + 20 phút ghi chép. **Không bỏ phần
documentation**; giảm phạm vi bài tập thay vì bỏ ghi chép.

### Quy tắc làm việc mỗi ngày

1.  Mở issue hoặc ghi rõ mục tiêu: "Hôm nay hoàn thành X".
2.  Chia mục tiêu thành một thay đổi nhỏ có thể kiểm tra.
3.  Tạo branch Git cho feature hoặc dùng commit nhỏ trên branch cá nhân.
4.  Chạy test và ghi lại kết quả thực tế, không ghi kết quả dự kiến như
    thể đã chạy.
5.  Nếu bị kẹt quá 20 phút: đọc error message, tạo ví dụ nhỏ nhất tái
    hiện lỗi, tra tài liệu chính thức, ghi lại nguyên nhân.
6.  Kết thúc buổi bằng một commit có thông điệp rõ ràng và cập nhật
    `docs/learning-log.md`.

### Cách xác định "đã học xong"

Một chủ đề chỉ được đánh dấu hoàn thành khi m có thể: - Giải thích bằng
lời cơ chế hoạt động, không chỉ định nghĩa. - Viết được một demo nhỏ mà
không cần chép nguyên bài hướng dẫn. - Nêu ít nhất hai lỗi/thất bại có
thể xảy ra. - Viết test hoặc thử nghiệm chứng minh hành vi. - Ghi lại
trade-off, giới hạn và cách debug.

------------------------------------------------------------------------

# 1. Phạm vi dự án xuyên suốt

## 1.1. Sản phẩm cần xây

**Tên:** `backend-lab-oms`

OMS là một hệ thống quản lý sản phẩm, tồn kho, đơn hàng và trạng thái xử
lý. Bắt đầu bằng **modular monolith** trong một ứng dụng Spring Boot;
chỉ tách worker/consumer khi có lý do rõ ràng. Không xây microservices
ngay từ ngày đầu.

### Nghiệp vụ tối thiểu

-   Quản lý sản phẩm: tạo, xem, cập nhật giá, cập nhật tồn kho.
-   Tạo đơn có nhiều dòng sản phẩm.
-   Kiểm tra tồn kho và tránh bán vượt số lượng.
-   Chuyển trạng thái đơn hợp lệ.
-   Tra cứu lịch sử trạng thái đơn.
-   Cache dữ liệu đọc thường xuyên bằng Redis.
-   Publish event khi đơn được tạo; consumer xử lý tác vụ bất đồng bộ
    qua Kafka.
-   Gửi cập nhật trạng thái đến client qua WebSocket.
-   Upload hóa đơn hoặc file minh họa lên MinIO.
-   Chạy nhiều API instance sau reverse proxy/load balancer.
-   Build/test/package/deploy qua CI/CD.
-   Triển khai bản local lên Kubernetes.
-   Thử nghiệm replication/HA trong môi trường local hoặc VM khi đủ thời
    gian.

### Những thứ chưa cần làm

-   Không cần frontend đầy đủ; dùng curl, Postman hoặc Swagger UI.
-   Không cần cloud trả phí; ưu tiên Docker Compose, kind/Minikube và VM
    local.
-   Không cần triển khai mọi thành phần production-grade trong 12 tuần.
-   Không coi một demo local là bằng chứng cho HA production.
-   Không thêm công nghệ chỉ để "có trong CV"; phải ghi rõ vấn đề nó
    giải quyết.

## 1.2. Stack gợi ý

-   Java 21 LTS hoặc phiên bản Java được dự án/lead yêu cầu.
-   Spring Boot phiên bản stable tương thích với Java đã chọn.
-   Maven.
-   MySQL 8.x.
-   Redis.
-   Apache Kafka ở chế độ KRaft cho môi trường học.
-   MinIO/S3-compatible object storage.
-   WebSocket với Spring.
-   Docker Engine + Docker Compose.
-   GitHub Actions (hoặc GitLab CI nếu nhóm dùng GitLab).
-   HAProxy, Keepalived trên Linux/VM khi có thể.
-   Kubernetes local qua kind hoặc Minikube.
-   JUnit 5, Mockito, Testcontainers (nếu máy đủ tài nguyên), k6 hoặc
    JMeter.
-   Git, curl, Postman/Bruno, IDE, SQL client.

**Quan trọng:** kiểm tra compatibility matrix và tài liệu chính thức
trước khi pin version. Ghi version thực tế trong `docs/environment.md`;
không dùng tag `latest` cho image mà m muốn tái lập chính xác.

## 1.3. Cấu trúc repository đề xuất

``` text
backend-lab-oms/
├── README.md
├── pom.xml
├── Dockerfile
├── compose.yaml
├── .env.example
├── .gitignore
├── .github/
│   └── workflows/
│       └── ci.yml
├── src/
│   ├── main/java/com/example/oms/
│   │   ├── product/
│   │   ├── inventory/
│   │   ├── order/
│   │   ├── payment/
│   │   ├── notification/
│   │   ├── shared/
│   │   └── OmsApplication.java
│   └── test/java/com/example/oms/
├── infra/
│   ├── mysql/
│   ├── redis/
│   ├── kafka/
│   ├── minio/
│   ├── haproxy/
│   ├── keepalived/
│   └── k8s/
├── load-tests/
├── scripts/
└── docs/
    ├── requirements.md
    ├── architecture/
    │   ├── context.md
    │   ├── containers.md
    │   └── diagrams/
    ├── database/
    │   ├── schema.md
    │   └── query-benchmarks.md
    ├── adr/
    ├── api/
    ├── runbooks/
    ├── test-reports/
    ├── incidents/
    ├── learning-log.md
    └── weekly-reviews/
```

Có thể điều chỉnh package theo quy ước team. Đây là gợi ý để học, không
phải cấu trúc bắt buộc duy nhất.

## 1.4. Definition of Done cho mọi feature

-   [ ] Code compile thành công.
-   [ ] Có test cho happy path và ít nhất một edge case.
-   [ ] Lỗi được xử lý có chủ đích; không nuốt exception.
-   [ ] API/contract hoặc hành vi được mô tả trong docs.
-   [ ] README/config hướng dẫn chạy đã được cập nhật nếu cần.
-   [ ] Không commit secret, password thật, token hay file `.env` cá
    nhân.
-   [ ] Có commit nhỏ, message mô tả thay đổi.
-   [ ] Nếu thay đổi kiến trúc, thêm ADR hoặc cập nhật sơ đồ.
-   [ ] Nếu tuyên bố cải thiện performance, có phép đo trước/sau và ghi
    môi trường đo.

------------------------------------------------------------------------

# 2. Lịch 12 tuần tổng quan

  -----------------------------------------------------------------------
  Tuần                    Chủ đề chính            Đầu ra bắt buộc
  ----------------------- ----------------------- -----------------------
  1                       Java Core, OOP, Git,    Java domain model, unit
                          Maven                   tests, repo chạy được

  2                       Spring Boot, REST,      CRUD API + CI tự chạy
                          validation, CI cơ bản   test

  3                       SQL, schema,            Schema và benchmark
                          transaction, index      query

  4                       Query tuning,           Báo cáo `EXPLAIN` và
                          partition, locking      concurrency test

  5                       Thread, thread pool,    Demo concurrency và
                          race condition          test chứng minh

  6                       Redis, cache,           Cache-aside + đo
                          idempotency, System     latency
                          Design fundamentals     

  7                       Kafka và event-driven   Producer/consumer,
                          design                  retry, idempotency

  8                       WebSocket, MinIO,       Real-time status +
                          reliability patterns    upload file

  9                       Docker, Compose,        Hệ thống local chạy
                          networking, reverse     bằng một lệnh
                          proxy                   

  10                      CI/CD đầy đủ, HAProxy,  Pipeline
                          load test               build/image/deploy test

  11                      MySQL replication,      Replication/failover
                          Keepalived, HA concepts lab và runbook

  12                      Kubernetes,             K8s deployment +
                          observability, System   architecture pack
                          Design tổng kết         
  -----------------------------------------------------------------------

Lộ trình 12 tuần là **bản tối thiểu có chủ đích**. Nếu một chủ đề chưa
đạt Definition of Done, hãy kéo dài chủ đề đó; không cần chạy đua để
tick đủ công nghệ.

------------------------------------------------------------------------

# 3. Kế hoạch thực hành từng ngày

Mỗi ngày dưới đây tương ứng 2 giờ. "Docs" là đầu ra phải ghi vào
repository, không chỉ ghi nhớ trong đầu.

## Tuần 1 --- Java Core, OOP, Git và thiết kế domain

**Mục tiêu tuần:** viết domain model có nghiệp vụ, hiểu
OOP/Collections/exception và tạo repo có test.

### Ngày 1 --- Setup môi trường và Git

-   **Học (25 phút):** kiểm tra Java/JDK, Maven, Git; hiểu repository,
    commit, branch, `.gitignore`.
-   **Làm (65 phút):**
    1.  Tạo repository `backend-lab-oms`.
    2.  Tạo Maven project Java.
    3.  Tạo `README.md`, `.gitignore`, `docs/environment.md`.
    4.  Ghi version JDK, Maven, OS, IDE.
    5.  Tạo branch `feature/domain-model`.
-   **Test (15 phút):** chạy `mvn test`; kiểm tra clone/build được từ
    thư mục sạch nếu làm được.
-   **Docs (15 phút):** README ghi mục tiêu dự án, yêu cầu cài đặt và
    lệnh chạy.
-   **Done:** project compile; không commit file build output hoặc
    secret.

### Ngày 2 --- Class, encapsulation, immutability

-   **Học:** class/object, constructor, access modifier, encapsulation,
    `final`, immutable object.
-   **Làm:** tạo `Product`, `OrderItem`, `Order`, `Money` (nếu phù hợp);
    không cho code bên ngoài sửa trực tiếp state không hợp lệ.
-   **Test:** thử số lượng âm, giá âm, đơn không có item; quyết định
    trường hợp nào phải bị từ chối.
-   **Docs:** `docs/learning-log.md`: phân biệt entity, value object và
    DTO bằng ví dụ trong project.
-   **Done:** invariant nghiệp vụ được bảo vệ ở domain layer, không chỉ
    ở Controller.

### Ngày 3 --- Interface, polymorphism, composition

-   **Học:** abstraction, interface, composition over inheritance, SOLID
    ở mức thực dụng.
-   **Làm:** thiết kế `PaymentStrategy` với `CashPayment` và
    `BankTransferPayment` giả lập; tránh tạo abstraction không có nhu
    cầu thực.
-   **Test:** thêm một phương thức thanh toán mới mà không sửa logic
    tính tổng tiền đơn.
-   **Docs:** tạo ADR ngắn: "Vì sao chọn strategy/interface cho
    payment?" Ghi cả điểm bất lợi.
-   **Done:** giải thích được polymorphism và lý do chọn thiết kế.

### Ngày 4 --- Collections, Generics, equals/hashCode

-   **Học:** `List`, `Set`, `Map`, `HashMap`, generic, equals/hashCode,
    BigDecimal.
-   **Làm:** tính tổng đơn bằng `BigDecimal`; tạo lookup sản phẩm theo
    ID; loại item trùng theo quy tắc nghiệp vụ.
-   **Test:** giá trị tiền lẻ, item trùng, ID không tồn tại; không dùng
    `double` cho tiền.
-   **Docs:** bảng ngắn so sánh List/Set/Map và trường hợp dùng trong
    OMS.
-   **Done:** chọn cấu trúc dữ liệu có lý do và có test.

### Ngày 5 --- Exceptions, Stream API, Optional

-   **Học:** checked/unchecked exception, exception propagation, Stream
    API, `Optional`.
-   **Làm:** viết các hàm tìm sản phẩm, tính tổng, lọc đơn theo trạng
    thái; dùng exception nghiệp vụ có ý nghĩa.
-   **Test:** input rỗng, dữ liệu không tìm thấy, lỗi không bị che giấu.
-   **Docs:** ghi quy tắc exception: exception nào là validation, nghiệp
    vụ, hạ tầng.
-   **Done:** không dùng `Optional.get()` mù quáng; không catch rồi bỏ
    qua lỗi.

### Ngày 6 --- Ôn tập và mini-review

-   **Ôn (15 phút):** tự giải thích bốn trụ cột OOP, immutable/mutable,
    equals/hashCode.
-   **Làm (65 phút):** hoàn thiện domain model, refactor tên biến, loại
    logic lặp.
-   **Test (20 phút):** chạy toàn bộ unit tests; thêm ít nhất 5 test
    case cho domain.
-   **Docs (20 phút):** tạo `docs/weekly-reviews/week-01.md` gồm: đã
    làm, lỗi khó nhất, 3 câu hỏi tự trả lời, việc còn thiếu.
-   **Done tuần:** domain layer có test; repo có README và learning log.

------------------------------------------------------------------------

## Tuần 2 --- Spring Boot, REST API, validation, testing và CI đầu tiên

**Mục tiêu tuần:** chạy API thực sự, hiểu request flow và có CI tự động
build/test.

### Ngày 1 --- Spring IoC/DI và cấu trúc ứng dụng

-   **Học:** IoC, dependency injection, Spring Bean,
    Controller/Service/Repository.
-   **Làm:** tạo Spring Boot app; tổ chức package `product`, `order`,
    `shared`; inject dependency qua constructor.
-   **Test:** ứng dụng khởi động; kiểm tra bean wiring và một service
    test.
-   **Docs:** sơ đồ luồng request: Controller → Service → Repository.
-   **Done:** không dùng `new` để tự tạo service dependency trong
    controller.

### Ngày 2 --- REST API và HTTP

-   **Học:** HTTP methods/status codes, REST resource, idempotency ở mức
    API, pagination cơ bản.
-   **Làm:** endpoint sản phẩm: create, get by ID, list, update; dùng
    DTO request/response.
-   **Test:** 201/200/400/404 phù hợp; kiểm tra response shape.
-   **Docs:** `docs/api/products.md` có endpoint, request/response mẫu
    và error codes.
-   **Done:** API contract rõ ràng, không trả entity JPA trực tiếp.

### Ngày 3 --- Validation và error handling

-   **Học:** Bean Validation, `@Valid`, `@ControllerAdvice`, error
    response thống nhất.
-   **Làm:** validate tên, giá, số lượng; map lỗi thành response có mã
    lỗi và message phù hợp.
-   **Test:** request sai định dạng, thiếu field, ID không tồn tại.
-   **Docs:** bảng lỗi API và cách client xử lý.
-   **Done:** lỗi đầu vào không làm app trả 500 ngoài ý muốn.

### Ngày 4 --- JPA/Hibernate và MySQL local

-   **Học:** entity, repository, persistence context, transaction cơ
    bản, migration.
-   **Làm:** kết nối MySQL qua Docker Compose; tạo entity/repository cho
    Product; dùng Flyway hoặc công cụ migration được chọn.
-   **Test:** integration test cho tạo và đọc sản phẩm.
-   **Docs:** `docs/database/schema.md` mô tả bảng, cột, key, quan hệ.
-   **Done:** schema có thể tạo lại bằng migration; cấu hình qua
    environment variables.

### Ngày 5 --- Unit test, integration test, Maven

-   **Học:** JUnit 5, Mockito, test slice và integration test khác nhau
    thế nào.
-   **Làm:** unit test service và integration test repository/controller
    phù hợp; không mock mọi thứ một cách máy móc.
-   **Test:** chạy `mvn test`; cố tình làm một test fail để xác nhận
    build fail.
-   **Docs:** `docs/testing.md` ghi lệnh chạy test và phạm vi từng loại
    test.
-   **Done:** test chạy lặp lại được.

### Ngày 6 --- CI cơ bản bằng GitHub Actions

-   **Học:** CI là gì, trigger, job, step, runner, artifact.
-   **Làm:** tạo `.github/workflows/ci.yml` để checkout code, setup JDK,
    cache Maven nếu phù hợp, chạy `mvn verify`.
-   **Test:** push branch, xem log workflow; tạo thay đổi gây test fail
    và xác nhận pipeline fail.
-   **Docs:** README hiển thị trạng thái CI nếu phù hợp; ghi cách xử lý
    pipeline fail.
-   **Done tuần:** pull request/push có pipeline build/test tự động.

------------------------------------------------------------------------

## Tuần 3 --- MySQL, data modeling và transaction

**Mục tiêu tuần:** thiết kế schema có lý do, hiểu ACID và chứng minh
nghiệp vụ đặt hàng đúng trong trường hợp bình thường.

### Ngày 1 --- SQL và quan hệ

-   **Học:** primary/foreign key, normalization, one-to-many, join.
-   **Làm:** thiết kế `products`, `orders`, `order_items`; viết
    migration.
-   **Test:** insert dữ liệu; query đơn hàng cùng các dòng sản phẩm;
    kiểm tra FK.
-   **Docs:** ERD và giải thích tại sao `order_items` cần lưu giá tại
    thời điểm mua.
-   **Done:** schema đáp ứng nghiệp vụ và có constraint.

### Ngày 2 --- Query, grouping, pagination

-   **Học:** `JOIN`, `GROUP BY`, aggregate, subquery/CTE,
    `LIMIT/OFFSET`.
-   **Làm:** API danh sách đơn, lọc status, khoảng ngày, phân trang.
-   **Test:** dữ liệu rỗng, trang cuối, filter kết hợp; kiểm tra thứ tự
    ổn định khi pagination.
-   **Docs:** lưu 5 query quan trọng và ý nghĩa của chúng.
-   **Done:** query đúng trên dữ liệu test đại diện.

### Ngày 3 --- ACID và transaction

-   **Học:** atomicity, consistency, isolation, durability; transaction
    boundary trong Spring.
-   **Làm:** tạo đơn và order items trong một transaction; mô phỏng lỗi
    giữa quá trình để xem rollback.
-   **Test:** cố tình ném exception sau khi tạo order nhưng trước khi
    ghi item; xác minh không còn dữ liệu nửa vời.
-   **Docs:** sequence diagram của create order và transaction boundary.
-   **Done:** giải thích được dữ liệu nào commit/rollback.

### Ngày 4 --- Isolation và lost update

-   **Học:** READ COMMITTED, REPEATABLE READ, phantom/lost update ở mức
    phù hợp; đọc tài liệu MySQL version đang dùng.
-   **Làm:** mở hai transaction cập nhật cùng một sản phẩm; tạo kịch bản
    tranh chấp.
-   **Test:** quan sát kết quả với transaction mặc định; thử locking phù
    hợp.
-   **Docs:** `docs/database/concurrency.md` ghi kịch bản, kết quả quan
    sát và cách xử lý.
-   **Done:** phân biệt lý thuyết isolation với hành vi thực tế của
    phiên bản database.

### Ngày 5 --- Index và EXPLAIN

-   **Học:** B-tree, selectivity, composite index, leftmost prefix,
    write overhead.
-   **Làm:** tạo dữ liệu giả đủ lớn; chạy truy vấn tìm đơn theo
    customer/status/date; dùng `EXPLAIN`.
-   **Test:** so sánh kế hoạch trước/sau index; ghi thời gian và
    estimated rows.
-   **Docs:** bảng benchmark có câu SQL, index, dữ liệu, môi trường và
    kết quả.
-   **Done:** không kết luận chỉ từ "có index"; giải thích query planner
    chọn index nào.

### Ngày 6 --- Transaction review và database documentation

-   **Làm:** rà soát constraints, index và transaction; thêm test cho
    duplicate/order invalid.
-   **Test:** chạy test tích hợp; thử tạo order với product không tồn
    tại và số lượng không hợp lệ.
-   **Docs:** hoàn thiện ERD, schema, migration instructions, 3
    ADR/decision notes nếu có thay đổi quan trọng.
-   **Done tuần:** schema và nghiệp vụ đặt hàng chạy được, có benchmark
    đầu tiên.

------------------------------------------------------------------------

## Tuần 4 --- MySQL tuning, partition, locking và connection pool

**Mục tiêu tuần:** biết chọn index, hiểu partition không phải "nút tăng
tốc thần kỳ", và xử lý cập nhật tồn kho đồng thời.

### Ngày 1 --- Đọc query plan

-   **Học:** `EXPLAIN`, `EXPLAIN ANALYZE` nếu version hỗ trợ, estimated
    vs actual rows, filesort/temp table.
-   **Làm:** tạo 3 truy vấn chậm có thể tái hiện; thử thay index và điều
    kiện WHERE.
-   **Test:** ghi kết quả trước/sau; giữ lại một phương án không cải
    thiện để học.
-   **Docs:** cập nhật benchmark, không chỉ ghi kết luận mà ghi bằng
    chứng.
-   **Done:** đọc được key/access type/rows/Extra ở mức cơ bản.

### Ngày 2 --- Composite và covering index

-   **Học:** thứ tự cột, equality/range, order by, covering index, chi
    phí ghi.
-   **Làm:** thử `(customer_id, status, created_at)` so với các biến
    thể; dùng cùng dataset và query.
-   **Test:** đo SELECT và thử INSERT/UPDATE để quan sát trade-off.
-   **Docs:** ghi lý do chọn index cuối cùng và query nào hưởng lợi.
-   **Done:** giải thích được vì sao index rộng hơn không phải lúc nào
    cũng tốt hơn.

### Ngày 3 --- Partition

-   **Học:** RANGE/LIST/HASH, partition pruning, partition key và các
    giới hạn unique key trong MySQL.
-   **Làm:** tạo bảng `order_events` phân vùng theo thời gian trên một
    bảng lab riêng; nạp dữ liệu nhiều tháng.
-   **Test:** query có/không có điều kiện thời gian; kiểm tra kế hoạch
    và partition pruning theo version.
-   **Docs:** `docs/database/partition-lab.md` ghi mục tiêu, DDL, kết
    quả và hạn chế.
-   **Done:** biết khi nào partition có ích cho quản lý dữ liệu
    lớn/lifecycle và khi nào không cần.

### Ngày 4 --- Locking và inventory correctness

-   **Học:** `SELECT ... FOR UPDATE`, optimistic locking/version column,
    deadlock.
-   **Làm:** chọn một chiến lược để xử lý giảm stock; bổ sung constraint
    và transaction.
-   **Test:** hai request cùng mua item cuối cùng; chỉ một request được
    thành công.
-   **Docs:** sequence diagram và giải thích vì sao kiểm tra stock ở
    Java trước khi transaction là chưa đủ.
-   **Done:** không thể bán vượt tồn kho trong test concurrency.

### Ngày 5 --- Connection pool và performance

-   **Học:** connection pool, pool exhaustion, timeout, query duration.
-   **Làm:** kiểm tra cấu hình pool của Spring; tạo tải đọc/ghi vừa
    phải; quan sát pool metrics/log nếu đã cấu hình.
-   **Test:** so sánh concurrency thấp/cao; không tăng pool size vô tội
    vạ.
-   **Docs:** ghi cấu hình ban đầu, dấu hiệu bottleneck và giới hạn của
    lab.
-   **Done:** hiểu connection DB là tài nguyên hữu hạn.

### Ngày 6 --- Báo cáo tuning

-   **Làm:** chạy lại các query quan trọng và inventory concurrency
    test.
-   **Docs:** viết `docs/weekly-reviews/week-04.md` cùng bảng "vấn đề →
    giả thuyết → phép thử → kết quả → quyết định".
-   **Done tuần:** có báo cáo index/partition/locking có số liệu thực.

------------------------------------------------------------------------

## Tuần 5 --- Threads, multithreading và Java concurrency

**Mục tiêu tuần:** nhận diện race condition, quản lý thread pool và phân
biệt CPU-bound với I/O-bound.

### Ngày 1 --- Thread và shared state

-   **Học:** process/thread, stack/heap ở mức cơ bản, thread lifecycle,
    context switching.
-   **Làm:** viết chương trình nhiều thread cùng tăng biến đếm; chạy
    nhiều lần.
-   **Test:** so kết quả mong muốn với kết quả thực tế; không dựa vào
    một lần chạy.
-   **Docs:** ghi cách tái hiện race condition.
-   **Done:** giải thích `count++` không atomic.

### Ngày 2 --- Synchronization

-   **Học:** `synchronized`, `Lock`, `AtomicInteger`, critical section.
-   **Làm:** tạo ba phiên bản counter; so tính đúng đắn và thời gian.
-   **Test:** nhiều thread, nhiều vòng lặp; ghi kết quả nhiều lần.
-   **Docs:** bảng so sánh đơn giản, phạm vi phù hợp và giới hạn của
    từng cách.
-   **Done:** chọn primitive dựa vào yêu cầu chứ không theo cảm tính.

### Ngày 3 --- ExecutorService và thread pool

-   **Học:** task queue, fixed pool, bounded queue, rejection policy,
    shutdown.
-   **Làm:** viết worker xử lý tác vụ giả lập; giới hạn số thread và
    queue.
-   **Test:** tăng số tác vụ vượt khả năng xử lý; quan sát
    queue/rejection và shutdown.
-   **Docs:** ghi các tham số pool và hành vi khi quá tải.
-   **Done:** không tạo thread vô hạn cho mỗi request.

### Ngày 4 --- Future và CompletableFuture

-   **Học:** async composition, exception handling, timeout,
    cancellation.
-   **Làm:** mô phỏng lấy giá, phí vận chuyển và thông tin khách hàng
    song song; phân biệt tác vụ bắt buộc và tùy chọn.
-   **Test:** một task chậm, một task fail, một task thành công; đảm bảo
    response không bị sai.
-   **Docs:** sequence diagram luồng xử lý và cách propagate error.
-   **Done:** hiểu async không tự động có nghĩa là nhanh hơn.

### Ngày 5 --- Deadlock, starvation và Java Memory Model

-   **Học:** lock ordering, deadlock, starvation, visibility,
    `volatile`; xem xét virtual threads nếu dùng Java 21+.
-   **Làm:** tạo ví dụ deadlock có kiểm soát; sửa bằng thứ tự lock nhất
    quán.
-   **Test:** dùng thread dump hoặc log để nhận diện thread đang chờ.
-   **Docs:** ghi triệu chứng, nguyên nhân và phương án khắc phục.
-   **Done:** biết thu thập bằng chứng thay vì đoán.

### Ngày 6 --- Load/concurrency mini-lab

-   **Làm:** tạo một endpoint lab có tác vụ giả lập CPU-bound hoặc
    I/O-bound; so cách xử lý đồng bộ và executor.
-   **Test:** đo throughput, latency và error rate ở concurrency khác
    nhau.
-   **Docs:** lưu kết quả; ghi rõ đây là benchmark local, không đại diện
    production.
-   **Done tuần:** có code, test và ghi chép concurrency.

------------------------------------------------------------------------

## Tuần 6 --- Redis, caching và System Design fundamentals

**Mục tiêu tuần:** triển khai cache-aside, hiểu invalidation và thiết kế
một API có mục tiêu hiệu năng rõ ràng.

### Ngày 1 --- System Design cơ bản

-   **Học:** latency, throughput, availability, scalability,
    reliability, bottleneck.
-   **Làm:** vẽ context diagram và container diagram cho OMS; liệt kê
    yêu cầu chức năng/phi chức năng.
-   **Test:** review sơ đồ bằng cách lần theo request tạo đơn và đọc sản
    phẩm.
-   **Docs:** `docs/architecture/context.md` và
    `docs/architecture/containers.md`.
-   **Done:** mọi thành phần trong sơ đồ có lý do tồn tại.

### Ngày 2 --- Redis cơ bản

-   **Học:** key/value, TTL, String/Hash, eviction policy, persistence ở
    mức overview.
-   **Làm:** chạy Redis bằng Compose; thao tác GET/SET/TTL; tạo key
    convention cho product cache.
-   **Test:** restart container và quan sát theo cấu hình persistence
    thực tế.
-   **Docs:** `docs/redis/key-design.md` ghi naming, TTL, dữ liệu lưu và
    dữ liệu không lưu.
-   **Done:** không lưu thông tin nhạy cảm không cần thiết vào cache.

### Ngày 3 --- Cache-aside

-   **Học:** cache hit/miss, cache-aside, stale data, cache stampede.
-   **Làm:** API đọc product kiểm tra Redis trước, miss thì đọc MySQL và
    ghi cache có TTL.
-   **Test:** gọi lặp lại; xác nhận miss/hit bằng log hoặc metrics.
-   **Docs:** sequence diagram cache hit và cache miss.
-   **Done:** có bằng chứng hành vi cache.

### Ngày 4 --- Cache invalidation

-   **Học:** write-through/write-around ở mức khái niệm, invalidation,
    TTL, consistency trade-off.
-   **Làm:** cập nhật giá sản phẩm; thiết kế xóa hoặc cập nhật cache.
-   **Test:** đọc ngay sau update; xác minh không trả giá cũ ngoài chính
    sách đã chọn.
-   **Docs:** ADR "Chiến lược cache sản phẩm" nêu trade-off.
-   **Done:** có chính sách nhất quán rõ ràng, không chỉ "thêm Redis là
    xong".

### Ngày 5 --- Idempotency và distributed lock

-   **Học:** idempotency key, retry, atomic operation, giới hạn của
    distributed lock.
-   **Làm:** API tạo đơn nhận idempotency key; lưu kết quả theo thiết kế
    phù hợp; từ chối hoặc trả lại kết quả cũ khi request lặp.
-   **Test:** gửi cùng key nhiều lần; không tạo nhiều đơn.
-   **Docs:** ghi key scope, TTL, trạng thái xử lý và các tình huống
    lỗi.
-   **Done:** duplicate request test pass. Không coi Redis lock là thay
    thế transaction/constraint của DB.

### Ngày 6 --- Benchmark cache và system design review

-   **Làm:** dùng script/k6/JMeter gửi cùng tập request với cache
    bật/tắt.
-   **Test:** ghi p50/p95 latency, throughput, error rate; đảm bảo so
    sánh cùng môi trường.
-   **Docs:** `docs/test-reports/cache-benchmark.md`; cập nhật sơ đồ hệ
    thống và câu hỏi còn mở.
-   **Done tuần:** cache có test, policy và số liệu.

------------------------------------------------------------------------

## Tuần 7 --- Kafka, event-driven architecture và reliability

**Mục tiêu tuần:** publish/consume event, hiểu retry, duplicate delivery
và khoảng trống giữa database commit với publish event.

### Ngày 1 --- Kafka fundamentals

-   **Học:** broker, topic, partition, offset, producer, consumer group,
    replication ở mức khái niệm.
-   **Làm:** chạy Kafka KRaft bằng cấu hình lab phù hợp; tạo topic
    `order-created`.
-   **Test:** gửi và đọc message bằng CLI hoặc code sample.
-   **Docs:** sơ đồ producer → topic → consumer và glossary.
-   **Done:** giải thích topic, partition, offset, consumer group.

### Ngày 2 --- Spring Kafka producer

-   **Học:** serialization, key, event schema, timeout và lỗi publish.
-   **Làm:** sau khi tạo đơn, phát `OrderCreatedEvent` với event ID,
    order ID, timestamp và schema version.
-   **Test:** kiểm tra message đến topic; xác minh event không chứa dữ
    liệu nhạy cảm không cần thiết.
-   **Docs:** `docs/events/order-created.md` mô tả schema và ý nghĩa
    trường.
-   **Done:** event contract được version hóa ở mức tối thiểu.

### Ngày 3 --- Consumer và consumer group

-   **Học:** offset commit, partition ordering, rebalance, consumer
    concurrency.
-   **Làm:** consumer xử lý order-created và ghi lịch sử xử lý hoặc
    trạng thái phù hợp.
-   **Test:** chạy nhiều consumer; quan sát phân phối partition và thứ
    tự trong cùng key.
-   **Docs:** ghi assumptions về ordering và consumer group.
-   **Done:** hiểu ordering chỉ được đảm bảo trong phạm vi partition
    theo điều kiện phù hợp, không phải toàn topic.

### Ngày 4 --- Idempotent consumer và retry

-   **Học:** at-least-once, duplicate message, retry/backoff,
    dead-letter topic.
-   **Làm:** lưu event ID đã xử lý bằng cơ chế bền vững; tạo lỗi tạm
    thời và lỗi không thể retry mãi.
-   **Test:** xử lý cùng event hai lần; kiểm tra side effect chỉ xảy ra
    một lần theo thiết kế.
-   **Docs:** runbook retry/DLT và cách replay an toàn.
-   **Done:** duplicate message không tạo tác động nghiệp vụ trùng.

### Ngày 5 --- Transactional Outbox

-   **Học:** dual-write problem: DB commit thành công nhưng publish
    Kafka thất bại, hoặc ngược lại.
-   **Làm:** ghi order và outbox event trong cùng DB transaction; worker
    đọc outbox và publish event.
-   **Test:** mô phỏng crash trước/sau publish; bảo đảm event có thể
    publish lại mà consumer idempotent.
-   **Docs:** ADR "Vì sao cần outbox?" cùng sequence diagram.
-   **Done:** giải thích được outbox giải quyết vấn đề gì và không giải
    quyết vấn đề gì.

### Ngày 6 --- Kafka failure lab

-   **Làm:** dừng consumer, gửi thêm event, khởi động lại; quan sát lag
    và recovery.
-   **Test:** kiểm tra retry, duplicate, poison message và logging.
-   **Docs:** `docs/incidents/kafka-consumer-down.md` gồm phát hiện, ảnh
    hưởng, phục hồi, phòng ngừa.
-   **Done tuần:** luồng event chạy được và có cách vận hành khi lỗi.

------------------------------------------------------------------------

## Tuần 8 --- WebSocket, MinIO và distributed-system patterns

**Mục tiêu tuần:** đẩy trạng thái real-time đến client và quản lý file
bên ngoài database.

### Ngày 1 --- Socket, HTTP và WebSocket

-   **Học:** TCP socket khác HTTP thế nào; WebSocket full-duplex; SSE là
    lựa chọn một chiều.
-   **Làm:** tạo WebSocket endpoint gửi thông báo cập nhật trạng thái
    đơn.
-   **Test:** kết nối bằng client thử nghiệm; gửi event demo và nhận
    thông báo.
-   **Docs:** mô tả handshake, connection lifecycle và authentication
    assumptions.
-   **Done:** giải thích được vì sao WebSocket khác REST polling.

### Ngày 2 --- Kafka đến WebSocket

-   **Học:** event consumer và connection ownership khi có nhiều
    instance.
-   **Làm:** khi order status được cập nhật, gửi thông báo đến client
    đang subscribe.
-   **Test:** reconnect client; API phải cho lấy trạng thái hiện tại
    thay vì phụ thuộc hoàn toàn vào message đã bỏ lỡ.
-   **Docs:** sequence diagram cập nhật trạng thái và reconnect.
-   **Done:** client có cách đồng bộ lại state sau mất kết nối.

### Ngày 3 --- MinIO/object storage

-   **Học:** object, bucket, key, metadata, presigned URL và access
    policy.
-   **Làm:** chạy MinIO local; tạo bucket và upload/download file qua
    backend.
-   **Test:** file tồn tại, file không tồn tại, định dạng/dung lượng
    sai, truy cập không được phép.
-   **Docs:** `docs/storage/minio.md` ghi cấu hình local, quyền truy cập
    và naming convention.
-   **Done:** không public bucket tùy tiện; không commit access key
    thật.

### Ngày 4 --- Metadata và lỗi giữa các hệ thống

-   **Học:** vì sao MySQL và object storage không có chung transaction
    ACID mặc định.
-   **Làm:** lưu object key, owner, content type, size và trạng thái
    upload trong DB.
-   **Test:** mô phỏng upload thành công nhưng ghi DB lỗi; thiết kế
    cleanup/reconciliation phù hợp.
-   **Docs:** runbook cho object orphan và metadata thiếu.
-   **Done:** có chiến lược xử lý partial failure.

### Ngày 5 --- Timeout, retry, circuit breaker, rate limit

-   **Học:** timeout, exponential backoff, jitter, circuit breaker,
    bulkhead và rate limiting.
-   **Làm:** thêm timeout có giới hạn cho một dependency giả lập; tránh
    retry không giới hạn.
-   **Test:** dependency chậm, lỗi liên tục và phục hồi; quan sát
    latency và lỗi.
-   **Docs:** ADR về timeout/retry; ghi trường hợp không nên retry.
-   **Done:** không có retry vô hạn hoặc chồng retry ở nhiều layer mà
    không tính toán.

### Ngày 6 --- End-to-end integration

-   **Làm:** tạo order → publish event → consumer cập nhật → WebSocket
    notify; upload file qua MinIO.
-   **Test:** happy path và ít nhất ba failure case.
-   **Docs:** sơ đồ sequence và checklist test tích hợp.
-   **Done tuần:** các thành phần tích hợp được, giới hạn còn lại được
    ghi rõ.

------------------------------------------------------------------------

## Tuần 9 --- Docker, Docker Compose, Linux networking và reverse proxy

**Mục tiêu tuần:** mọi dependency có thể khởi động có kiểm soát bằng
Compose, hiểu container network và reverse proxy.

### Ngày 1 --- Docker fundamentals

-   **Học:** image, container, layer, registry, volume, network, port
    mapping.
-   **Làm:** build image cho Spring Boot; chạy app container.
-   **Test:** dừng/xóa container và quan sát dữ liệu ở đâu còn/mất.
-   **Docs:** `docs/infra/docker-basics.md` với các lệnh thực tế.
-   **Done:** phân biệt image với container và volume.

### Ngày 2 --- Dockerfile tốt hơn

-   **Học:** multi-stage build, layer caching, non-root user, image size
    và secret handling.
-   **Làm:** tạo Dockerfile multi-stage phù hợp với dự án; tránh copy
    `.env` vào image.
-   **Test:** build image; kiểm tra app khởi động và user chạy process.
-   **Docs:** ghi cách build, tag, run image.
-   **Done:** image có thể tái tạo và không chứa secret.

### Ngày 3 --- Docker Compose

-   **Học:** service, depends_on, healthcheck, environment, volume và
    network.
-   **Làm:** khai báo API, MySQL, Redis, Kafka, MinIO theo từng bước;
    không cần bật mọi dependency ngay nếu máy yếu.
-   **Test:** khởi động/dừng; xác nhận API retry/wait hợp lý khi
    dependency chưa sẵn sàng.
-   **Docs:** README có lệnh start/stop/reset dữ liệu.
-   **Done:** môi trường local có thể dựng lại từ đầu.

### Ngày 4 --- Networking và Web Proxy

-   **Học:** DNS trong Docker network, localhost vs service name, TCP
    port, HTTP, TLS, forward vs reverse proxy.
-   **Làm:** chạy hai API instance và reverse proxy; kiểm tra header,
    route và status.
-   **Test:** request từ host và từ container; xác minh port mapping.
-   **Docs:** sơ đồ network với các port và luồng request.
-   **Done:** chẩn đoán được lỗi "connection refused", sai host và sai
    port.

### Ngày 5 --- HAProxy cơ bản

-   **Học:** backend/server pool, round robin, least connections, health
    check, L4/L7.
-   **Làm:** cấu hình HAProxy route tới hai API instance.
-   **Test:** gọi qua proxy, dừng một API, quan sát health check và phân
    phối traffic.
-   **Docs:** `infra/haproxy/` có config lab và
    `docs/runbooks/haproxy.md`.
-   **Done:** backend chết không tiếp tục nhận traffic sau khi health
    check phát hiện theo cấu hình.

### Ngày 6 --- Debug và reproducibility

-   **Làm:** xóa container/volume lab theo kế hoạch rồi dựng lại; không
    xóa dữ liệu cần giữ.
-   **Test:** kiểm tra logs, health checks, restart policy và dữ liệu
    persistent.
-   **Docs:** runbook "Local environment không khởi động"; ghi 5 lỗi và
    cách xác minh.
-   **Done tuần:** hệ thống local khởi động có hướng dẫn và proxy hoạt
    động.

------------------------------------------------------------------------

## Tuần 10 --- CI/CD hoàn chỉnh, registry, deployment và load testing

**Mục tiêu tuần:** từ commit đến môi trường test có quy trình lặp lại,
kiểm soát lỗi và bảo mật secrets.

### Ngày 1 --- Git workflow và pipeline stages

-   **Học:** branch, pull request, status check, CI vs continuous
    delivery/deployment.
-   **Làm:** chuẩn hóa branch/PR; pipeline gồm compile, unit test,
    integration test nếu runner hỗ trợ, package.
-   **Test:** tạo lỗi test và xác nhận pipeline chặn bước sau.
-   **Docs:** `docs/ci-cd/pipeline.md` mô tả stage và điều kiện thành
    công.
-   **Done:** pipeline là bắt buộc trước khi merge theo khả năng repo.

### Ngày 2 --- Docker build trong CI

-   **Học:** build artifact, image tag, immutable tag, registry
    credentials.
-   **Làm:** pipeline build Docker image khi branch phù hợp; tag theo
    commit SHA.
-   **Test:** xác minh image build fail nếu app không compile; tránh
    dùng `latest` làm định danh duy nhất.
-   **Docs:** ghi chiến lược tag, retention và nơi lưu image.
-   **Done:** image có thể truy về commit nguồn.

### Ngày 3 --- Secrets và môi trường

-   **Học:** secret management, least privilege, environment separation.
-   **Làm:** chuyển credentials khỏi source; dùng GitHub Actions Secrets
    hoặc cơ chế phù hợp; tạo `.env.example` chỉ có placeholder.
-   **Test:** rà soát Git history/current files; xác nhận secret không
    xuất hiện trong log.
-   **Docs:** runbook xoay vòng credential nếu lộ secret.
-   **Done:** không có credential thật trong repo hoặc output pipeline.

### Ngày 4 --- CD lên môi trường test

-   **Học:** deployment environment, approval, smoke test, rollback.
-   **Làm:** tự động deploy lên môi trường test an toàn (VM hoặc local
    runner có chủ đích); nếu chưa có host, tự động dựng Compose
    integration environment và chạy smoke test.
-   **Test:** deployment thành công và cố tình deploy bản lỗi trong lab;
    kiểm tra phát hiện/rollback.
-   **Docs:** `docs/runbooks/deploy-test.md`.
-   **Done:** pipeline có bước xác nhận ứng dụng thực sự healthy, không
    chỉ build thành công.

### Ngày 5 --- Load test

-   **Học:** throughput, concurrency, p50/p95/p99, error rate, warm-up,
    test data.
-   **Làm:** tạo kịch bản đọc sản phẩm và tạo đơn với mức tải nhỏ tăng
    dần bằng k6/JMeter.
-   **Test:** lưu kết quả baseline; không chạy stress test làm ảnh hưởng
    hệ thống không thuộc quyền kiểm soát.
-   **Docs:** `docs/test-reports/baseline.md` ghi cấu hình máy, version,
    dataset và kịch bản.
-   **Done:** báo cáo có thể tái lập và không đưa ra kết luận vượt quá
    bằng chứng.

### Ngày 6 --- CI/CD failure lab

-   **Làm:** mô phỏng test fail, image build fail, deployment health
    check fail.
-   **Test:** pipeline phải dừng đúng chỗ; xem logs và khôi phục.
-   **Docs:** cập nhật pipeline diagram, rollback steps, known
    limitations.
-   **Done tuần:** code → test → image → test deployment có quy trình rõ
    ràng.

------------------------------------------------------------------------

## Tuần 11 --- MySQL replication, Keepalived và High Availability

**Mục tiêu tuần:** hiểu redundancy/failover, replication lag, VIP và vì
sao HA cần kiểm thử failure mode.

### Ngày 1 --- HA, availability, RPO/RTO

-   **Học:** SPOF, redundancy, failover, failback, RPO, RTO,
    split-brain.
-   **Làm:** vẽ failure map cho OMS: API, proxy, DB, Redis, Kafka,
    object storage.
-   **Test:** với mỗi thành phần, ghi tác động khi nó dừng.
-   **Docs:** `docs/architecture/failure-modes.md`.
-   **Done:** phân biệt "có nhiều instance" với "đã có HA".

### Ngày 2 --- MySQL replication lab

-   **Học:** primary/replica, binlog, replication lag, read/write split.
-   **Làm:** nếu tài nguyên cho phép, dựng hai MySQL instance trong lab
    riêng; nếu không, nghiên cứu cấu hình và thực hành trên VM sau.
-   **Test:** ghi trên primary, đọc replica; quan sát độ trễ và lỗi cấu
    hình.
-   **Docs:** `docs/database/replication-lab.md` ghi topology, version,
    kết quả.
-   **Done:** không đưa replica vào đường đọc yêu cầu read-after-write
    mà không có chiến lược nhất quán.

### Ngày 3 --- Failover và consistency

-   **Học:** promotion, failover manual vs automatic, data loss window,
    split-brain prevention.
-   **Làm:** viết runbook giả định primary mất kết nối; xác định ai/điều
    kiện nào được phép promote replica.
-   **Test:** mô phỏng gián đoạn trong lab; không thử trên DB
    production.
-   **Docs:** runbook failover, recovery và xác minh dữ liệu.
-   **Done:** giải thích được vì sao tự động promote mà không có
    fencing/quorum phù hợp có thể nguy hiểm.

### Ngày 4 --- Keepalived/VRRP

-   **Học:** VIP, VRRP, MASTER/BACKUP, health-check script và điều kiện
    chuyển vai trò.
-   **Làm:** trên hai Linux VM nếu có, cài Keepalived và HAProxy; cấu
    hình VIP lab.
-   **Test:** dừng node/proxy theo kịch bản an toàn; quan sát VIP chuyển
    sang node dự phòng.
-   **Docs:** sơ đồ network, config đã loại secret và runbook kiểm tra
    VIP.
-   **Done:** phân biệt failover VIP với bảo toàn connection đang tồn
    tại.

### Ngày 5 --- Fault injection và incident response

-   **Học:** timeout, partial failure, graceful degradation, alerting.
-   **Làm:** thử lần lượt API down, Redis down, consumer down; không
    dừng tất cả cùng lúc.
-   **Test:** đo ảnh hưởng và xác minh phục hồi.
-   **Docs:** tạo 2 incident reports theo mẫu: symptom, impact,
    timeline, root cause, fix, prevention.
-   **Done:** mỗi thử nghiệm có giả thuyết và kết luận dựa trên
    log/metrics.

### Ngày 6 --- HA review

-   **Làm:** rà soát SPOF; phân loại mức độ ưu tiên khắc phục.
-   **Docs:** ADR "Phạm vi HA của lab"; ghi rõ thành phần nào chưa HA và
    tại sao.
-   **Done tuần:** có sơ đồ failure mode và runbook, không tuyên bố HA
    production chỉ dựa trên demo.

------------------------------------------------------------------------

## Tuần 12 --- Kubernetes, observability và System Design tổng kết

**Mục tiêu tuần:** triển khai API lên Kubernetes local, quan sát
self-healing và trình bày được thiết kế hệ thống với trade-off.

### Ngày 1 --- Kubernetes fundamentals

-   **Học:** cluster, node, control plane, Pod, Deployment, ReplicaSet,
    Service.
-   **Làm:** tạo cluster local bằng kind hoặc Minikube; deploy một
    container mẫu.
-   **Test:** inspect Pod, logs, events và service endpoint.
-   **Docs:** `docs/infra/kubernetes-local.md` ghi cài đặt, version và
    lệnh.
-   **Done:** giải thích Deployment quản lý replica như thế nào.

### Ngày 2 --- Deploy Spring Boot

-   **Học:** manifests, ConfigMap, Secret, readiness/liveness/startup
    probe.
-   **Làm:** tạo Deployment 2--3 replicas và Service; cấu hình qua
    environment variables.
-   **Test:** gọi API, xóa một Pod, quan sát Pod mới; thử readiness fail
    trong lab.
-   **Docs:** ghi YAML và giải thích từng trường quan trọng.
-   **Done:** traffic không được đưa đến Pod chưa ready theo cấu hình.

### Ngày 3 --- Resource, rollout và rollback

-   **Học:** requests/limits, scheduling, rolling update, rollout
    history.
-   **Làm:** cấu hình resource hợp lý cho lab; cập nhật image/version và
    thực hành rollback.
-   **Test:** quan sát rollout status; cố tình dùng image sai để hiểu
    failure.
-   **Docs:** `docs/runbooks/k8s-rollout.md`.
-   **Done:** có thể triển khai version mới và khôi phục version cũ.

### Ngày 4 --- Observability

-   **Học:** logs, metrics, traces; correlation/request ID; RED/USE ở
    mức cơ bản.
-   **Làm:** bổ sung health endpoint, request ID/log có cấu trúc; nếu
    còn thời gian, thêm Prometheus/Grafana hoặc OpenTelemetry.
-   **Test:** truy từ một request đến log liên quan; kiểm tra metrics cơ
    bản.
-   **Docs:** `docs/observability.md` nêu metric cần theo dõi và ngưỡng
    cảnh báo thử nghiệm.
-   **Done:** biết tìm bằng chứng khi API chậm hoặc lỗi.

### Ngày 5 --- System Design interview/document

-   **Học:** requirements, estimation, API, data model, high-level
    design, bottleneck, trade-offs.
-   **Làm:** viết design cho OMS ở quy mô nhỏ và kịch bản tăng tải; nêu
    giả định về traffic thay vì bịa số liệu.
-   **Test:** review design bằng các failure case: DB down, duplicate
    event, cache stale, consumer lag, instance crash.
-   **Docs:** `docs/architecture/system-design-review.md` có
    requirements, diagram, bottlenecks, trade-offs và phương án tiến
    hóa.
-   **Done:** trình bày được thiết kế trong 10--15 phút bằng lời và sơ
    đồ.

### Ngày 6 --- Final review và portfolio

-   **Làm:** chạy lại từ README trên môi trường sạch; sửa hướng dẫn
    thiếu; đóng issue còn lại hoặc ghi rõ limitation.
-   **Test:** chạy test suite, smoke test, benchmark quan trọng; không
    ghi "pass" nếu chưa chạy.
-   **Docs:** hoàn thiện README, architecture diagram, API docs, CI/CD
    diagram, benchmark report, 3--5 ADR và weekly reviews.
-   **Done tuần:** người khác có thể clone repo, chạy theo hướng dẫn và
    hiểu giới hạn của dự án.

------------------------------------------------------------------------

# 4. System Design: khung tư duy dùng cho mọi bài

Khi thiết kế URL Shortener, Rate Limiter, Chat hay OMS, luôn đi theo
trình tự:

1.  **Clarify requirements:** chức năng chính, người dùng, read/write
    pattern, yêu cầu nhất quán.
2.  **Non-functional requirements:** latency, throughput, availability,
    durability, security; nếu chưa có số liệu thì ghi rõ giả định.
3.  **API design:** endpoint, request/response, error semantics,
    pagination, idempotency.
4.  **Data model:** entity, key, index, constraints, access pattern.
5.  **High-level architecture:** client, API, database, cache, queue,
    object storage.
6.  **Bottleneck analysis:** CPU, DB, network, lock contention, queue
    lag, hot key.
7.  **Scale strategy:** vertical/horizontal scaling, partitioning,
    replication, caching, async.
8.  **Failure handling:** timeout, retry, duplicate, partial failure,
    failover, recovery.
9.  **Observability/security:** logs, metrics, traces, authentication,
    authorization, secrets.
10. **Trade-offs:** nêu vì sao chọn phương án A thay vì B, chi phí và
    giới hạn.

Không cần đưa Kafka, Redis hay Kubernetes vào mọi thiết kế. Chỉ thêm khi
có yêu cầu hoặc bottleneck hợp lý.

## Bài tập System Design mở rộng sau 12 tuần

  -----------------------------------------------------------------------
  Bài                     Trọng tâm               Đầu ra cần có
  ----------------------- ----------------------- -----------------------
  URL Shortener           key generation, cache,  API + schema +
                          redirect                diagram + bottleneck

  Rate Limiter            token bucket/sliding    algorithm + concurrency
                          window, Redis           tests

  Notification System     Kafka, retries,         event contract + DLT
                          delivery status         runbook

  Chat System             WebSocket, connection   sequence diagram +
                          management              reconnect strategy

  File Storage            MinIO/S3, metadata,     upload/download flow +
                          access control          cleanup strategy

  E-commerce              transaction, inventory, consistency model +
                          outbox                  failure analysis

  Metrics/Logging         ingestion, retention,   data flow +
  platform                query                   storage/retention
                                                  trade-offs
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 5. CI/CD: checklist kiến thức và bài thực hành

## CI --- Continuous Integration

-   [ ] Trigger theo push và pull request.
-   [ ] Compile/package bằng Maven.
-   [ ] Unit test và integration test phù hợp.
-   [ ] Pipeline fail khi test fail.
-   [ ] Lưu test report/artifact khi phù hợp.
-   [ ] Không để credentials xuất hiện trong log.
-   [ ] Có cache dependency nếu giúp tiết kiệm thời gian và không gây
    lỗi stale cache.

## Image build và registry

-   [ ] Multi-stage Dockerfile.
-   [ ] Tag image theo commit SHA/version.
-   [ ] Không dựa vào `latest` duy nhất.
-   [ ] Có kiểm tra image build.
-   [ ] Giới hạn quyền registry token.
-   [ ] Biết cách dọn image/tag cũ.

## CD --- Delivery/Deployment

-   [ ] Deploy vào môi trường test trước.
-   [ ] Config khác nhau giữa local/test/prod nhưng không hard-code
    secret.
-   [ ] Chạy smoke test sau deploy.
-   [ ] Health check và rollback procedure.
-   [ ] Có approval nếu môi trường yêu cầu.
-   [ ] Database migration có chiến lược tương thích với
    rollout/rollback.
-   [ ] Có release notes hoặc commit/version truy vết được.

## Bài tập bắt buộc

1.  Tạo một test cố tình fail → CI phải fail.
2.  Tạo Docker image từ commit → tag truy về commit.
3.  Deploy lên môi trường test → smoke test chạy.
4.  Tạo deployment không healthy → pipeline dừng hoặc rollback theo
    thiết kế.
5.  Viết runbook cho lỗi build, lỗi registry và lỗi deployment.

------------------------------------------------------------------------

# 6. Documentation: viết gì, viết ở đâu và viết đến mức nào?

Documentation không phải bài văn dài. Nó phải giúp m hoặc một người khác
**chạy lại, kiểm chứng, debug và hiểu lý do thiết kế**.

## 6.1. `README.md`

Nên có: - Mục tiêu và phạm vi dự án. - Kiến trúc tổng quan (diagram). -
Yêu cầu môi trường và version. - Cách cấu hình `.env`. - Cách khởi
động/dừng. - Lệnh chạy test. - API docs. - Link đến architecture, ADR,
benchmark và runbook. - Những giới hạn đã biết.

## 6.2. Learning log --- ghi mỗi ngày

Dùng mẫu:

``` markdown
# YYYY-MM-DD — Chủ đề

## Goal
Hôm nay cần chứng minh điều gì?

## What I learned
- Khái niệm 1:
- Khái niệm 2:

## Implementation
- Đã sửa/tạo file nào?
- Quyết định kỹ thuật nào?

## Evidence
- Lệnh đã chạy:
- Test/benchmark:
- Kết quả quan sát thực tế:

## Problems and debugging
- Triệu chứng:
- Giả thuyết:
- Cách kiểm tra:
- Root cause:
- Cách sửa:

## Trade-offs / limitations
- Ưu điểm:
- Nhược điểm:
- Khi nào không nên dùng:

## Questions to revisit
- Câu hỏi 1:
- Câu hỏi 2:

## Next session
Việc nhỏ tiếp theo:
```

## 6.3. ADR --- Architecture Decision Record

Dùng khi có quyết định ảnh hưởng kiến trúc, ví dụ chọn cache-aside hoặc
transactional outbox.

``` markdown
# ADR-0001: <Decision title>

- Status: Proposed | Accepted | Superseded
- Date: YYYY-MM-DD

## Context
Vấn đề gì cần giải quyết? Các constraint là gì?

## Decision
Chọn phương án nào?

## Alternatives
Các phương án khác đã cân nhắc.

## Consequences
Ưu điểm, nhược điểm, chi phí vận hành và rủi ro.

## Validation
Test/benchmark nào hỗ trợ quyết định?
```

## 6.4. Benchmark report

``` markdown
# Benchmark: <scenario>

## Goal
Cần trả lời câu hỏi gì?

## Environment
- CPU/RAM:
- OS/runtime:
- App version/commit:
- Database/cache version:
- Dataset size:

## Workload
- Request mix:
- Concurrency:
- Duration/warm-up:
- Test tool and command:

## Results
- Throughput:
- p50/p95/p99 latency:
- Error rate:
- CPU/memory:
- Other metrics:

## Interpretation
Điều gì được số liệu hỗ trợ? Điều gì chưa thể kết luận?

## Reproduction
Lệnh và dữ liệu cần thiết để chạy lại.
```

## 6.5. Incident report / runbook

Mỗi lần chủ động làm hỏng lab, hãy ghi: - Triệu chứng và ảnh hưởng. -
Timeline. - Logs/metrics đã xem. - Root cause hoặc điều chưa xác định. -
Cách phục hồi. - Cách phòng ngừa. - Bằng chứng xác nhận hệ thống hồi
phục.

------------------------------------------------------------------------

# 7. Tài liệu chính thức nên dùng

Ưu tiên documentation chính thức để tránh học từ cấu hình lỗi thời. Đọc
đúng mục đang học, không cần đọc hết tài liệu.

-   Java: https://dev.java/learn/
-   Java API: https://docs.oracle.com/en/java/javase/
-   Spring Boot: https://docs.spring.io/spring-boot/
-   Spring Guides: https://spring.io/guides
-   Maven: https://maven.apache.org/guides/
-   JUnit 5: https://junit.org/junit5/docs/current/user-guide/
-   MySQL: https://dev.mysql.com/doc/
-   Redis: https://redis.io/docs/latest/
-   Apache Kafka: https://kafka.apache.org/documentation/
-   Docker: https://docs.docker.com/
-   GitHub Actions: https://docs.github.com/actions
-   HAProxy: https://docs.haproxy.org/
-   Keepalived: https://www.keepalived.org/manpage.html
-   Kubernetes: https://kubernetes.io/docs/home/
-   MinIO: https://min.io/docs/
-   k6: https://grafana.com/docs/k6/latest/

Tài liệu có thể thay đổi theo phiên bản. Khi gặp một config hoặc API cụ
thể, luôn kiểm tra tài liệu đúng phiên bản đang chạy.

------------------------------------------------------------------------

# 8. Checklist tổng kết cuối lộ trình

## Java/Spring

-   [ ] Giải thích OOP, DI, transaction và request lifecycle.
-   [ ] Viết REST API có validation, error handling và test.
-   [ ] Tổ chức domain logic không phụ thuộc chặt vào Controller.

## MySQL

-   [ ] Thiết kế schema và constraints.
-   [ ] Đọc `EXPLAIN` và giải thích index.
-   [ ] Thử partition và giải thích giới hạn.
-   [ ] Kiểm thử transaction/locking cho tồn kho.

## Concurrency/Redis/Kafka

-   [ ] Tái hiện và sửa race condition.
-   [ ] Triển khai cache-aside và invalidation.
-   [ ] Xử lý duplicate event, retry và consumer failure.
-   [ ] Giải thích dual-write problem và outbox.

## Networking/HA

-   [ ] Phân biệt forward/reverse proxy và load balancer.
-   [ ] Thử backend health check.
-   [ ] Giải thích VIP failover, replication lag, RPO/RTO.
-   [ ] Không tuyên bố HA khi chưa thử failure mode.

## Docker/CI/CD/Kubernetes

-   [ ] Dựng môi trường local theo hướng dẫn.
-   [ ] CI tự build/test và fail đúng khi test fail.
-   [ ] Build image có tag truy vết được.
-   [ ] Deploy test, smoke test và rollback.
-   [ ] Deploy API lên Kubernetes local và quan sát self-healing.

## System Design/Documentation

-   [ ] Có context/container diagram.
-   [ ] Có API contract và schema docs.
-   [ ] Có ít nhất 3 ADR quan trọng.
-   [ ] Có benchmark trước/sau.
-   [ ] Có ít nhất 2 incident reports/runbooks.
-   [ ] Trình bày kiến trúc và trade-off trong 10--15 phút.

------------------------------------------------------------------------

# 9. Nếu bị chậm tiến độ thì cắt gì trước?

**Không cắt:** Java/Spring, SQL/transaction/index, test, Git, Docker cơ
bản, CI, Redis cache-aside, Kafka fundamentals, System Design
fundamentals và documentation.

**Có thể chuyển sang giai đoạn sau:** Keepalived trên hai VM, MySQL
failover tự động, Kafka cluster nhiều broker, MinIO production
deployment, Kubernetes stateful workload, Prometheus/Grafana nâng cao.

Mục tiêu của 12 tuần không phải "biết hết mọi công nghệ". Mục tiêu là có
một dự án backend chạy được, có test, có pipeline, có benchmark và có
tài liệu đủ tốt để m giải thích được cơ chế, failure mode và trade-off.
Sau đó, m có thể mở rộng từng nhánh theo yêu cầu thực tế của team.
