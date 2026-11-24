# Bài 1: Khởi tạo và kết nối VPS

## 1. Khởi tạo VPS

- Cloud provider: AWS EC2 (Free Tier)
- Hệ điều hành: Ubuntu Server 22.04 LTS
- Instance type: t2.micro (1 vCPU, 1 GB RAM), ổ đĩa 8–20 GB
- Security Group: mở cổng 22 (SSH)
- Key pair: `staging-key.pem` (tải về máy cá nhân)

Ảnh cấu hình VPS trên giao diện Cloud:

![Cấu hình VPS](images/vps-config.png)

## 2. Kết nối SSH từ máy cá nhân

```bash
# Linux / macOS: phân quyền cho file key
chmod 400 staging-key.pem

# Kết nối bằng IP public của VPS
ssh -i staging-key.pem ubuntu@<IP_PUBLIC>
```

Trên Windows PowerShell:

```powershell
ssh -i .\staging-key.pem ubuntu@<IP_PUBLIC>
```

Kiểm tra sau khi đăng nhập:

```bash
whoami
hostname
lsb_release -a
```

Ảnh đăng nhập SSH thành công:

![SSH thành công](images/ssh-login.png)
