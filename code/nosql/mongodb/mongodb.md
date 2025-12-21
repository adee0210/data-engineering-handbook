# MongoDB: Tổng quan chi tiết

## 1. MongoDB là gì?
MongoDB là hệ quản trị cơ sở dữ liệu NoSQL dạng document, lưu trữ dữ liệu dưới dạng BSON (Binary JSON). Thay vì bảng (table) như RDBMS, MongoDB dùng collection (tập hợp) các document (tài liệu). MongoDB được phát triển để đáp ứng nhu cầu lưu trữ dữ liệu phi cấu trúc, linh hoạt, dễ mở rộng và hiệu suất cao.

- **NoSQL**: Không dùng bảng, không ràng buộc schema cứng nhắc, phù hợp dữ liệu thay đổi linh hoạt.
- **Document-oriented**: Dữ liệu lưu dưới dạng document (JSON/BSON), mỗi document có thể có cấu trúc khác nhau.
- **Open Source**: Miễn phí, cộng đồng lớn, có bản thương mại hỗ trợ doanh nghiệp.

## 2. Kiến trúc hạ tầng MongoDB
### 2.1. Thành phần cơ bản
- **Document**: Đơn vị lưu trữ cơ bản, dạng JSON/BSON. Ví dụ:
  ```json
  {
    "_id": ObjectId("..."),
    "name": "User01",
    "age": 25,
    "skills": ["python", "mongodb"],
    "address": {"city": "Hanoi", "district": "Ba Dinh"}
  }
  ```
- **Collection**: Tập hợp các document, tương tự table nhưng không có schema cố định.
- **Database**: Tập hợp các collections.
- **Instance**: Một tiến trình mongod chạy trên server, quản lý 1 hoặc nhiều database.

### 2.2. Kiến trúc mở rộng
- **Replica Set**: Nhóm các mongod instance để đảm bảo tính sẵn sàng (HA), tự động failover. Một replica set có 1 primary (ghi/đọc chính) và nhiều secondary (chỉ đọc, backup).
- **Sharding**: Chia nhỏ dữ liệu trên nhiều server để mở rộng ngang (horizontal scaling). Dữ liệu được phân mảnh (shard) dựa trên shard key.
- **Mongos**: Router cho các sharded cluster, nhận query từ client và phân phối đến các shard phù hợp.

### 2.3. Sơ đồ tổng quan
```
Client <-> Mongos <-> Shard1, Shard2, ... <-> Replica Set (Primary, Secondary)
```

## 3. Khái niệm chính
- **Schema-less**: Mỗi document có thể có cấu trúc khác nhau, không cần định nghĩa schema trước.
- **Index**: Tăng tốc truy vấn, hỗ trợ nhiều loại index (single, compound, text, geo, hashed, wildcard, ...).
- **Aggregation Pipeline**: Xử lý dữ liệu phức tạp qua nhiều bước (match, group, sort, project, lookup, unwind, ...).
- **Change Streams**: Theo dõi thay đổi realtime trên collection/database/cluster.
- **Transactions**: Hỗ trợ multi-document transaction (ACID) từ phiên bản 4.0, nhưng hiệu năng thấp hơn so với RDBMS.
- **Capped Collection**: Collection giới hạn dung lượng, tự động xóa bản ghi cũ khi đầy (phù hợp log, cache).
- **TTL Index**: Tự động xóa document sau thời gian nhất định (phù hợp session, cache).
- **GridFS**: Lưu trữ file lớn (ảnh, video, ...), chia nhỏ file thành các chunk.

## 4. Ưu điểm của MongoDB
- **Linh hoạt**: Không cần schema cố định, dễ thay đổi cấu trúc dữ liệu, phù hợp dữ liệu phi cấu trúc hoặc thay đổi liên tục.
- **Dễ mở rộng**: Hỗ trợ sharding, scale-out dễ dàng, phù hợp hệ thống lớn, phân tán.
- **Hiệu suất cao**: Đọc/ghi nhanh với workload lớn, tối ưu cho insert/update nhiều, phù hợp dữ liệu phi cấu trúc.
- **Tính sẵn sàng cao**: Replica set, tự động failover, đảm bảo hệ thống luôn hoạt động.
- **Dễ tích hợp**: Hỗ trợ nhiều ngôn ngữ (Python, Java, Node.js, Go, ...), driver phong phú, REST API, Atlas cloud.
- **Tính năng mạnh**: Aggregation pipeline, indexing, full-text search, geo-spatial, change streams, TTL, GridFS.
- **Quản trị dễ dàng**: Công cụ quản lý trực quan (Compass, Atlas UI), backup/restore tiện lợi.

## 5. Nhược điểm của MongoDB
- **Không ACID mạnh như RDBMS**: Dù có transaction nhưng không tối ưu cho nghiệp vụ cần consistency tuyệt đối, transaction multi-document hiệu năng thấp.
- **Tốn dung lượng**: BSON lưu trữ nhiều metadata hơn so với bảng truyền thống, index nhiều cũng tốn RAM/disk.
- **Join hạn chế**: Không join phức tạp như SQL, phải dùng aggregation hoặc lookup, hiệu năng join lớn không cao.
- **Tối ưu query phức tạp khó hơn**: Không có query planner mạnh như RDBMS, phải thiết kế index hợp lý.
- **Yêu cầu quản trị tốt**: Sharding, replica set cần hiểu rõ để vận hành ổn định, dễ gặp lỗi nếu cấu hình sai.
- **Không phù hợp dữ liệu có cấu trúc chặt chẽ**: Nếu dữ liệu ít thay đổi, schema cố định, RDBMS sẽ tối ưu hơn.

## 6. Khi nào nên dùng MongoDB?
- Dữ liệu phi cấu trúc, thay đổi linh hoạt (log, IoT, user profile, CMS, social, ...).
- Ứng dụng cần scale-out lớn, nhiều node, dữ liệu phân tán toàn cầu.
- Ứng dụng realtime, cần change streams, geo-spatial, full-text search, TTL, cache.
- Khi tốc độ phát triển, thay đổi schema nhanh là ưu tiên (startup, MVP, microservice).
- Lưu trữ file lớn (GridFS), session, cache, log, analytics.

## 7. Khi không nên dùng MongoDB?
- Ứng dụng tài chính, ngân hàng, cần consistency mạnh, nhiều transaction phức tạp, join nhiều bảng.
- Hệ thống cần join phức tạp, phân tích dữ liệu dạng bảng lớn, báo cáo BI.
- Dữ liệu có cấu trúc chặt chẽ, ít thay đổi, cần enforce schema, nhiều ràng buộc khóa ngoại.
- Khi cần tối ưu dung lượng lưu trữ, chi phí phần cứng thấp.
- Khi đội ngũ chưa có kinh nghiệm quản trị NoSQL, sharding, replica set.

## 8. Tổng kết
MongoDB phù hợp cho ứng dụng hiện đại, dữ liệu phi cấu trúc, cần mở rộng linh hoạt, realtime, tích hợp nhanh. Tuy nhiên, cần cân nhắc kỹ về consistency, transaction, join, và quản trị khi triển khai ở quy mô lớn hoặc nghiệp vụ tài chính.

---
**Tài liệu tham khảo:**
- https://www.mongodb.com/docs/
- https://www.mongodb.com/compare/mongodb-vs-mysql
- https://viblo.asia/p/mongodb-co-ban-phan-1-tong-quan-va-cai-dat-ORNZqQyQ5o6
- https://viblo.asia/p/mongodb-co-ban-phan-2-truy-van-va-indexing-ORNZqQyQ5o6
