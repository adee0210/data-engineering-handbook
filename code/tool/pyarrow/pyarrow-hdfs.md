# Hướng dẫn sử dụng PyArrow với HDFS

## 1. Giới thiệu
PyArrow là thư viện Python mạnh mẽ cho xử lý dữ liệu lớn, hỗ trợ tích hợp với HDFS (Hadoop Distributed File System). PyArrow cho phép thao tác file và thư mục trong HDFS, đọc/ghi dữ liệu dưới nhiều format như Parquet, CSV, JSON, v.v. với hiệu suất cao.

## 2. Cài đặt
```bash
pip install pyarrow
```

### 2.5. Cấu hình Environment (Bắt buộc)
Để PyArrow có thể kết nối HDFS, cần cấu hình các biến môi trường sau trong `~/.bashrc` hoặc `~/.zshrc`:

```bash
# --- Java ---
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH

# --- Hadoop ---
export HADOOP_HOME=/home/duc/F_APACHE/hadoop-3.3.6
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
export HADOOP_STREAMING=$HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-3.3.6.jar
export HADOOP_LOG_DIR=$HADOOP_HOME/logs
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

# >>> PyArrow + HDFS (Phần bắt buộc để fix lỗi CLASSPATH) >>>
export CLASSPATH="$($HADOOP_HOME/bin/hadoop classpath --glob)"
export ARROW_LIBHDFS_DIR=$HADOOP_HOME/lib/native
```

Sau khi thêm, chạy `source ~/.bashrc` (hoặc `source ~/.zshrc`) để áp dụng.

**Lưu ý quan trọng:**
- `CLASSPATH` phải được set động bằng `hadoop classpath --glob` để PyArrow load được các JAR files cần thiết.
- `ARROW_LIBHDFS_DIR` trỏ đến thư mục chứa libhdfs.so (native library cho HDFS).
- Nếu không cấu hình đúng, sẽ gặp lỗi "HDFS connection failed" hoặc "CLASSPATH not set".

## 3. Kết nối HDFS
```python
from pyarrow import fs

# Kết nối HDFS
hdfs = fs.HadoopFileSystem(host="localhost", port=9000, user="duc")

# Kiểm tra kết nối
print(hdfs.get_file_info("/"))
```

**Lưu ý:** User phải có quyền truy cập thư mục tương ứng trong HDFS.

### 3.5. User và Permissions trong HDFS
Hiểu đúng về user và permissions rất quan trọng để tránh lỗi "Permission denied".

#### Giải thích output của `hdfs dfs -ls /user`
Khi chạy lệnh `hdfs dfs -ls /user`, bạn có thể thấy output như:
```
Found 1 items
drwxr-xr-x   - duc supergroup          0 2025-12-10 00:16 /user/duc_le
```

**Phân tích từng phần:**
- `drwxr-xr-x`: Permissions (d=rwxr-xr-x, giống Unix)
- `-`: Không có ACLs
- `duc`: **Owner** của thư mục (user system thực hiện operation đầu tiên)
- `supergroup`: Group
- `0`: Size (bytes)
- `2025-12-10 00:16`: Modification time
- `/user/duc_le`: Path

**Quan trọng:** Owner là "duc", không phải "duc_le". Tên thư mục không nhất thiết match với owner!

#### Tại sao dùng user="duc" trong PyArrow?
- PyArrow yêu cầu user parameter phải match với **owner** của thư mục HDFS
- Nếu thư mục `/user/duc_le` có owner là "duc", thì trong code phải dùng `user="duc"`
- Nếu dùng `user="duc_le"`, sẽ gặp lỗi "Permission denied" vì user "duc_le" không sở hữu thư mục

#### Tạo thư mục với đúng owner
Để tạo thư mục với owner mong muốn:
```bash
# Chạy với user cụ thể (cần user tồn tại trong system)
sudo -u <desired_user> hdfs dfs -mkdir -p /user/<desired_user>

# Hoặc trong Python với user đúng
hdfs = fs.HadoopFileSystem(host="localhost", port=9000, user="<actual_owner>")
hdfs.create_dir("/user/<actual_owner>/my_dir")
```

#### Ví dụ thực tế
Giả sử output của `hdfs dfs -ls /user` là:
```
Found 2 items
drwxr-xr-x   - duc supergroup          0 2025-12-10 00:34 /user/duc
drwxr-xr-x   - duc supergroup          0 2025-12-10 00:16 /user/duc_le
```

**Giải thích:**
- Cả hai thư mục `/user/duc` và `/user/duc_le` đều có owner là "duc"
- Bạn có thể dùng `user="duc"` để truy cập cả hai thư mục
- Tên thư mục không ảnh hưởng đến quyền truy cập, chỉ owner HDFS mới quan trọng

```python
# Có thể truy cập cả hai thư mục với cùng user
hdfs = fs.HadoopFileSystem(host="localhost", port=9000, user="duc")

# Truy cập /user/duc
info_duc = hdfs.get_file_info(fs.FileSelector("/user/duc", recursive=False))

# Truy cập /user/duc_le  
info_duc_le = hdfs.get_file_info(fs.FileSelector("/user/duc_le", recursive=False))
```

#### Quyền truy cập của Owner
**Quan trọng:** Owner của thư mục có quyền **full access** ngay lập tức!

**Ví dụ thực tế về quyền:**
Giả sử ban đầu chỉ có `/user/duc_le` với owner "duc":
```
drwxr-xr-x   - duc supergroup          0 2025-12-10 00:16 /user/duc_le
```

- User "duc" là owner → có quyền **rwx** (read, write, execute)  
- Permissions `drwxr-xr-x` = owner=rwx, group=rx, other=rx
- Vậy user "duc" có thể truy cập `/user/duc_le` ngay từ đầu, **không cần tạo** `/user/duc`

**Kết luận:** Tên thư mục chỉ là label. Quyền truy cập phụ thuộc vào **owner HDFS**. Nếu bạn là owner, bạn có full quyền trên thư mục đó!

## 4. CRUD Operations cho Thư mục

### Create (Tạo thư mục)
```python
# Tạo thư mục
hdfs.create_dir("/user/duc/test_dir")

# Tạo thư mục đệ quy
hdfs.create_dir("/user/duc/parent/child", recursive=True)
```

### Read (Đọc/Liệt kê)
```python
from pyarrow import fs

# Liệt kê thư mục
selector = fs.FileSelector("/user/duc", recursive=False)
info = hdfs.get_file_info(selector)

for file_info in info:
    print(f"Path: {file_info.path}, Type: {file_info.type.name}, Size: {file_info.size}")
```

### Update (Di chuyển/Đổi tên)
```python
# Di chuyển thư mục
hdfs.move("/user/duc/old_dir", "/user/duc/new_dir")

# Sao chép thư mục
hdfs.copy_file("/user/duc/source_dir", "/user/duc/dest_dir")
```

### Delete (Xóa)
```python
# Xóa thư mục rỗng
hdfs.delete_dir("/user/duc/empty_dir")

# Xóa thư mục có nội dung (có thể cần xóa file trước)
# Xóa tất cả file trong thư mục trước
selector = fs.FileSelector("/user/duc/dir_to_delete", recursive=True)
files = hdfs.get_file_info(selector)
for f in files:
    if f.type.name == "File":
        hdfs.delete_file(f.path)

hdfs.delete_dir("/user/duc/dir_to_delete")
```

## 5. Đọc/Ghi File

### Đọc/Ghi Text File
```python
# Ghi text
with hdfs.open_output_stream("/user/duc/test.txt") as f:
    f.write(b"Hello HDFS from PyArrow!")

# Đọc text
with hdfs.open_input_stream("/user/duc/test.txt") as f:
    content = f.read().decode('utf-8')
    print(content)
```

### Đọc/Ghi Binary Data
```python
# Ghi binary
data = b"Binary data example"
with hdfs.open_output_stream("/user/duc/binary.dat") as f:
    f.write(data)

# Đọc binary
with hdfs.open_input_stream("/user/duc/binary.dat") as f:
    binary_content = f.read()
    print(binary_content)
```

## 6. Xử lý Dữ liệu Lớn với Parquet

### Chuyển đổi từ JSON/Dict/List sang Parquet

#### Từ Dict/List Python
```python
import pyarrow as pa
import pyarrow.parquet as pq

# Từ list of dicts
data_list = [
    {"id": 1, "name": "Alice", "age": 25},
    {"id": 2, "name": "Bob", "age": 30},
    {"id": 3, "name": "Charlie", "age": 35}
]

# Tạo PyArrow Table từ list of dicts
table = pa.Table.from_pylist(data_list)

# Ghi vào HDFS dưới dạng Parquet
with hdfs.open_output_stream("/user/duc/data_from_list.parquet") as f:
    pq.write_table(table, f, compression='snappy')
```

#### Từ Dict với Arrays
```python
# Dict với arrays (column-oriented)
data_dict = {
    "id": [1, 2, 3, 4, 5],
    "name": ["Alice", "Bob", "Charlie", "David", "Eve"],
    "age": [25, 30, 35, 40, 45],
    "scores": [[85, 90], [78, 82], [92, 88], [75, 80], [88, 95]]
}

# Tạo Table từ dict
table = pa.table(data_dict)

# Ghi Parquet
with hdfs.open_output_stream("/user/duc/data_from_dict.parquet") as f:
    pq.write_table(table, f)
```

#### Từ JSON String
```python
import json

# JSON string
json_str = '''
[
    {"id": 1, "name": "Alice", "data": {"score": 85, "grade": "A"}},
    {"id": 2, "name": "Bob", "data": {"score": 78, "grade": "B"}}
]
'''

# Parse JSON
data = json.loads(json_str)

# Chuyển sang Table (handle nested dict)
table = pa.Table.from_pylist(data)

# Ghi Parquet
with hdfs.open_output_stream("/user/duc/data_from_json.parquet") as f:
    pq.write_table(table, f)
```

#### Từ JSON File
```python
# Đọc JSON file từ HDFS
with hdfs.open_input_stream("/user/duc/input.json") as f:
    json_data = json.load(f)

# Nếu là list of dicts
if isinstance(json_data, list):
    table = pa.Table.from_pylist(json_data)
else:
    # Nếu là single dict, wrap in list
    table = pa.Table.from_pylist([json_data])

# Ghi Parquet
with hdfs.open_output_stream("/user/duc/converted.parquet") as f:
    pq.write_table(table, f)
```

#### Xử lý Nested JSON/Dict
```python
# Nested data
nested_data = [
    {
        "user": {"id": 1, "name": "Alice"},
        "orders": [
            {"item": "book", "price": 20},
            {"item": "pen", "price": 5}
        ]
    },
    {
        "user": {"id": 2, "name": "Bob"},
        "orders": [
            {"item": "notebook", "price": 15}
        ]
    }
]

# PyArrow tự động handle nested structures
table = pa.Table.from_pylist(nested_data)

# Ghi Parquet (giữ nested structure)
with hdfs.open_output_stream("/user/duc/nested_data.parquet") as f:
    pq.write_table(table, f, use_dictionary=True)
```

### Ghi DataFrame vào Parquet
```python
import pandas as pd
import pyarrow as pa
import pyarrow.parquet as pq

# Tạo DataFrame mẫu
df = pd.DataFrame({
    'id': range(1000000),
    'value': [f'value_{i}' for i in range(1000000)],
    'timestamp': pd.date_range('2023-01-01', periods=1000000, freq='1min')
})

# Chuyển sang PyArrow Table
table = pa.Table.from_pandas(df)

# Ghi vào HDFS dưới dạng Parquet
with hdfs.open_output_stream("/user/duc/large_data.parquet") as f:
    pq.write_table(table, f, compression='snappy')
```

### Đọc Parquet từ HDFS
```python
# Đọc toàn bộ file
with hdfs.open_input_file("/user/duc/large_data.parquet") as f:
    table = pq.read_table(f)
    df = table.to_pandas()

print(f"Loaded {len(df)} rows")
```

### Streaming Read cho Dữ liệu Lớn
```python
# Đọc theo chunk để xử lý dữ liệu lớn
with hdfs.open_input_file("/user/duc/large_data.parquet") as f:
    parquet_file = pq.ParquetFile(f)
    
    for batch in parquet_file.iter_batches(batch_size=10000):
        # Xử lý từng batch
        df_batch = batch.to_pandas()
        print(f"Processing batch with {len(df_batch)} rows")
        # Thực hiện xử lý: aggregation, filter, etc.
```

## 7. Các Format Data Khác

## 7. Các Format Data Khác

### CSV
```python
import pyarrow.csv as pacsv

# Từ dict/list sang CSV
data_list = [
    {"id": 1, "name": "Alice", "age": 25},
    {"id": 2, "name": "Bob", "age": 30}
]
table = pa.Table.from_pylist(data_list)

# Ghi CSV
with hdfs.open_output_stream("/user/duc/data.csv") as f:
    pacsv.write_csv(table, f)

# Đọc CSV
with hdfs.open_input_file("/user/duc/data.csv") as f:
    table = pacsv.read_csv(f)
```

### JSON
```python
import pyarrow.json as pajson

# Từ dict/list sang JSON Lines
data_list = [
    {"id": 1, "name": "Alice", "data": {"score": 85}},
    {"id": 2, "name": "Bob", "data": {"score": 78}}
]
table = pa.Table.from_pylist(data_list)

# Ghi JSON Lines
with hdfs.open_output_stream("/user/duc/data.jsonl") as f:
    pajson.write_json(table, f)

# Đọc JSON
with hdfs.open_input_file("/user/duc/data.jsonl") as f:
    table = pajson.read_json(f)
```

### ORC
```python
import pyarrow.orc as paorc

# Từ dict/list sang ORC
data_dict = {
    "id": [1, 2, 3],
    "name": ["Alice", "Bob", "Charlie"],
    "age": [25, 30, 35]
}
table = pa.table(data_dict)

# Ghi ORC
with hdfs.open_output_stream("/user/duc/data.orc") as f:
    paorc.write_table(table, f)

# Đọc ORC
with hdfs.open_input_file("/user/duc/data.orc") as f:
    table = paorc.read_table(f)
```

## 8. Xử lý Dữ liệu Lớn (Big Data)

### Chunked Reading
```python
# Đọc file lớn theo chunk
chunk_size = 64 * 1024 * 1024  # 64MB

with hdfs.open_input_stream("/user/duc/huge_file.dat") as f:
    while True:
        chunk = f.read(chunk_size)
        if not chunk:
            break
        # Xử lý chunk
        process_chunk(chunk)
```

### Parallel Processing
```python
from concurrent.futures import ThreadPoolExecutor

def process_file(file_path):
    with hdfs.open_input_file(file_path) as f:
        # Xử lý file
        pass

# Danh sách file cần xử lý
file_paths = ["/user/duc/file1.parquet", "/user/duc/file2.parquet"]

with ThreadPoolExecutor(max_workers=4) as executor:
    executor.map(process_file, file_paths)
```

### Memory Management
```python
# Sử dụng PyArrow's compute functions để xử lý in-memory
import pyarrow.compute as pc

# Filter dữ liệu mà không load toàn bộ vào memory
with hdfs.open_input_file("/user/duc/large.parquet") as f:
    table = pq.read_table(f, filters=[('value', '>', 100)])
    
    # Aggregation
    result = pc.sum(table['numeric_column'])
    print(f"Sum: {result}")
```

## 9. Best Practices

- **User Permissions**: Luôn sử dụng user có quyền truy cập thư mục tương ứng
- **Path Management**: Sử dụng absolute path trong HDFS
- **Error Handling**: Wrap operations trong try-except
- **Compression**: Sử dụng compression (snappy, gzip) cho dữ liệu lớn
- **Partitioning**: Phân chia dữ liệu theo ngày/tháng cho truy vấn hiệu quả
- **Caching**: Sử dụng PyArrow's caching cho repeated reads

## 10. Troubleshooting

### Permission Denied
- Kiểm tra user trong HadoopFileSystem match với owner thư mục
- Sử dụng thư mục `/user/<username>/`

### Connection Failed
- Đảm bảo HDFS đang chạy: `jps` để check NameNode, DataNode
- Kiểm tra host và port

### Memory Issues
- Sử dụng chunked reading cho file lớn
- Tăng memory limit nếu cần: `export ARROW_DEFAULT_MEMORY_POOL_BYTES=2147483648` (2GB)

## 11. Tài liệu tham khảo
- [PyArrow Documentation](https://arrow.apache.org/docs/python/)
- [Hadoop HDFS Documentation](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html)
- [Parquet Format](https://parquet.apache.org/)