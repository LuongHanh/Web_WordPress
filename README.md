# Lớp: 58KTPM - Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421

**Bài tập 03:**  

# SỬ DỤNG WORDPRESS ĐỂ TẠO WEB SITE
## deadline : 23h59 ngày 12 tháng 5 năm 2026.

### ĐỀ BÀI

1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO docker compose chứa: 
- Mariadb: sử dụng **image: mariadb:latest** để làm hệ quản trị csdl cho wordpress
- Phpmyadmin: sư dụng **image: phpmyadmin:latest** để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết)
- WordPress: Sử dụng **image: wordpress:latest**, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin

2. Yêu cầu: sau khi có 3 service này trong file docker-compose.yml :
- Cấu hình để hệ thống chạy
- Sử dụng cloudflare tunnel để public web này lên 1 sub-domain
- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...
- Tạo 1 bài viết trong wordpress giới thiệu về ngành học mà em yêu thích trong trường TNUT. bài viết phải chứa hình ảnh, video, ...
- Nhận xét việc sử dụng mã nguồn mở wordpress để tạo website (tốn công sức thế nào, dễ/khó dùng ra sao, tốn kém tài nguyên(ssh/ram) của máy chủ ra sao,....)

### BÀI LÀM

Ok. Để thực hiện yêu cầu trên, ta cần:

Trước tiên tạo 1 thư mục project `mkdir web_wordpress` chui vào nó `cd web_wordpress` tạo file docker-compose.yml có nội dung như sau:
<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/29e87d67-a86c-4cba-9dce-b8a7cfffffc3" />

Do là cổng 8080 là được dùng cho dịch cụ phpMyAdmin của bài tập 2, nên bài này phpMyAdmin sẽ dùng cổng 8088. Tương tự cổng 8000 cũng bị dịch vụ trong bài tập 2 chiếm rồi, wordpress sẽ dùng cổng 8008. Các service cũng phải dặt tên khác với các service đã tồn tại.

```
# Khai báo phiên bản định dạng của Docker Compose (3.8 là bản ổn định và phổ biến)
version: '3.8'

# Định nghĩa các dịch vụ (containers) sẽ chạy trong hệ thống
services:

  # --- DỊCH VỤ CƠ SỞ DỮ LIỆU ---
  bt3mariadb:
    # Sử dụng bản mới nhất của MariaDB làm hệ quản trị CSDL
    image: mariadb:latest
    # Đặt tên cho container để dễ quản lý (thay vì để Docker tự đặt tên ngẫu nhiên)
    container_name: bt3_mariadb
    # Tự động khởi động lại container nếu máy chủ reboot hoặc service bị lỗi
    restart: always
    # Thiết lập các thông số môi trường (biến cấu hình bên trong container)
    environment:
      MYSQL_ROOT_PASSWORD: root_password # Mật khẩu cao nhất của hệ thống CSDL
      MYSQL_DATABASE: wordpress_db      # Tự động tạo một database tên là wordpress_db
      MYSQL_USER: hanh               # Tạo một user riêng cho WordPress
      MYSQL_PASSWORD: 0402       # Mật khẩu cho user trên
    # Lưu trữ dữ liệu lâu dài (Persistent Data)
    volumes:
      # Gắn vùng nhớ db_data vào thư mục chứa dữ liệu của MariaDB để khi xóa container ko mất data
      - db_data:/var/lib/mysql

  # --- DỊCH VỤ QUẢN LÝ CSDL QUA GIAO DIỆN WEB ---
  bt3phpmyadmin:
    # Sử dụng image phpMyAdmin để quản lý MariaDB trực quan hơn
    image: phpmyadmin:latest
    container_name: bt3_phpmyadmin
    restart: always
    # Mở cổng truy cập
    ports:
      - "8088:80" # Truy cập phpMyAdmin qua địa chỉ http://localhost:8080
    environment:
      PMA_HOST: bt3mariadb              # Chỉ định phpMyAdmin kết nối tới service có tên là 'bt3mariadb'
      MYSQL_ROOT_PASSWORD: 0402 # Dùng pass root để có toàn quyền quản lý
    # Chỉ chạy phpmyadmin SAU KHI dịch vụ 'bt3mariadb' đã sẵn sàng
    depends_on:
      - bt3mariadb

  # --- DỊCH VỤ WORDPRESS ---
  wordpress:
    # Sử dụng image mã nguồn mở WordPress mới nhất
    image: wordpress:latest
    container_name: wordpress
    restart: always
    # Mở cổng truy cập cho Website
    ports:
      - "8008:80" # Truy cập web qua địa chỉ http://localhost:8008
    environment:
      WORDPRESS_DB_HOST: bt3mariadb             # Kết nối tới container tên 'bt3mariadb'
      WORDPRESS_DB_USER: hanh        # User đã tạo ở trên
      WORDPRESS_DB_PASSWORD: 0402 # Pass đã tạo ở trên
      WORDPRESS_DB_NAME: wordpress_db   # Database đã tạo ở trên
    # Lưu trữ code và các file upload (ảnh, video) của người dùng
    volumes:
      - wp_data:/var/www/html
    # Đảm bảo database chạy trước thì WordPress mới kết nối được để cài đặt
    depends_on:
      - bt3mariadb

  bt3cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: bt3_cloudflared
    restart: always

    command: tunnel --no-autoupdate run --token eyJhIjoiNTI0YmRjMzY2YjgzZTg3N2RkMWIxM2MyMGJiNmY1YmIiLCJ0IjoiYWM2YjhiNzYtYjhiYS00ODliLWI3OTYtYWY0MGFjYzk5ZTRkIiwicyI6IlptVmhNV001T0RRdE9UVmhNUzAwTjJRM0xUZzFOREl0WVRjek5UVmlZVGN3TlRsaiJ9

    depends_on:
      - wordpress


# Khai báo các vùng lưu trữ dữ liệu (volumes) độc lập với vòng đời của container
volumes:
  db_data: # Nơi lưu trữ dữ liệu database thực sự
  wp_data: # Nơi lưu trữ mã nguồn và ảnh/video của web
```


