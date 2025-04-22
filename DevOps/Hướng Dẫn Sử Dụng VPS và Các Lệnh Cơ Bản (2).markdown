# Hướng Dẫn Sử Dụng VPS và Các Lệnh Cơ Bản

Tài liệu này cung cấp hướng dẫn chi tiết về cách sử dụng **VPS (Virtual Private Server)** trên hệ điều hành Linux (ví dụ: Ubuntu), bao gồm các lệnh cơ bản để quản lý server, cài đặt công cụ như **Docker**, theo dõi trạng thái tài nguyên (RAM, CPU), và cấu hình mở cổng mạng. Tài liệu được thiết kế dành cho người mới bắt đầu, với các ví dụ thực tế và giải thích dễ hiểu.

---

## Tổng Quan Về VPS

**VPS** là một máy chủ ảo được tạo ra từ một máy chủ vật lý, cung cấp quyền truy cập root để bạn quản lý hệ thống. VPS thường được sử dụng để:

- Chạy ứng dụng web (Node.js, Django, PHP, v.v.).
- Host cơ sở dữ liệu (MySQL, MongoDB).
- Triển khai container với Docker.
- Giám sát và quản lý hệ thống.

Để sử dụng VPS, bạn cần kết nối qua **SSH** (Secure Shell) bằng công cụ như **Terminal** (Linux/macOS), **PuTTY** (Windows), hoặc **Termius**.

---

## 1. Kết Nối Với VPS

### Yêu cầu

- Địa chỉ IP của VPS (ví dụ: `192.168.1.100`).
- Tên người dùng (thường là `root` hoặc `ubuntu`).
- Mật khẩu hoặc khóa SSH (file `.pem` hoặc `.ppk`).

### Lệnh Kết Nối

1. **Kết nối bằng mật khẩu**:

   ```bash
   ssh <username>@<vps-ip>
   ```

   Ví dụ:

   ```bash
   ssh ubuntu@192.168.1.100
   ```

   Nhập mật khẩu khi được yêu cầu.

2. **Kết nối bằng khóa SSH**:

   ```bash
   ssh -i <path-to-key> <username>@<vps-ip>
   ```

   Ví dụ:

   ```bash
   ssh -i ~/.ssh/mykey.pem ubuntu@192.168.1.100
   ```

### Lưu Ý

- Đảm bảo file khóa SSH có quyền `600`:

  ```bash
  chmod 600 ~/.ssh/mykey.pem
  ```
- Nếu gặp lỗi "Connection refused", kiểm tra port SSH (mặc định: 22) hoặc firewall.

---

## 2. Các Lệnh Cơ Bản Để Quản Lý VPS

### Cập Nhật Hệ Thống

- Cập nhật danh sách gói và nâng cấp hệ thống:

  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

### Quản Lý File và Thư Mục

- **Xem danh sách file/thư mục**:

  ```bash
  ls -l # Hiển thị chi tiết
  ls -a # Hiển thị file ẩn
  ```
- **Tạo thư mục**:

  ```bash
  mkdir <folder-name>
  ```

  Ví dụ: `mkdir myapp`
- **Tạo file**:

  ```bash
  touch <file-name>
  ```

  Ví dụ: `touch config.txt`
- **Sao chép file/thư mục**:

  ```bash
  cp <source> <destination>
  ```

  Ví dụ: `cp config.txt config_backup.txt`
- **Di chuyển/đổi tên**:

  ```bash
  mv <source> <destination>
  ```

  Ví dụ: `mv config.txt /home/ubuntu/`
- **Xóa file/thư mục**:

  ```bash
  rm <file> # Xóa file
  rm -r <folder> # Xóa thư mục
  ```

### Quản Lý Quyền

- **Thay đổi quyền**:

  ```bash
  chmod <permissions> <file>
  ```

  Ví dụ: `chmod 644 config.txt` (chủ sở hữu đọc/ghi, người khác chỉ đọc).
- **Thay đổi chủ sở hữu**:

  ```bash
  chown <user>:<group> <file>
  ```

  Ví dụ: `chown ubuntu:ubuntu config.txt`

### Quản Lý User

- **Thêm user**:

  ```bash
  sudo adduser <username>
  ```
- **Thêm user vào nhóm sudo**:

  ```bash
  sudo usermod -aG sudo <username>
  ```

---

## 3. Cài Đặt Docker

**Docker** là nền tảng để chạy ứng dụng trong các container, giúp triển khai và quản lý ứng dụng dễ dàng.

### Bước 1: Cài Đặt Docker

1. Cập nhật hệ thống:

   ```bash
   sudo apt update
   ```
2. Cài đặt các gói cần thiết:

   ```bash
   sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
   ```
3. Thêm GPG key của Docker:

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```
4. Thêm repository Docker:

   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```
5. Cài đặt Docker:

   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io
   ```

### Bước 2: Kích Hoạt và Kiểm Tra Docker

- Khởi động Docker:

  ```bash
  sudo systemctl enable docker
  sudo systemctl start docker
  ```
- Kiểm tra trạng thái:

  ```bash
  sudo systemctl status docker
  ```
- Kiểm tra phiên bản:

  ```bash
  docker --version
  ```

### Bước 3: Cho Phép User Không Root Chạy Docker

- Thêm user vào nhóm `docker`:

  ```bash
  sudo usermod -aG docker $USER
  ```
- Đăng xuất và đăng nhập lại để áp dụng.

### Bước 4: Chạy Một Container Ví Dụ

- Chạy container Nginx:

  ```bash
  docker run -d -p 80:80 nginx
  ```
- Truy cập `http://<vps-ip>` để kiểm tra Nginx.

### Các Lệnh Docker Cơ Bản

- **Xem danh sách container**:

  ```bash
  docker ps # Container đang chạy
  docker ps -a # Tất cả container
  ```
- **Dừng container**:

  ```bash
  docker stop <container-id>
  ```
- **Xóa container**:

  ```bash
  docker rm <container-id>
  ```
- **Xem danh sách image**:

  ```bash
  docker images
  ```
- **Xóa image**:

  ```bash
  docker rmi <image-id>
  ```

---

## 4. Theo Dõi Trạng Thái Tài Nguyên (RAM, CPU, Disk)

### Kiểm Tra CPU

- **Sử dụng** `top` **hoặc** `htop`:

  ```bash
  top
  ```

  hoặc

  ```bash
  sudo apt install htop -y
  htop
  ```
  - **Giải thích**:
    - `%CPU`: Tỷ lệ sử dụng CPU của mỗi process.
    - `load average`: Tải trung bình trong 1, 5, 15 phút (so với số core CPU).
- **Xem thông tin CPU**:

  ```bash
  lscpu
  ```

  Hiển thị số core, tốc độ, v.v.

### Kiểm Tra RAM

- **Sử dụng** `free`:

  ```bash
  free -h
  ```
  - **Giải thích**:
    - `total`: Tổng RAM.
    - `used`: RAM đang sử dụng.
    - `free`: RAM còn trống.
    - `available`: RAM khả dụng cho ứng dụng mới.
- **Xem chi tiết**:

  ```bash
  cat /proc/meminfo
  ```

### Kiểm Tra Disk

- **Sử dụng** `df`:

  ```bash
  df -h
  ```

  Hiển thị dung lượng đĩa, phần trăm sử dụng.
- **Xem I/O**:

  ```bash
  iostat -x 1
  ```

  Cần cài đặt: `sudo apt install sysstat -y`.

### Kiểm Tra Network

- **Xem lưu lượng mạng**:

  ```bash
  iftop
  ```

  Cần cài đặt: `sudo apt install iftop -y`.
- **Xem port đang mở**:

  ```bash
  netstat -tuln
  ```

  Cần cài đặt: `sudo apt install net-tools -y`.

---

## 5. Mở Cổng Mạng và Quản Lý Firewall

### Kiểm Tra Port

- Xem các port đang mở:

  ```bash
  ss -tuln
  ```

  hoặc

  ```bash
  netstat -tuln
  ```

### Sử Dụng UFW (Uncomplicated Firewall)

- **Cài đặt UFW**:

  ```bash
  sudo apt install ufw -y
  ```
- **Kích hoạt UFW**:

  ```bash
  sudo ufw enable
  ```
- **Mở cổng**:

  ```bash
  sudo ufw allow <port>/<protocol>
  ```

  Ví dụ:

  ```bash
  sudo ufw allow 80/tcp # Mở port 80 cho HTTP
  sudo ufw allow 22/tcp # Mở port 22 cho SSH
  sudo ufw allow 443/tcp # Mở port 443 cho HTTPS
  ```
- **Đóng cổng**:

  ```bash
  sudo ufw deny <port>/<protocol>
  ```
- **Xem trạng thái UFW**:

  ```bash
  sudo ufw status
  ```
- **Tắt UFW (nếu cần)**:

  ```bash
  sudo ufw disable
  ```

### Mở Cổng Với iptables (Nâng Cao)

- **Xem rules hiện tại**:

  ```bash
  sudo iptables -L -n -v
  ```
- **Mở cổng**:

  ```bash
  sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
  ```
- **Lưu rules**:

  ```bash
  sudo apt install iptables-persistent -y
  sudo iptables-save > /etc/iptables/rules.v4
  ```

---

## 6. Các Lệnh Hữu Ích Khác

### Quản Lý Process

- **Xem danh sách process**:

  ```bash
  ps aux
  ```
- **Kết thúc process**:

  ```bash
  kill <pid>
  ```

  hoặc

  ```bash
  kill -9 <pid> # Buộc dừng
  ```

### Kiểm Tra Uptime

- Xem thời gian server hoạt động:

  ```bash
  uptime
  ```

### Tải File Từ URL

- Sử dụng `wget` hoặc `curl`:

  ```bash
  wget <url>
  ```

  hoặc

  ```bash
  curl -O <url>
  ```

### Nén/Giải Nén File

- **Nén**:

  ```bash
  tar -czvf archive.tar.gz <folder>
  ```
- **Giải nén**:

  ```bash
  tar -xzvf archive.tar.gz
  ```

### Kiểm Tra Phiên Bản Hệ Điều Hành

- Xem thông tin OS:

  ```bash
  lsb_release -a
  ```

  hoặc

  ```bash
  cat /etc/os-release
  ```

---

## 7. Mẹo Quản Lý VPS Hiệu Quả

- **Bảo mật**:

  - Đổi port SSH mặc định (22) để tránh tấn công:

    ```bash
    sudo nano /etc/ssh/sshd_config
    # Thay Port 22 thành Port 2222 (hoặc khác)
    sudo systemctl restart sshd
    ```
  - Vô hiệu hóa đăng nhập root qua SSH:

    ```bash
    sudo nano /etc/ssh/sshd_config
    # Đặt PermitRootLogin no
    sudo systemctl restart sshd
    ```
  - Cài đặt **fail2ban** để chặn IP tấn công:

    ```bash
    sudo apt install fail2ban -y
    sudo systemctl enable fail2ban
    ```

- **Sao lưu**:

  - Sao lưu dữ liệu thường xuyên:

    ```bash
    rsync -av /path/to/data /path/to/backup
    ```
  - Sử dụng dịch vụ backup của nhà cung cấp VPS (nếu có).

- **Giám sát**:

  - Cài đặt công cụ giám sát như **Prometheus**, **Node Exporter**, và **Grafana** (như tài liệu trước).
  - Sử dụng `htop` hoặc `glances` để theo dõi trực quan:

    ```bash
    sudo apt install glances -y
    glances
    ```

---

## Lưu Ý

- **Kiểm tra quyền**: Luôn chạy lệnh `sudo` khi cần quyền root, nhưng cẩn thận để tránh xóa nhầm dữ liệu.
- **Firewall**: Đảm bảo mở đúng cổng cần thiết (80, 443, 22) và đóng các cổng không dùng.
- **Cập nhật thường xuyên**: Chạy `sudo apt update && sudo apt upgrade` định kỳ để bảo mật.
- **Log lỗi**: Nếu gặp lỗi, kiểm tra log hệ thống:

  ```bash
  tail -f /var/log/syslog
  ```

  hoặc

  ```bash
  journalctl -u <service>
  ```

  Ví dụ: `journalctl -u docker`.

---

Tài liệu này cung cấp các lệnh cơ bản và hướng dẫn thực tế để quản lý VPS, từ kết nối SSH, cài đặt Docker, đến theo dõi tài nguyên và cấu hình firewall. Nếu bạn cần bổ sung chi tiết (ví dụ: cài đặt Nginx, MySQL, hoặc tích hợp CI/CD), hãy cho tôi biết!