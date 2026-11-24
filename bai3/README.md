# Bài 3: Tăng cường bảo mật SSH

## 1. Tạo cặp SSH Key trên máy cá nhân

```bash
ssh-keygen -t ed25519 -C "deploy@staging" -f ~/.ssh/id_ed25519_deploy
# Private key: ~/.ssh/id_ed25519_deploy
# Public key : ~/.ssh/id_ed25519_deploy.pub
```

## 2. Copy Public Key lên VPS cho user `deploy`

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_deploy.pub deploy@<IP_PUBLIC>
```

Hoặc làm thủ công trên VPS:

```bash
sudo mkdir -p /home/deploy/.ssh
sudo nano /home/deploy/.ssh/authorized_keys   # dán nội dung file .pub
sudo chown -R deploy:deploy /home/deploy/.ssh
sudo chmod 700 /home/deploy/.ssh
sudo chmod 600 /home/deploy/.ssh/authorized_keys
```

Thử đăng nhập bằng key **trước khi** tắt mật khẩu:

```bash
ssh -i ~/.ssh/id_ed25519_deploy deploy@<IP_PUBLIC>
```

## 3. Cấu hình `/etc/ssh/sshd_config`

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```

Các dòng cần chỉnh (xem [sshd_config.snippet](./sshd_config.snippet)):

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

> Ubuntu 22.04: kiểm tra thêm thư mục `/etc/ssh/sshd_config.d/` (ví dụ `50-cloud-init.conf`)
> vì file trong đó có thể ghi đè `PasswordAuthentication yes`.

```bash
sudo grep -r "PasswordAuthentication" /etc/ssh/sshd_config.d/
sudo sshd -t                 # kiểm tra cú pháp
sudo systemctl restart ssh
```

## 4. Kiểm tra

```bash
ssh -i ~/.ssh/id_ed25519_deploy deploy@<IP_PUBLIC>   # thành công
ssh root@<IP_PUBLIC>                                  # bị từ chối
```

Ảnh đăng nhập root bị từ chối:

![root bị từ chối](images/root-denied.png)
