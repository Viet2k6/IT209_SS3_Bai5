# Bài 5: Thiết lập thư mục Web an toàn tại `/opt/`

Hướng dẫn triển khai và thiết lập phân quyền an toàn cho ứng dụng web chạy Nginx đặt tại thư mục `/opt/my-app/`.

---

## 1. Cấu trúc thư mục

```text
/opt/my-app/
├── html/
│   └── index.html
└── logs/
    ├── access.log
    └── error.log
```

---

## 2. Nginx Configuration

File cấu hình: `my-app.conf` (thường đặt tại `/etc/nginx/conf.d/my-app.conf` hoặc `/etc/nginx/sites-available/my-app.conf`):

```nginx
server {
    listen 80;
    server_name localhost;

    root /opt/my-app/html;
    index index.html;

    access_log /opt/my-app/logs/access.log;
    error_log /opt/my-app/logs/error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## 3. Ownership và Permissions

Bảng phân quyền và quyền sở hữu chi tiết:

| Đường dẫn | Owner:Group | Quyền (Octal) | Ý nghĩa bảo mật |
| :--- | :--- | :---: | :--- |
| `/opt/my-app/` | `devops:www-data` | `750` | User đọc/ghi/thực thi, group đọc/thực thi, other không có quyền |
| `/opt/my-app/html/` | `devops:www-data` | `750` | User quản trị toàn quyền, Nginx duyệt thư mục |
| `index.html` | `devops:www-data` | `640` | User đọc/ghi, Nginx chỉ đọc |
| `/opt/my-app/logs/` | `devops:www-data` | `770` | Cho phép cả user và Nginx ghi log |
| `access.log` | `devops:www-data` | `660` | Cho phép cả user và Nginx đọc/ghi file log |
| `error.log` | `devops:www-data` | `660` | Cho phép cả user và Nginx đọc/ghi file log |

---

## 4. Kiểm tra quyền ghi

Chạy lệnh dưới quyền user `devops` để kiểm tra khả năng chỉnh sửa mã nguồn:

```bash
sudo -u devops bash -c 'echo "Update Test" >> /opt/my-app/html/index.html'
```

*Kết quả kiểm tra nội dung:*

```text
Update Test
```

---

## 5. Kiểm tra Nginx

Kiểm tra cú pháp cấu hình, khởi động lại dịch vụ và test request:

```bash
# Kiểm tra cú pháp
sudo nginx -t

# Khởi động lại Nginx
sudo service nginx restart

# Gửi HTTP request kiểm tra
curl http://localhost
```

*Kết quả:*

```html
<h1>My App</h1>
Update Test
```

---

## 6. Kiểm tra access log

Kiểm tra 10 dòng log mới nhất để xác nhận Nginx ghi nhận request thành công:

```bash
tail -n 10 /opt/my-app/logs/access.log
```

*Log ghi nhận request:*

```text
127.0.0.1 - - [01/Oct/2026:22:03:01 +0700] "GET / HTTP/1.1" 200 28 "-" "curl/8.18.0"
```