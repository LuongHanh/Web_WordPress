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

Trước tiên tạo 1 thư mục project `mkdir wordpress_web` chui vào nó `cd web_wordpress` tạo file docker-compose.yml;
Do là cổng 8080 là được dùng cho dịch cụ phpMyAdmin của bài tập 2, nên bài này phpMyAdmin sẽ dùng cổng 8888, các dịch vụ khác không cần mở cổng, truy cập qua tên service. Các service cũng phải dặt tên khác với các service đã tồn tại.

Thêm nữa; em sử dụng luôn tunnel dùng trong bài tập 2 và add route mới;
<img width="959" height="464" alt="image" src="https://github.com/user-attachments/assets/9909b12c-336d-4e96-b1c1-0b519213fd73" />

Nội dung docker-compose.yml:

```
services:
  wp_mariadb:
    image: mariadb:latest
    container_name: wp_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "123"
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: hanh
      MYSQL_PASSWORD: "0402"
    volumes:
      - db_data:/var/lib/mysql

  wp_phpmyadmin:
    image: phpmyadmin:latest
    container_name: wp_phpmyadmin
    restart: always
    ports:
      - "8888:80"
    environment:
      PMA_HOST: wp_mariadb
    depends_on:
      - wp_mariadb

  wp_wordpress:
    image: wordpress:latest
    container_name: wp_wordpress
    restart: always
    # Không mở ports vì truy cập qua Cloudflare Tunnel
    environment:
      WORDPRESS_DB_HOST: wp_mariadb
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: "123"
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - wp_mariadb

  wp_cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: wp_cloudflared
    restart: always
    command: tunnel --no-autoupdate run --token eyJhIjoiNTI0YmRjMzY2YjgzZTg3N2RkMWIxM2MyMGJiNmY1YmIiLCJ0IjoiYWU5NzU2MjktZmEyNi00ZWRmLTkxMjAtNmNhODlkOGVkOWUwIiwicyI6Ill6ZzBZekU1T1RVdFpXUXlOaTAwTnpjM0xUZ3pZVFV0WWpJNVlqWXpORGsxWmpnNSJ9
    depends_on:
      - wp_wordpress

volumes:
  db_data:
  wp_data:
```

Sau khi `sudo docker compose up -d` ta truy cập https://lvhblog.firstmydm.io.vn/ ta được:

Chọn ngôn ngữ:
<img width="959" height="497" alt="image" src="https://github.com/user-attachments/assets/720f6304-6ecc-41b2-93b1-c1f69b55f03e" />

Tạo tài khoản, đặt mật khẩu đơn giản cho nhanh:
<img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/2800d05d-5ebe-4075-8c69-4d6264b876da" />

<img width="959" height="467" alt="image" src="https://github.com/user-attachments/assets/efc71815-67e1-4ea1-86e3-0d503a578496" />

Đăng nhập tài khoản vừa tạo:
<img width="959" height="440" alt="image" src="https://github.com/user-attachments/assets/441df731-cfc9-4771-86a0-1ad9e9dd9e01" />

Trang giao diện khi vừa vào:
<img width="959" height="506" alt="image" src="https://github.com/user-attachments/assets/d371322b-4406-484e-a1c7-bc9168abe42a" />

Tạo 1 vài bài posts:
<img width="959" height="500" alt="image" src="https://github.com/user-attachments/assets/b33bb0e8-cd3e-4537-8e0c-aaaa2bb1c15d" />

View posts ta được:
<img width="959" height="481" alt="image" src="https://github.com/user-attachments/assets/61a36c50-5dd7-49a3-9890-d2221cd0ee03" />

Tạo thêm 1 bài posts khác:
<img width="959" height="485" alt="image" src="https://github.com/user-attachments/assets/d4bcab69-7f38-4ce8-902f-93d13d77c8d8" />

<img width="950" height="495" alt="image" src="https://github.com/user-attachments/assets/16540eab-6905-4d7e-85a4-0a49863ce722" />

<img width="959" height="510" alt="image" src="https://github.com/user-attachments/assets/ac7266b2-5859-4b94-b413-1b55d80ea815" />

Nhận xét về việc sử dụng WordPress:
- Về công sức: Việc triển khai qua Docker giúp tiết kiệm rất nhiều thời gian so với cài đặt thủ công. WordPress cung cấp sẵn khung sườn, chỉ mất khoảng 30 phút để có một website hoàn chỉnh với đầy đủ tính năng CMS.
- Độ khó: Rất dễ sử dụng đối với người dùng cuối nhờ trình soạn thảo Gutenberg (kéo thả block). Tuy nhiên, để cấu hình tối ưu trên Docker thì cần kiến thức vững về Network và Container.
- Tài nguyên máy chủ (RAM/CPU): * RAM: Qua kiểm tra bằng lệnh docker stats, hệ thống tiêu tốn tổng cộng khoảng 450MB - 520MB RAM (MariaDB chiếm nhiều nhất khoảng 200MB, WordPress khoảng 150MB, còn lại là PhpMyAdmin và Cloudflared).
+ CPU: Rất nhẹ, dao động từ 0.1% - 1% khi ở trạng thái nghỉ.
+ SSH/Storage: Toàn bộ Image Docker chiếm khoảng 1.2GB ổ cứng.
