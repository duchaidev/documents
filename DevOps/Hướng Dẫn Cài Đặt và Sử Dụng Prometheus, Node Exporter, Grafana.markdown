# Hướng Dẫn Cài Đặt và Sử Dụng Prometheus, Node Exporter, Grafana

Tài liệu này cung cấp cái nhìn tổng quan về **Prometheus**, **Node Exporter**, và **Grafana**, giải thích công dụng của từng công cụ và hướng dẫn cách cài đặt, cấu hình, sử dụng chúng để giám sát hệ thống một cách hiệu quả.

---

## Tổng Quan và Công Dụng

### 1. Node Exporter – “Người Thu Thập Dữ Liệu Hệ Thống”

- **Công dụng**:
  - Thu thập các chỉ số (metrics) từ hệ thống của server, bao gồm:
    - **CPU**: Tỷ lệ sử dụng, load average, hiệu suất từng core.
    - **RAM**: Dung lượng sử dụng, còn trống.
    - **Disk**: Dung lượng, tốc độ đọc/ghi.
    - **Network**: Lưu lượng, tốc độ truyền/nhận.
    - **Hệ thống**: Uptime, file system, I/O, số lượng process.
  - Chạy dưới dạng một chương trình nhỏ trên mỗi server cần giám sát.
  - Cung cấp dữ liệu qua HTTP endpoint để Prometheus thu thập.
- **Ví dụ so sánh**: Giống như một cảm biến đo nhiệt độ, liên tục gửi dữ liệu về các thông số hệ thống.
- **Khi nào cần**: Dùng để giám sát tài nguyên phần cứng của server (Linux, Windows, v.v.).

### 2. Prometheus – “Bộ Não Lưu Trữ và Xử Lý Dữ Liệu”

- **Công dụng**:
  - **Thu thập dữ liệu**: Kéo (pull) metrics từ Node Exporter hoặc các dịch vụ khác qua HTTP.
  - **Lưu trữ**: Lưu dữ liệu dạng time-series (theo thời gian) trong cơ sở dữ liệu nội bộ.
  - **Truy vấn**: Sử dụng ngôn ngữ **PromQL** để phân tích và truy vấn dữ liệu (ví dụ: tỷ lệ CPU trung bình trong 5 phút).
  - **Cảnh báo**: Gửi cảnh báo (qua email, Slack, Telegram, v.v.) khi chỉ số vượt ngưỡng (ví dụ: CPU > 90%).
  - **Tích hợp**: Kết nối với Grafana hoặc các công cụ khác để trực quan hóa.
- **Ví dụ so sánh**: Giống như tổng đài nhận thông tin từ các cảm biến (Node Exporter), lưu trữ, xử lý, và phát cảnh báo khi cần.
- **Khi nào cần**: Khi bạn cần một hệ thống giám sát tập trung, phân tích và cảnh báo dựa trên dữ liệu thời gian thực.

### 3. Grafana – “Màn Hình Trực Quan Hóa”

- **Công dụng**:
  - Kết nối với Prometheus (hoặc các nguồn dữ liệu khác như MySQL, InfluxDB).
  - Tạo các **dashboard** trực quan với biểu đồ, bảng, hoặc số liệu:
    - Theo dõi CPU, RAM, Disk, Network theo thời gian thực.
    - Hiển thị trạng thái hệ thống qua các biểu đồ như line chart, gauge, heatmap.
    - Tự động làm mới dữ liệu (refresh) để cập nhật liên tục.
  - Hỗ trợ thiết lập **alert** dựa trên dữ liệu từ Prometheus.
  - Cho phép chia sẻ dashboard với team hoặc tích hợp vào hệ thống nội bộ.
- **Ví dụ so sánh**: Giống như màn hình trung tâm trong phòng điều khiển, giúp con người dễ dàng hiểu và theo dõi trạng thái hệ thống.
- **Khi nào cần**: Khi bạn muốn trực quan hóa dữ liệu từ Prometheus hoặc các nguồn khác để dễ dàng phân tích và đưa ra quyết định.

---

## Hướng Dẫn Cài Đặt và Sử Dụng

Dưới đây là hướng dẫn chi tiết để cài đặt và cấu hình **Node Exporter**, **Prometheus**, và **Grafana** trên một server Linux (ví dụ: Ubuntu). Các bước này giả định bạn có quyền `sudo` và server đã cài đặt các công cụ cơ bản như `wget`, `tar`, và `systemctl`.

### 1. Cài Đặt Node Exporter

#### Bước 1: Tải và giải nén Node Exporter

```bash
# Tải phiên bản mới nhất (kiểm tra phiên bản trên https://github.com/prometheus/node_exporter/releases)
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz

# Giải nén
tar xvfz node_exporter-1.8.2.linux-amd64.tar.gz
cd node_exporter-1.8.2.linux-amd64
```

#### Bước 2: Di chuyển và chạy Node Exporter

```bash
# Di chuyển file thực thi
sudo mv node_exporter /usr/local/bin/

# Tạo user để chạy Node Exporter
sudo useradd --no-create-home --shell /bin/false node_exporter

# Tạo service systemd
sudo nano /etc/systemd/system/node_exporter.service
```

Nội dung file `node_exporter.service`:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

#### Bước 3: Khởi động Node Exporter

```bash
# Kích hoạt và khởi động service
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# Kiểm tra trạng thái
sudo systemctl status node_exporter
```

#### Bước 4: Kiểm tra Node Exporter

- Mở trình duyệt và truy cập: `http://<server-ip>:9100/metrics`
- Bạn sẽ thấy danh sách các metrics hệ thống mà Node Exporter thu thập.

**Lưu ý**: Cài đặt Node Exporter trên **mỗi server** cần giám sát.

---

### 2. Cài Đặt Prometheus

#### Bước 1: Tải và giải nén Prometheus

```bash
# Tải phiên bản mới nhất (kiểm tra trên https://github.com/prometheus/prometheus/releases)
wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz

# Giải nén
tar xvfz prometheus-2.54.1.linux-amd64.tar.gz
cd prometheus-2.54.1.linux-amd64
```

#### Bước 2: Cấu hình Prometheus

- Tạo thư mục lưu trữ và di chuyển file:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/
```

- Chỉnh sửa file cấu hình `/etc/prometheus/prometheus.yml` để thêm Node Exporter:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["<node-exporter-ip>:9100"] # Thay bằng IP của server chạy Node Exporter
```

#### Bước 3: Tạo service cho Prometheus

```bash
# Tạo user
sudo useradd --no-create-home --shell /bin/false prometheus

# Phân quyền
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus

# Tạo service
sudo nano /etc/systemd/system/prometheus.service
```

Nội dung file `prometheus.service`:

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

#### Bước 4: Khởi động Prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

#### Bước 5: Kiểm tra Prometheus

- Truy cập giao diện web Prometheus: `http://<prometheus-ip>:9090`
- Vào mục **Status > Targets** để kiểm tra xem Node Exporter có được kết nối không.
- Sử dụng PromQL để truy vấn, ví dụ: `node_cpu_seconds_total` để xem CPU usage.

---

### 3. Cài Đặt Grafana

#### Bước 1: Cài đặt Grafana

```bash
# Thêm repository của Grafana
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

# Cài đặt
sudo apt-get update
sudo apt-get install grafana
```

#### Bước 2: Khởi động Grafana

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

#### Bước 3: Kết nối Grafana với Prometheus

1. Truy cập giao diện Grafana: `http://<grafana-ip>:3000`
2. Đăng nhập với tài khoản mặc định: `admin/admin` (đổi mật khẩu ngay sau đó).
3. Thêm nguồn dữ liệu:
   - Vào **Configuration > Data Sources > Add data source**.
   - Chọn **Prometheus**.
   - Nhập URL của Prometheus: `http://<prometheus-ip>:9090`.
   - Nhấn **Save & Test**.

#### Bước 4: Tạo Dashboard

1. Vào **Create > Dashboard > Add new panel**.
2. Chọn nguồn dữ liệu là Prometheus.
3. Nhập truy vấn PromQL, ví dụ:
   - CPU usage: `100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`
   - RAM usage: `100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)`
4. Tùy chỉnh biểu đồ (line, gauge, bar) và lưu dashboard.

#### Bước 5: Thiết lập Alert (Tùy chọn)

1. Trong dashboard, chọn panel và vào tab **Alert**.
2. Tạo rule, ví dụ: Gửi cảnh báo nếu CPU > 90% trong 5 phút.
3. Cấu hình kênh thông báo (email, Slack, Telegram) trong **Alerting > Notification channels**.

---

## Cách Sử Dụng

### 1. Sử Dụng Node Exporter

- **Kiểm tra metrics**: Truy cập `http://<node-exporter-ip>:9100/metrics` để xem dữ liệu thô.
- **Giám sát nhiều server**: Cài Node Exporter trên từng server và thêm vào cấu hình Prometheus.

### 2. Sử Dụng Prometheus

- **Truy vấn dữ liệu**:
  - Vào giao diện Prometheus (`http://<prometheus-ip>:9090`).
  - Dùng PromQL để phân tích, ví dụ:
    - CPU usage: `rate(node_cpu_seconds_total{mode="user"}[5m])`
    - Disk I/O: `rate(node_disk_io_time_seconds_total[5m])`
- **Cảnh báo**:
  - Chỉnh sửa file `prometheus.yml` để thêm rules cảnh báo:
    ```yaml
    alerting:
      alertmanagers:
        - static_configs:
            - targets: ["<alertmanager-ip>:9093"]
    rule_files:
      - "alert_rules.yml"
    ```
  - Tạo file `alert_rules.yml`:
    ```yaml
    groups:
      - name: example
        rules:
          - alert: HighCPUUsage
            expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
            for: 5m
            labels:
              severity: critical
            annotations:
              summary: "High CPU usage detected on {{ $labels.instance }}"
    ```

### 3. Sử Dụng Grafana

- **Tạo dashboard**:
  - Sử dụng các mẫu dashboard có sẵn từ [Grafana Labs](https://grafana.com/grafana/dashboards/) (ví dụ: dashboard ID 1860 cho Node Exporter).
  - Nhập ID dashboard vào Grafana để import.
- **Tùy chỉnh**:
  - Thêm các panel mới với các chỉ số như CPU, RAM, Network.
  - Sử dụng biến (variables) để chọn server động (ví dụ: chọn instance từ danh sách Node Exporter).
- **Chia sẻ**:
  - Xuất dashboard dưới dạng JSON hoặc chia sẻ link với team.

---

### 3. Grafana – “Màn Hình Trực Quan Hóa”

- **Công dụng**:
  - Kết nối Prometheus để tạo dashboard trực quan (biểu đồ, gauge).
  - Thiết lập cảnh báo qua email, Slack, Telegram, v.v.
- **Ví dụ so sánh**: Như màn hình trung tâm hiển thị dữ liệu dễ hiểu.

---

## Hướng Dẫn Cài Đặt và Sử Dụng

Các bước cài đặt **Node Exporter**, **Prometheus**, và **Grafana** đã được trình bày trong tài liệu trước. Dưới đây là hướng dẫn bổ sung để tích hợp **Dashboard 1860** và thiết lập **cảnh báo Telegram** khi CPU hoặc RAM vượt quá 40%.

### 1. Tích Hợp Dashboard 1860 (Node Exporter Full)

**Dashboard 1860** là một dashboard phổ biến trên Grafana Labs, được thiết kế để trực quan hóa toàn diện các chỉ số từ Node Exporter, bao gồm CPU, RAM, Disk, Network, và nhiều thông số khác.

#### Bước 1: Truy cập Grafana

- Mở giao diện Grafana tại `http://<grafana-ip>:3000`.
- Đăng nhập với tài khoản (mặc định: `admin/admin`).

#### Bước 2: Import Dashboard 1860

1. Vào **Create > Import**.
2. Nhập ID dashboard: `1860` hoặc dán URL: `https://grafana.com/grafana/dashboards/1860`.
3. Nhấn **Load**.
4. Chọn **Prometheus** làm nguồn dữ liệu (data source) trong dropdown.
5. (Tùy chọn) Đặt tên dashboard (ví dụ: "Node Exporter Full").
6. Nhấn **Import**.

#### Bước 3: Kiểm Tra Dashboard

- Sau khi import, dashboard sẽ hiển thị các panel như:
  - **CPU Usage**: Tỷ lệ sử dụng CPU theo thời gian.
  - **Memory Usage**: RAM sử dụng, còn trống.
  - **Disk I/O**: Tốc độ đọc/ghi ổ đĩa.
  - **Network Traffic**: Lưu lượng mạng.
- Chọn **instance** (server chạy Node Exporter) từ dropdown ở đầu dashboard để xem dữ liệu cụ thể.

#### Bước 4: Tùy Chỉnh (Tùy Chọn)

- Thêm biến (variable) để chọn nhiều server:
  1. Vào **Dashboard settings > Variables > Add variable**.
  2. Đặt tên: `instance`, loại: `Query`.
  3. Query PromQL: `label_values(node_uname_info, instance)`.
  4. Áp dụng để dashboard tự động cập nhật danh sách server.
- Tùy chỉnh panel: Nhấp vào tiêu đề panel, chọn **Edit** để thay đổi PromQL hoặc kiểu biểu đồ.

#### Lưu Ý

- Đảm bảo Node Exporter và Prometheus đang hoạt động (kiểm tra `http://<prometheus-ip>:9090/targets`).
- Nếu dashboard không hiển thị dữ liệu, kiểm tra lại nguồn dữ liệu Prometheus trong Grafana.

---

### 2. Thiết Lập Cảnh Báo Telegram Khi CPU và RAM Vượt Quá 40%

Grafana hỗ trợ gửi cảnh báo qua Telegram bằng cách sử dụng **webhook** hoặc **bot**. Dưới đây là hướng dẫn chi tiết để thiết lập cảnh báo khi CPU hoặc RAM vượt quá 40%.

#### Bước 1: Tạo Bot Telegram

1. Mở Telegram, tìm `@BotFather`.
2. Gửi lệnh `/start` và `/newbot`.
3. Đặt tên và username cho bot (ví dụ: `@MyMonitoringBot`).
4. Lưu **token** của bot (ví dụ: `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`).
5. Tạo một nhóm Telegram và thêm bot vào nhóm.
6. Lấy **chat ID** của nhóm:
   - Gửi một tin nhắn bất kỳ trong nhóm.
   - Truy cập: `https://api.telegram.org/bot<your-bot-token>/getUpdates`.
   - Tìm `chat.id` trong kết quả (ví dụ: `-123456789`).

#### Bước 2: Cấu Hình Kênh Thông Báo Trong Grafana

1. Truy cập Grafana (`http://<grafana-ip>:3000`).
2. Vào **Alerting > Notification channels > New channel**.
3. Cấu hình kênh:
   - **Name**: `Telegram Alert`
   - **Type**: `Telegram`
   - **Bot API Token**: Dán token từ BotFather (ví dụ: `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`).
   - **Chat ID**: Dán chat ID của nhóm (ví dụ: `-123456789`).
4. Nhấn **Send Test** để kiểm tra (bot sẽ gửi tin nhắn thử vào nhóm).
5. Nhấn **Save**.

#### Bước 3: Tạo Quy Tắc Cảnh Báo Cho CPU

1. Vào dashboard 1860, chọn panel **CPU Usage** (hoặc tạo panel mới).
2. Nhấp vào tiêu đề panel, chọn **Edit**.
3. Trong tab **Alert**, nhấn **Create Alert**.
4. Cấu hình quy tắc:
   - **Name**: `High CPU Usage`
   - **Condition**: Sử dụng truy vấn PromQL:
     ```promql
     100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 40
     ```
   - **Evaluate every**: `1m` (kiểm tra mỗi phút).
   - **For**: `5m` (gửi cảnh báo nếu vượt ngưỡng trong 5 phút).
   - **Notification**: Chọn kênh `Telegram Alert`.
   - **Message**:
     ```
     🚨 High CPU Usage on {{ $labels.instance }}
     CPU: {{ $value }}% (Threshold: 40%)
     ```
5. Nhấn **Save**.

#### Bước 4: Tạo Quy Tắc Cảnh Báo Cho RAM

1. Tương tự, chọn panel **Memory Usage** hoặc tạo panel mới.
2. Trong tab **Alert**, nhấn **Create Alert**.
3. Cấu hình quy tắc:
   - **Name**: `High Memory Usage`
   - **Condition**: Sử dụng truy vấn PromQL:
     ```promql
     100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) > 40
     ```
   - **Evaluate every**: `1m`.
   - **For**: `5m`.
   - **Notification**: Chọn kênh `Telegram Alert`.
   - **Message**:
     ```
     🚨 High Memory Usage on {{ $labels.instance }}
     Memory: {{ $value }}% (Threshold: 40%)
     ```
4. Nhấn **Save**.

#### Bước 5: Kiểm Tra Cảnh Báo

- Tăng tải CPU hoặc RAM trên server (ví dụ: chạy lệnh `stress --cpu 8` để tải CPU).
- Kiểm tra nhóm Telegram để đảm bảo nhận được tin nhắn cảnh báo.
- Xem lịch sử cảnh báo trong Grafana: **Alerting > Alerts**.

---

## Cách Sử Dụng

### Sử Dụng Dashboard 1860

- **Theo dõi hệ thống**: Mở dashboard 1860 để xem trạng thái CPU, RAM, Disk, Network theo thời gian thực.
- **Chuyển đổi server**: Sử dụng dropdown `instance` để chọn server cần giám sát.
- **Tùy chỉnh giao diện**: Thay đổi kiểu biểu đồ (line, bar, gauge) hoặc thêm panel mới với PromQL tùy chỉnh.
- **Chia sẻ**: Xuất dashboard dưới dạng JSON hoặc chia sẻ link với team.

### Sử Dụng Cảnh Báo Telegram

- **Theo dõi cảnh báo**: Kiểm tra nhóm Telegram để nhận thông báo khi CPU hoặc RAM vượt 40%.
- **Tùy chỉnh ngưỡng**: Điều chỉnh ngưỡng (ví dụ: 50% thay vì 40%) trong truy vấn PromQL.
- **Mở rộng**: Thêm cảnh báo cho các chỉ số khác như Disk usage (`node_filesystem_free_bytes`) hoặc Network traffic (`node_network_receive_bytes_total`).

---

## Lưu Ý

- **Bảo mật Telegram**:
  - Giữ token bot và chat ID bí mật, tránh chia sẻ công khai.
  - Sử dụng nhóm Telegram riêng cho cảnh báo để dễ quản lý.
- **Tối ưu cảnh báo**:
  - Tránh đặt ngưỡng quá thấp (dưới 40%) để không nhận quá nhiều cảnh báo.
  - Sử dụng `for: 5m` để chỉ gửi cảnh báo khi vấn đề kéo dài, tránh cảnh báo giả.
- **Khắc phục sự cố**:
  - Nếu không nhận được tin nhắn Telegram:
    - Kiểm tra token và chat ID trong Grafana.
    - Đảm bảo bot không bị chặn trong nhóm.
  - Nếu dashboard 1860 không hiển thị dữ liệu:
    - Kiểm tra kết nối Prometheus trong Grafana.
    - Đảm bảo Node Exporter đang chạy và được thêm vào `prometheus.yml`.
- **Mở rộng**:
  - Tích hợp **Alertmanager** với Prometheus để quản lý cảnh báo phức tạp hơn (nhóm, lặp lại, v.v.).
  - Thêm các dashboard khác từ Grafana Labs (ví dụ: 405 cho Docker, 11074 cho Kubernetes).

---

Tài liệu này cung cấp hướng dẫn chi tiết để tích hợp **Dashboard 1860** và thiết lập **cảnh báo Telegram** cho CPU/RAM vượt 40% trong Grafana, bổ sung cho hệ thống giám sát với **Prometheus** và **Node Exporter**. Nếu bạn cần thêm hướng dẫn (ví dụ: tích hợp Alertmanager, thêm dashboard khác, hoặc cấu hình SSL), hãy cho tôi biết!

## Lưu Ý

- **Bảo mật**:
  - Thêm firewall (ví dụ: `ufw`) để giới hạn truy cập vào các port (9100, 9090, 3000).
  - Sử dụng Nginx làm reverse proxy và bật SSL cho Grafana/Prometheus.
- **Tối ưu**:
  - Giới hạn thời gian lưu trữ dữ liệu trong Prometheus (`--storage.tsdb.retention.time=15d`).
  - Sử dụng Grafana Cloud hoặc Loki để lưu trữ log dài hạn.
- **Khắc phục sự cố**:
  - Nếu Node Exporter không gửi dữ liệu: Kiểm tra firewall, port 9100, và trạng thái service.
  - Nếu Grafana không kết nối được với Prometheus: Đảm bảo URL Prometheus chính xác và không bị chặn.

---

Tài liệu này cung cấp hướng dẫn toàn diện để cài đặt và sử dụng bộ ba **Prometheus**, **Node Exporter**, và **Grafana** nhằm giám sát hệ thống một cách hiệu quả. Bạn có thể mở rộng bằng cách tích hợp thêm các công cụ như **Alertmanager** hoặc **Loki** để tăng cường khả năng cảnh báo và quản lý log. Nếu bạn cần bổ sung chi tiết hoặc hướng dẫn cụ thể hơn (ví dụ: tích hợp Telegram cho alert), hãy cho tôi biết!
