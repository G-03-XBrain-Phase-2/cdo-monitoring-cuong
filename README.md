# Báo Cáo Minh Chứng Hoàn Thành Bài Tập (Evidence File)

*Học viên điền các thông tin minh chứng và chèn ảnh chụp màn hình tương ứng vào các phần dưới đây để hoàn thiện báo cáo.*

---

## 1. Thông Tin Chung
- **Họ và tên:** [Điền họ tên của bạn]
- **Tên lớp / Session:** CDO Monitoring (Session 02 & 03)
- **ID của Instance EC2 đã sử dụng:** `i-xxxxxxxxxxxxxxxxx`
- **Địa chỉ IP Public của EC2:** `xx.xx.xx.xx`

---

## 2. Minh Chứng Bài Tập 1: Cài Đặt CloudWatch Agent trên EC2

### 2.1. Kết quả kiểm tra trạng thái dịch vụ trên máy chủ EC2
*Chạy lệnh sau trên EC2 và sao chép kết quả (output) dán vào đây:*
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

*Kết quả đầu ra thực tế:*
```json
// Dán output JSON trạng thái CloudWatch Agent của bạn ở đây
```

### 2.2. Kiểm tra log của CloudWatch Agent (Tùy chọn)
*Chạy lệnh kiểm tra log gần nhất để đảm bảo Agent đẩy metric thành công:*
```bash
tail -n 20 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

### 2.3. Ảnh chụp màn hình Namespace `CWAgent` trên CloudWatch Console
*Hãy chụp ảnh màn hình trang CloudWatch Console -> Metrics -> All metrics cho thấy Namespace **CWAgent** đã xuất hiện và hiển thị các custom metrics (ví dụ: `disk_used_percent`, `mem_used_percent`).*

> **[Chèn ảnh chụp màn hình Namespace CWAgent tại đây]**
> Ví dụ: `![CWAgent Metrics Console](./images/cwagent-metrics.png)`

---

## 3. Minh Chứng Bài Tập 2: Cấu Hình CPU Alarm và Gửi Mail Alert via SNS

### 3.1. Ảnh chụp màn hình Subscription đã được xác nhận (Confirmed) trong SNS Console
*Truy cập Amazon SNS -> Subscriptions. Chụp ảnh màn hình dòng subscription của bạn có trạng thái **Confirmed**.*

> **[Chèn ảnh chụp màn hình SNS Subscription tại đây]**
> Ví dụ: `![SNS Subscription Confirmed](./images/sns-subscription.png)`

### 3.2. Ảnh chụp màn hình Cấu hình CloudWatch Alarm
*Truy cập CloudWatch -> Alarms -> Chọn Alarm vừa tạo. Chụp ảnh cấu hình tổng quan bao gồm Tên Alarm, Điều kiện (CPU > 80% trong 5 phút).*

> **[Chèn ảnh chụp màn hình Alarm Configuration tại đây]**
> Ví dụ: `![CloudWatch Alarm Configuration](./images/cw-alarm-config.png)`

### 3.3. Ảnh chụp màn hình đồ thị Alarm chuyển sang trạng thái "In Alarm"
*Sau khi chạy lệnh `stress` trên EC2, chụp ảnh màn hình đồ thị CPU Utilization tăng vượt ngưỡng và chuyển sang trạng thái màu đỏ (In Alarm).*

> **[Chèn ảnh chụp màn hình Đồ thị In Alarm tại đây]**
> Ví dụ: `![CloudWatch Alarm In Alarm](./images/cw-alarm-triggered.png)`

### 3.4. Ảnh chụp màn hình Email Cảnh Báo từ AWS SNS
*Mở hòm thư cá nhân của bạn, chụp ảnh email thông báo tự động gửi từ AWS với trạng thái `ALARM: "EC2-High-CPU-Utilization-Alarm"`.*

> **[Chèn ảnh chụp màn hình Email Cảnh Báo tại đây]**
> Ví dụ: `![SNS Email Alert](./images/sns-email-alert.png)`
