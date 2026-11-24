# Bài 2: Tạo user `deploy` và cài đặt Docker

Danh sách đầy đủ các lệnh đã thực thi: [commands.txt](./commands.txt)

## Các bước

1. Đăng nhập VPS bằng quyền root (`sudo -i`).
2. Tạo user `deploy` và cấp quyền sudo:
   ```bash
   adduser deploy
   usermod -aG sudo deploy
   ```
3. Cài Docker Engine và Docker Compose plugin từ repo chính thức của Docker (xem `commands.txt`).
4. Thêm `deploy` vào group `docker` để chạy Docker không cần `sudo`:
   ```bash
   usermod -aG docker deploy
   ```
5. Chuyển sang user `deploy` và kiểm tra:
   ```bash
   su - deploy
   docker --version
   docker compose version
   ```

## Kết quả

`docker --version` chạy dưới quyền user `deploy`:

![docker version](images/docker-version.png)
