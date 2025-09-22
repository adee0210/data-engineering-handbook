# TỔNG QUAN
## 1. Giới thiệu
- MongoDB là NoSQL Database dạng document-oriented (lưu dữ liệu dưới dạng JSON-like BSON).
- Một bản ghi trong MongoDB là một document, đây là một cấu trúc dữ liệu bao gồm các cặp khóa-giá trị, tương tự như cấu trúc của các đối tượng JSON.

## 2.Các thuật ngữ
- **Database**: Một database trong MongoDB là tập hợp các collections. Đây là nơi dữ liệu được tổ chức và lưu trữ.
- **Document**: Trong MongoDB, các bản ghi được gọi là documents. Mỗi document là một cấu trúc dữ liệu chứa các cặp key-value. Các giá trị có thể là số, chuỗi, boolean, mảng, hoặc thậm chí là các document lồng nhau.
- **Collection**: Một collection là một nhóm các documents liên quan. Không giống như bảng trong SQL, các documents trong một collection không cần có cùng cấu trúc hoặc kiểu dữ liệu.

![MongoDB Logo](https://images.viblo.asia/full/a4c6af07-bd31-43b0-ac4e-7ca05d834abc.png)

- So sánh với SQL databases:

![MongoDB Logo](https://images.viblo.asia/8ed0c0e2-5b39-464b-ba60-68d9ec31d6f8.png)

## 3. Khi nào nên và không dùng
### a) Trường hợp điển hình nên dùng
- Schema thay đổi nhiều, phải ALTER TABLE(quá trình thêm sửa xóa , thay đổi dịnh dạng data cho cột trong bảng)
- Nhiều trường mang giá trị NULL làm bảng phình to, tốn storage, query chậm
- Rất nhiều cột, trong khi cột có cột không hoặc thiết kế trước siêu bảng tới hàng trăm cột
- Các data từ API,... dưới dạng JSON, không cần chuyển đổi lớn
- Khi cần scale ngang hoặc throughput cao(throughput hay số lượng công việc request, transaction, record mà hệ thống có thể xử lí trong 1 khoảng thời gian)


- Các trường hợp dữ liệu sinh ra liên tục theo thời gian, liên tục tăng

- Hỗ trợ full text search tốt giống google search, có thể tạo text index, tìm các bản ghi theo trường text khá lớn


- Lưu sẵn tất cả các data trong 1 doc, cần cho read(1 truy vấn đơn duy nhất để lấy toàn bộ data thay vì join) do cơ chế tải đọc lớn denormalize. 
### b) Không nên dùng
- Không phù hợp với dữ liệu đúng tuyệt đối tại 1 thời điểm khi nhiều người người thao tác cùng lúc. VD: Cả 2 mua máy, khi đọc thấy hàng còn 5 cái. khi cả 2 cùng đặt mua thì người kia hiển thị 5 - 1 = 4 , người này cũng hiển thị 5 - 1 = 4 trong khi phải là 3 => không đúng

- Mặc dù có Trasanction nhưng hiệu năng kém so với SQL

- Hạn chế RAM, do lưu data và index trên disk khi query sẽ load index và data vào ram (cơ chế WiredTiger) => nếu data không nằm trong ram sẽ đọc từ disk sẽ rất chậm. Khi 1 instance nhỏ (máy ít RAM) mà index chiếm nhiều RAM sẽ chậm





