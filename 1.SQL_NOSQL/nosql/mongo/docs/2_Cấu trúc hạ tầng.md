# KIẾN TRÚC HẠ TẦNG MONGO

## 1. Mongod (Core Database Process)
**Khái niệm:** Là thành phần lõi Mongo, là deamon (quá trình chạy nền). Xử lí tất cả các hoạt động của mongod bao gồm read/write, quản lí cache.

**Chức năng:**
- Xử lí truy vấn: Query language của MongoDB.
- Quản lí lưu trữ: Sử dụng WiredTiger để lưu trữ data trên đĩa. WiredTiger hỗ trợ nén dữ liệu, journaling (ghi log để phục hồi lỗi, khi chương trình bị ngắt đột ngột, log này sẽ lưu data trước đó và phục hồi sau), và checkpointing (lưu trạng thái định kỳ).
- Bộ nhớ đệm: Mongod sử dụng bộ nhớ RAM để cache dữ liệu thường dùng, giảm thời gian truy cập đĩa.
- Quản lý kết nối: Mongod xử lý các kết nối từ client, bao gồm authentication và authorization.
- Replication: Trong replica sets, mongod có thể hoạt động như primary (xử lý write) hoặc secondary (replicate data từ primary).
- Indexing: Tạo và quản lý indexes để tối ưu hóa truy vấn.

## 2. Mongos (Query Router)
**Khái niệm:** Là quá trình router trong sharded clusters, hoạt động như một proxy giữa client và các shard servers (mongod instances). Mongos không lưu trữ dữ liệu mà chỉ định tuyến truy vấn đến các shard phù hợp.

**Chức năng:**
- Định tuyến truy vấn: Phân tích truy vấn và gửi đến các shard liên quan dựa trên shard key.
- Aggregation và merging: Thu thập kết quả từ nhiều shard và merge chúng trước khi trả về client.
- Load balancing: Phân bổ tải giữa các shard để đảm bảo hiệu suất.
- Metadata caching: Lưu cache metadata từ config servers để giảm độ trễ.
- Hỗ trợ sharding: Quản lý việc split chunks và balancing dữ liệu giữa các shard.

## 3. Config Servers
**Khái niệm:** Là các mongod instances chuyên lưu trữ metadata cho sharded clusters, bao gồm thông tin về cấu trúc cluster, shard keys, và vị trí dữ liệu.

**Chức năng:**
- Lưu trữ config database: Bao gồm collections như chunks, shards, databases, collections.
- Đồng bộ hóa metadata: Sử dụng replica set để đảm bảo tính nhất quán và high availability (thường là 3 nodes).
- Hỗ trợ balancer: Cung cấp thông tin để mongos thực hiện balancing chunks giữa shards.
- Phục hồi cluster: Giúp khôi phục trạng thái cluster sau sự cố.
- Không xử lý dữ liệu người dùng: Chỉ metadata, nên yêu cầu tài nguyên thấp hơn so với data shards.

## 4. Replica Sets
**Khái niệm:** Là nhóm các mongod instances (thường 3-7 nodes) để cung cấp high availability và data redundancy. Bao gồm một primary và các secondary, với khả năng failover tự động.

**Chức năng:**
- Replication: Secondary replicate data từ primary qua oplog (operation log).
- Failover: Nếu primary thất bại, election process chọn secondary mới làm primary.
- Read scaling: Client có thể đọc từ secondary để giảm tải primary (với tùy chọn read preference).
- Data durability: Sử dụng write concern để đảm bảo data được replicate trước khi xác nhận write.
- Arbiter nodes: Nodes không lưu data, chỉ tham gia election để tránh tie votes.

## 5. Sharded Clusters
**Khái niệm:** Là kiến trúc mở rộng ngang (horizontal scaling) bằng cách phân chia dữ liệu thành các shards (mỗi shard là một replica set). Mongos làm router, config servers lưu metadata.

**Chức năng:**
- Phân phối dữ liệu: Dựa trên shard key để split collections thành chunks và phân bổ chúng giữa shards.
- Horizontal scaling: Thêm shards để tăng capacity mà không downtime.
- Balancing: Balancer process tự động di chuyển chunks để cân bằng tải.
- Zone sharding: Gán zones để kiểm soát vị trí dữ liệu (ví dụ: theo địa lý).
- High throughput: Xử lý hàng triệu operations/giây bằng cách parallelize queries.

## 6. Các Thành Phần Hỗ Trợ Khác
- **MongoDB Drivers:** Các thư viện client-side (Java, Python, Node.js, v.v.) để kết nối và tương tác với mongod/mongos.
- **Storage Engines:** WiredTiger (mặc định), In-Memory (cho low-latency), Encrypted (cho bảo mật).
- **Security Features:** Authentication (SCRAM, X.509), Authorization (RBAC), Encryption (at-rest và in-transit).
- **Monitoring Tools:** MongoDB Ops Manager/Cloud Manager cho monitoring, backup, và automation.
- **Deployment Options:** Standalone, Replica Set, Sharded Cluster; On-premise hoặc Cloud (MongoDB Atlas).

## Lưu Ý Triển Khai Hạ Tầng
- **Hardware Requirements:** CPU đa lõi, RAM lớn (cho caching), SSD cho storage (WiredTiger hiệu suất cao với SSD).
- **Networking:** Sử dụng low-latency networks, firewall để bảo vệ ports (mặc định 27017 cho mongod).
- **Backup & Recovery:** Sử dụng mongodump/mongorestore, snapshots, hoặc continuous backup trong Atlas.
- **Best Practices:** Sử dụng monitoring (Prometheus integration), tuning parameters (cache size, connection pool), và regular maintenance (compact, repair).