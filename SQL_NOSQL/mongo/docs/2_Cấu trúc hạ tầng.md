# KIẾN TRÚC HẠ TẦNG MONGO

## 1. Mongod (Core Database Process)

- **Khái niệm**: Là thành phần lõi Mongo, là deamon(quá trình chạy nền). Xử lí tất cả các hoạt động của mongod bao gồm read/write , quản lí cache

**Chức năng**:
- Xử lí truy vấn: Query language của MongoDB
- Quản lí lưu trữ: Sử dụng WiredTiger để lưu trữ data trên đĩa. WiredTiger hỗ trợ nén dữ liệu, journaling (ghi log để phục hồi lỗi, khi chương trình bị ngắt đột ngột, log này sẽ lưu data trước đó và phục hồi sau), và checkpointing (lưu trạng thái định kỳ).
- Bộ nhớ đệm: Mongod sử dụng bộ nhớ RAM để cache dữ liệu thường dùng, giảm thời gian truy cập đĩa.