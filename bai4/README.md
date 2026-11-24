# Bài 4: Cấu hình Firewall UFW

## 1. Bật UFW (mở SSH trước khi enable)

```bash
sudo ufw allow 22/tcp            # mở SSH TRƯỚC để không bị khóa ngoài
sudo ufw default deny incoming   # chặn mọi kết nối đến
sudo ufw default allow outgoing
sudo ufw allow 8080/tcp          # mở cổng ứng dụng
sudo ufw enable
sudo ufw status verbose
```

## 2. Chạy thử Nginx ở cổng 8080 và 8081

```bash
docker run -d --name nginx-8080 -p 8080:80 nginx
docker run -d --name nginx-8081 -p 8081:80 nginx
```

### Lưu ý: Docker bỏ qua UFW

Docker tự ghi rule iptables khi publish port (`-p`), nên cổng 8081 **vẫn truy cập được**
dù UFW không mở. Để UFW thực sự chặn port của container, thêm rule vào chain `DOCKER-USER`
ở cuối file `/etc/ufw/after.rules`:

```
# BEGIN UFW AND DOCKER
*filter
:ufw-user-forward - [0:0]
:DOCKER-USER - [0:0]
-A DOCKER-USER -j RETURN -s 10.0.0.0/8
-A DOCKER-USER -j RETURN -s 172.16.0.0/12
-A DOCKER-USER -j RETURN -s 192.168.0.0/16
-A DOCKER-USER -j ufw-user-forward
-A DOCKER-USER -j DROP -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -d 172.16.0.0/12
-A DOCKER-USER -j DROP -p udp -m udp --dport 0:32767 -d 172.16.0.0/12
-A DOCKER-USER -j RETURN
COMMIT
# END UFW AND DOCKER
```

```bash
sudo systemctl restart ufw
# Chỉ cho phép đi vào container nginx-8080 (port bên trong container là 80)
IP_8080=$(docker inspect -f '{{.NetworkSettings.IPAddress}}' nginx-8080)
sudo ufw route allow proto tcp from any to $IP_8080 port 80
sudo ufw status verbose
```

Cách đơn giản hơn nếu chỉ cần chặn 8081: bind container vào localhost
`docker run -d --name nginx-8081 -p 127.0.0.1:8081:80 nginx`.

## 3. Kiểm tra từ máy cá nhân

```bash
curl -I http://<IP_PUBLIC>:8080   # HTTP/1.1 200 OK
curl -I --max-time 5 http://<IP_PUBLIC>:8081   # timeout -> bị chặn
```

> Trên AWS/GCP cần mở thêm 8080 (và 8081 để so sánh) trong Security Group/Firewall của Cloud,
> nếu không thì cả hai cổng đều bị chặn từ phía Cloud chứ không phải do UFW.

## Kết quả

`sudo ufw status verbose`:

![ufw status](images/ufw-status.png)

Truy cập cổng 8080 thành công, 8081 thất bại:

![8080 ok](images/port-8080-ok.png)
![8081 fail](images/port-8081-fail.png)
