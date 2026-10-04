# Bài 1: Thay đổi cổng kết nối SSH (SSH Port Hardening)

## Mục tiêu
Thay đổi cổng dịch vụ SSH mặc định từ 22 sang 2222 trên máy chủ ảo nhằm hạn chế các cuộc quét cổng tự động và cấu hình tường lửa UFW để tránh nguy cơ bị khóa quyền truy cập.

## Các bước thực hiện bằng tài khoản `devops` có quyền sudo

### 1. Thay đổi cổng kết nối SSH
Sử dụng text editor (như nano) để chỉnh sửa file cấu hình của SSH daemon:
```bash
sudo nano /etc/ssh/sshd_config
```
Tìm dòng có chứa `#Port 22` hoặc `Port 22` và thay đổi thành:
```
Port 2222
```
Lưu thay đổi và thoát khỏi editor.

### 2. Mở cổng 2222 trên tường lửa UFW
Cấu hình tường lửa UFW để cho phép traffic TCP đi qua cổng 2222:
```bash
sudo ufw allow 2222/tcp
```
*(Có thể kiểm tra lại trạng thái của UFW bằng lệnh: `sudo ufw status`)*

### 3. Khởi động lại dịch vụ SSH
Restart lại SSH daemon để áp dụng cấu hình cổng mới:
```bash
sudo systemctl restart ssh
```
*(Note: Đối với một số bản phân phối, lệnh có thể là `sudo systemctl restart sshd`)*

> **Lưu ý quan trọng**: Tuyệt đối không đóng cửa sổ Terminal đang cấu hình để đề phòng bị mất kết nối, chỉ mở thêm Terminal mới để kiểm tra.

### 4. Kiểm tra kết nối từ máy cá nhân

Từ Terminal mới trên máy cá nhân, kiểm tra đăng nhập qua cổng 2222 mới:
```bash
ssh -p 2222 devops@<IP_ADDRESS_DROPLET>
```

Và thử đăng nhập qua cổng 22 cũ để đảm bảo cổng này đã bị đóng:
```bash
ssh -p 22 devops@<IP_ADDRESS_DROPLET>
```

---

## Trích xuất log terminal chứng minh

```text
# Trên máy cá nhân - Test kết nối cổng mới 2222
$ ssh -p 2222 devops@192.168.1.100
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Thu Oct  5 00:35:00 2026 from 192.168.1.50
devops@droplet:~$ exit
logout
Connection to 192.168.1.100 closed.

# Trên máy cá nhân - Test kết nối cổng cũ 22 (Bị chặn)
$ ssh -p 22 devops@192.168.1.100
ssh: connect to host 192.168.1.100 port 22: Connection refused
```
