# Bài tập: Tường lửa với UFW

## Mục tiêu

Cấu hình tường lửa UFW trên VPS để chỉ mở những cổng thật sự cần thiết, chặn mọi kết nối vào khác.

**Kịch bản:** Một ứng dụng web chạy trên VPS cloud, lắng nghe ở cổng `8080`. Chỉ cần mở cổng SSH (`22`) để quản trị và cổng `8080` để người dùng truy cập ứng dụng.

## Các lệnh đã thực hiện

1. Mặc định chặn mọi kết nối đi vào:

```bash
sudo ufw default deny incoming
```

2. Mặc định cho phép mọi kết nối đi ra:

```bash
sudo ufw default allow outgoing
```

3. Mở cổng SSH **trước tiên** để không tự khóa mình khỏi server khi bật tường lửa:

```bash
sudo ufw allow 22/tcp
```

4. Mở cổng `8080` cho ứng dụng web:

```bash
sudo ufw allow 8080/tcp
```

5. Bật tường lửa (`--force` để bỏ qua câu hỏi xác nhận):

```bash
sudo ufw --force enable
```

6. Xem trạng thái, chính sách mặc định và danh sách rule:

```bash
sudo ufw status verbose
```

7. Liệt kê các cổng TCP đang lắng nghe và tiến trình sở hữu:

```bash
sudo ss -tlnp
```

## Giải thích

- **Vì sao `deny incoming` là mặc định an toàn nhất?** Theo nguyên tắc "đặc quyền tối thiểu": mọi kết nối vào đều bị chặn, chỉ những cổng được mở rõ ràng mới truy cập được. Nếu vô tình cài thêm dịch vụ lắng nghe trên cổng khác, dịch vụ đó cũng không bị lộ ra Internet.
- **Vì sao giữ `allow outgoing`?** Server vẫn cần chủ động kết nối ra ngoài để cập nhật gói (`apt update`), phân giải DNS, gọi API bên ngoài… Chặn chiều ra sẽ làm hỏng các chức năng này mà lợi ích bảo mật không nhiều.
- **Vì sao phải mở cổng 22 trước khi bật tường lửa?** Ta đang quản trị VPS qua SSH. Nếu bật UFW với `deny incoming` mà chưa có rule cho cổng 22, kết nối SSH mới sẽ bị chặn và ta bị khóa ngoài server (chỉ còn cách vào qua console của nhà cung cấp cloud).
- **Cổng "đang lắng nghe" khác cổng "truy cập được từ bên ngoài":**
  - `ss -tlnp` cho biết tiến trình nào đang **lắng nghe** trên cổng nào bên trong máy (ví dụ nginx trên `0.0.0.0:8080`).
  - Rule tường lửa quyết định gói tin từ bên ngoài có **được đi tới** cổng đó hay không.
  - Một cổng chỉ truy cập được từ bên ngoài khi **cả hai** điều kiện đều đúng: có dịch vụ lắng nghe **và** tường lửa cho phép.

## Kết quả

Nội dung bằng chứng (output của `ufw status verbose`, `ss -tlnp` và lệnh `curl` kiểm tra từ bên ngoài):

```text
# ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
8080/tcp                   ALLOW IN    Anywhere                  
8090/tcp                   ALLOW IN    Anywhere                  
2906/tcp                   ALLOW IN    Anywhere                  
2907/tcp                   ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
8080/tcp (v6)              ALLOW IN    Anywhere (v6)             
8090/tcp (v6)              ALLOW IN    Anywhere (v6)             
2906/tcp (v6)              ALLOW IN    Anywhere (v6)             
2907/tcp (v6)              ALLOW IN    Anywhere (v6)             

-----
# ss -tlnp
State  Recv-Q Send-Q Local Address:Port Peer Address:PortProcess                                                                               
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=69490,fd=6),("nginx",pid=69489,fd=6),("nginx",pid=18702,fd=6))   
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=665,fd=3))                                                        
LISTEN 0      511          0.0.0.0:2906      0.0.0.0:*    users:(("nginx",pid=69490,fd=13),("nginx",pid=69489,fd=13),("nginx",pid=18702,fd=13))
LISTEN 0      511          0.0.0.0:2907      0.0.0.0:*    users:(("nginx",pid=69490,fd=19),("nginx",pid=69489,fd=19),("nginx",pid=18702,fd=19))
LISTEN 0      511          0.0.0.0:8090      0.0.0.0:*    users:(("nginx",pid=69490,fd=15),("nginx",pid=69489,fd=15),("nginx",pid=18702,fd=15))
LISTEN 0      511          0.0.0.0:8080      0.0.0.0:*    users:(("nginx",pid=69490,fd=10),("nginx",pid=69489,fd=10),("nginx",pid=18702,fd=10))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=69490,fd=12),("nginx",pid=69489,fd=12),("nginx",pid=18702,fd=12))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=665,fd=4))                                                        
LISTEN 0      511             [::]:2906         [::]:*    users:(("nginx",pid=69490,fd=14),("nginx",pid=69489,fd=14),("nginx",pid=18702,fd=14))
LISTEN 0      511             [::]:2907         [::]:*    users:(("nginx",pid=69490,fd=20),("nginx",pid=69489,fd=20),("nginx",pid=18702,fd=20))
LISTEN 0      511             [::]:8090         [::]:*    users:(("nginx",pid=69490,fd=16),("nginx",pid=69489,fd=16),("nginx",pid=18702,fd=16))
LISTEN 0      511             [::]:8080         [::]:*    users:(("nginx",pid=69490,fd=11),("nginx",pid=69489,fd=11),("nginx",pid=18702,fd=11))
-----
# curl http://160.250.246.219:8080 (from outside)
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Tue, 06 Oct 2026 21:34:46 GMT
Content-Type: text/html
Content-Length: 6567
Last-Modified: Sun, 04 Oct 2026 21:37:09 GMT
Connection: keep-alive
ETag: "6ac2c705-19a7"
Accept-Ranges: bytes

```

**Xác nhận:**

- `Status: active`: tường lửa đang bật, chính sách mặc định là `deny (incoming), allow (outgoing)`.
- `22/tcp` và `8080/tcp` đều `ALLOW IN` từ `Anywhere` (cả IPv4 và IPv6).
- `ss -tlnp` cho thấy `sshd` lắng nghe cổng `22` và `nginx` lắng nghe cổng `8080`.
- Lệnh `curl` từ bên ngoài tới cổng `8080` trả về `HTTP/1.1 200 OK`, tức là ứng dụng truy cập được qua tường lửa.

> **Lưu ý:** Trên VPS này còn có các rule mở cổng `80`, `8090`, `2906`, `2907` (nginx đang lắng nghe các cổng này). Các rule đó không thuộc phạm vi bài tập. Nếu không cần, có thể xóa bằng `sudo ufw delete allow <port>/tcp` để chỉ giữ lại cổng `22` và `8080`.
