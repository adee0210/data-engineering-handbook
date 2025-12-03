# Hướng dẫn thao tác PostgreSQL trong Docker

## 1. Truy cập vào container PostgreSQL

```sh
sudo docker exec -it postgres bash
```

## 2. Đăng nhập vào PostgreSQL bằng user đã cấu hình

```sh
psql -U le_duc -d symbol_db
```

Nếu user khác, thay bằng tên user phù hợp:
```sh
psql -U <user> -d <database>
```

## 3. Tạo user mới (nếu cần)

Trong giao diện psql:
```sql
CREATE USER <username> WITH PASSWORD '<password>';
ALTER USER <username> WITH CREATEDB;
```
Ví dụ:
```sql
CREATE USER le_duc WITH PASSWORD 'ducle1610';
ALTER USER le_duc WITH CREATEDB;
```

## 4. Tạo database mới

Trong giao diện psql:
```sql
CREATE DATABASE <database_name>;
```
Ví dụ:
```sql
CREATE DATABASE dl_ckvn;
```

## 5. Tạo bảng mới

Trong giao diện psql hoặc từ ứng dụng:
```sql
CREATE TABLE <table_name> (
    id SERIAL PRIMARY KEY,
    data JSONB
);
```
Ví dụ:
```sql
CREATE TABLE dl_ckvn (
    id SERIAL PRIMARY KEY,
    data JSONB
);
```

## 6. Thoát khỏi psql

```sql
\q
```

## 7. Thoát khỏi container

```sh
exit
```

---
**Lưu ý:**
- Các lệnh SQL phải chạy trong giao diện psql, không phải trong bash.
- Thay đổi tên user, database, bảng cho phù hợp với dự án của bạn.
