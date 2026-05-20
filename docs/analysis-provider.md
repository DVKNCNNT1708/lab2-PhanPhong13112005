# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: 05
- Product: B
- Provider service: IoT Ingestion (Nhóm 8 - B1)
- Consumer service: Core Business (B6)
- Người viết: Phan Lưu Phong
- Ngày: 20-05-2026

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| `TelemetryPayload` | Gói dữ liệu đo đạc đa hình từ cảm biến (nhiệt độ, độ ẩm). | `deviceId`, `timestamp`, `sensorData` | `notes` |
| `DeviceStatus` | Trạng thái hoạt động của thiết bị cảm biến. | `deviceId`, `status`, `lastSeen` | `batteryLevel` |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| POST | `/api/v1/telemetry` | Nạp dữ liệu | Liên tục theo chu kỳ đo (ví dụ 5s/lần) |
| GET | `/api/v1/telemetry` | Lấy danh sách lịch sử đo đạc | Khi Consumer cần đồng bộ lại dữ liệu |
| GET | `/health` | Health-check | Khi monitor trạng thái hệ thống |

---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Payload sai định dạng (thiếu timestamp, sai UUID) | `Problem` |
| 401 | Thiếu Bearer token xác thực | `Problem` |
| 403 | Token hợp lệ nhưng không có quyền ghi dữ liệu | `Problem` |
| 404 | Resource không tồn tại (Truy vấn sai eventId) | `Problem` |
| 409 | Xung đột nghiệp vụ (Trùng lặp eventId / Idempotency) | `Problem` |
| 422 | Dữ liệu đúng JSON nhưng vi phạm nghiệp vụ (Nhiệt độ > 1000°C) | `Problem` |

---

## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- Giả định 1: Thiết bị IoT hoặc API Gateway đã gắn sẵn `eventId` (UUID) trước khi gọi vào Ingestion để Ingestion dùng làm Idempotency Key.
- Giả định 2: Ingestion Service chỉ lưu trữ dữ liệu nóng trong 7 ngày, sau đó dữ liệu sẽ bị xóa hoặc chuyển sang kho lưu trữ lạnh.

## 5. Năng lực đáp ứng (SLA)
- **Thời gian phản hồi API (Latency):** P99 < 200ms đối với các request POST nạp dữ liệu từ thiết bị, đảm bảo xử lý realtime.
- **Tỷ lệ hoạt động (Uptime):** Cam kết 99.9% thời gian hoạt động.
- **Độ trễ hàng đợi (Queue Delay):** Message đẩy vào queue mất tối đa 1 giây để tới được hệ thống của Consumer (trong điều kiện mạng nội bộ bình thường).

## 6. Rủi ro tích hợp
| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Lượng thiết bị IoT tăng đột biến gửi data cùng lúc (Spike traffic) | Nghẽn mạng, treo service Ingestion | Áp dụng Rate Limiting ở cấp độ API Gateway và giới hạn số lượng connection đồng thời. |
| Message Broker (RabbitMQ/Kafka) bị sập tạm thời | B1 không thể đẩy data sang B6 qua Queue | Ingestion Service (B1) sẽ thiết kế cơ chế lưu tạm (buffer) xuống Database cục bộ, khi Queue sống lại sẽ đẩy bù. |