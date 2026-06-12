# Hướng Dẫn Từng Bước (Step-by-Step Guide) cho Bài Tập CDO Monitoring

Tài liệu này hướng dẫn chi tiết các bước để thực hiện và hoàn thành 2 bài tập giám sát hệ thống trên AWS:
1. **Bài tập 1 (Session 02):** Cài đặt và cấu hình CloudWatch Agent trên EC2.
2. **Bài tập 2 (Session 03):** Cấu hình CloudWatch Alarm gửi cảnh báo Email qua SNS khi CPU Utilization vượt quá 80% trong 5 phút.

---

## Chuẩn Bị Trước Khi Thực Hiện (Prerequisites)

1. **Tài khoản AWS:** Quyền truy cập vào AWS Management Console.
2. **Một máy chủ EC2:** Đã được khởi chạy (khuyên dùng Amazon Linux 2 hoặc Ubuntu).
3. **IAM Role cho EC2:** 
   - Để CloudWatch Agent có thể đẩy metric về CloudWatch, EC2 Instance cần được gán một IAM Role có đính kèm policy `CloudWatchAgentServerPolicy`.
   - **Các bước đính kèm:**
     1. Truy cập IAM Console -> Roles -> **Create role**.
     2. Chọn Trusted entity type: **AWS service**, Service or use case: **EC2**.
     3. Tại trang Add permissions, tìm kiếm và tick chọn policy: `CloudWatchAgentServerPolicy`.
     4. Đặt tên Role (ví dụ: `EC2-CloudWatchAgent-Role`) và nhấn **Create role**.
     5. Đi tới EC2 Console -> Chọn Instance của bạn -> **Actions** -> **Security** -> **Modify IAM role**.
     6. Chọn Role vừa tạo (`EC2-CloudWatchAgent-Role`) và nhấn **Update IAM role**.

---

## BÀI TẬP 1: Cài đặt CloudWatch Agent trên EC2 (Session 02)

Bài tập này giúp thu thập thêm các metric cấp hệ điều hành (OS-level metrics) như Memory, Disk Space... vốn mặc định không được CloudWatch thu thập từ EC2 hypervisor.

### Bước 1: Cài đặt gói CloudWatch Agent

Kết nối vào EC2 Instance của bạn qua SSH hoặc Systems Manager Session Manager, sau đó chạy lệnh cài đặt tương ứng với hệ điều hành:

* **Đối với Amazon Linux 2 / Amazon Linux 2023:**
  ```bash
  sudo yum install amazon-cloudwatch-agent -y
  ```

* **Đối với Ubuntu Server:**
  ```bash
  sudo apt-get update
  sudo apt-get install amazon-cloudwatch-agent -y
  ```

### Bước 2: Chạy Configuration Wizard để sinh cấu hình

AWS cung cấp một công cụ wizard tương tác để cấu hình các metric và log cần thu thập. Chạy lệnh sau:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

**Các tùy chọn gợi ý trong Wizard:**
1. **Operating System?** Chọn `1` (Linux).
2. **Are you using an EC2 or On-Premise instance?** Chọn `1` (EC2).
3. **Which user are you planning to run the agent?** Chọn `1` (cwagent) hoặc `2` (root).
4. **Do you want to turn on StatsD daemon?** Chọn `2` (No) nếu không cần đo metric ứng dụng.
5. **Do you want to turn on CollectD?** Chọn `2` (No).
6. **Do you want to monitor any host metrics?** Chọn `1` (Yes).
7. **Do you want to monitor CPU metrics at high resolution?** Chọn `1` (Yes, default 60s) hoặc tùy ý.
8. **Which default metrics config do you want?** Chọn `1` (Basic), `2` (Standard - Khuyên dùng) hoặc `3` (Advanced).
9. **Are you satisfied with the above config?** Chọn `1` (Yes).
10. **Do you have any existing CloudWatch Log Agent configuration?** Chọn `2` (No).
11. **Do you want to monitor any log files?** Chọn `2` (No) (hoặc `1` nếu bạn muốn giám sát log file cụ thể như `/var/log/messages`).
12. **Do you want to store the config in the SSM Parameter Store?** Chọn `2` (No) để lưu file cấu hình cục bộ trên EC2.

> Cấu hình sau khi hoàn thành sẽ được lưu tại: `/opt/aws/amazon-cloudwatch-agent/bin/config.json`.

### Bước 3: Khởi động và kích hoạt CloudWatch Agent

Để áp dụng cấu hình và khởi động Agent, hãy chạy các lệnh sau:

```bash
# Kích hoạt agent khởi động cùng hệ thống
sudo systemctl enable amazon-cloudwatch-agent

# Khởi chạy agent sử dụng file cấu hình vừa tạo
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json
```

### Bước 4: Kiểm tra trạng thái hoạt động của Agent

1. Chạy lệnh kiểm tra trạng thái trên EC2:
   ```bash
   sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
   ```
   *Kết quả mong đợi:* Trạng thái phải hiển thị `"status": "running"`.

2. Truy cập **CloudWatch Console** -> **Metrics** -> **All metrics**.
3. Bạn sẽ nhìn thấy một Namespace mới tên là **CWAgent**.
4. Truy cập vào **CWAgent** để kiểm tra các metric đặc thù của OS như `disk_used_percent` hoặc `mem_used_percent`.

---

## BÀI TẬP 2: Cấu hình CPU Alarm gửi Mail Alert qua SNS (Session 03)

Bài tập này thiết lập cơ chế tự động gửi thông báo qua Email khi chỉ số CPU Utilization của EC2 vượt quá 80% trong vòng 5 phút liên tục.

### Bước 1: Tạo SNS Topic & Đăng Ký Email (Subscription)

1. Truy cập **Amazon SNS (Simple Notification Service) Console**.
2. Tại menu bên trái, chọn **Topics** -> Chọn **Create topic**.
3. Cấu hình Topic:
   - **Type:** Chọn **Standard**.
   - **Name:** Nhập tên bất kỳ (ví dụ: `EC2-Alerts-Topic`).
   - Nhấn **Create topic**.
4. Sau khi tạo xong Topic, nhấn vào nút **Create subscription** để đăng ký nhận tin nhắn:
   - **Protocol:** Chọn **Email**.
   - **Endpoint:** Nhập địa chỉ Email cá nhân của bạn (ví dụ: `your-email@example.com`).
   - Nhấn **Create subscription**.
5. **Xác nhận Subscription:**
   - Mở hộp thư Email của bạn, tìm thư từ AWS Notification với tiêu đề *"AWS Notification - Subscription Confirmation"*.
   - Nhấp vào link **Confirm subscription** trong email. Trạng thái trên SNS Console sẽ chuyển từ `PendingConfirmation` sang `Confirmed`.

### Bước 2: Tạo CloudWatch Alarm

1. Truy cập **CloudWatch Console**.
2. Tại menu bên trái, chọn **Alarms** -> **All alarms** -> Chọn **Create alarm**.
3. Nhấp vào nút **Select metric**:
   - Chọn **EC2** -> **Per-Instance Metrics**.
   - Tìm kiếm Instance ID của bạn và chọn metric **CPUUtilization**.
   - Nhấn **Select metric**.

### Bước 3: Cấu hình Metric và Điều Kiện Cảnh Báo (Conditions)

1. **Metric Details:**
   - **Statistic:** Chọn `Average`.
   - **Period:** Chọn `5 minutes`.
2. **Conditions:**
   - **Threshold type:** Chọn **Static**.
   - **Whenever CPUUtilization is...** Chọn **Greater/Equal (>=)** hoặc **Greater (>)** và nhập giá trị `80`.
   - **Datapoints to alarm:** Nhập `1` out of `1` (nghĩa là 1 chu kỳ 5 phút đạt điều kiện sẽ báo động ngay).
3. Nhấn **Next**.

### Bước 4: Thiết lập Hành Động Gửi Thông Báo (Notification Action)

1. **Alarm state trigger:** Chọn **In alarm** (Kích hoạt khi ở trạng thái báo động).
2. **Send a notification to the following SNS topic:**
   - Chọn **Select an existing SNS topic**.
   - Chọn Topic bạn đã tạo ở Bước 1 (ví dụ: `EC2-Alerts-Topic`).
3. (Tùy chọn) Bạn có thể nhấn **Add notification** và chọn trigger **OK** để nhận thông báo khi CPU giảm xuống dưới 80% (Hệ thống đã phục hồi).
4. Nhấn **Next**.
5. Đặt tên cho Alarm (ví dụ: `EC2-High-CPU-Utilization-Alarm`) và nhập mô tả.
6. Nhấn **Next**, kiểm tra lại cấu hình và nhấn **Create alarm**.

### Bước 5: Kiểm tra và Giả Lập Tải CPU (Stress Test)

Để kiểm chứng Alarm và Email hoạt động chính xác, ta có thể cài đặt công cụ `stress` để nâng tải CPU của EC2 lên trên 80% trong 5 phút.

1. SSH vào EC2 và cài đặt công cụ stress:
   - **Amazon Linux:**
     ```bash
     sudo amazon-linux-extras install epel -y  # (Nếu dùng AL2)
     sudo yum install stress -y
     ```
   - **Ubuntu:**
     ```bash
     sudo apt-get install stress -y
     ```
2. Chạy lệnh stress kiểm tra tải (ví dụ máy EC2 có 1 hoặc 2 vCPU):
   ```bash
   # Tạo tải CPU liên tục trong 6 phút (360 giây)
   stress --cpu $(nproc) --timeout 360
   ```
3. Theo dõi biểu đồ trên CloudWatch Alarm. Khi trạng thái chuyển sang màu đỏ (**In Alarm**), bạn sẽ nhận được Email cảnh báo từ AWS SNS gửi về hòm thư đã đăng ký.
