# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: 05
- Product: B
- Consumer service: IoT Ingestion (Nhóm 8 - B1)
- Provider service: Core Business (B6)
- Người viết: Phan Lưu Phong
- Ngày: 12/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| `SensorEvent` | B1 dùng để đẩy dữ liệu đo đạc (nhiệt độ, độ ẩm) đã qua sơ chế sang cho B6 xử lý nghiệp vụ. | `eventId`, `deviceId`, `sensorType`, `value`, `timestamp` | `notes`, `batteryLevel` |
| `DevicePolicy` | B1 dùng để lấy cấu hình ngưỡng cảnh báo từ B6 (ví dụ: nhiệt độ > 40°C thì drop data). | `deviceId`, `thresholds`, `isActive` | `description` |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| POST | `/api/v1/core/events` | Gọi ngay khi B1 nhận được dữ liệu hợp lệ từ các thiết bị IoT cảm biến. | `201 Created` xác nhận B6 đã lưu thành công. |
| GET | `/api/v1/core/policies/devices/{deviceId}` | Gọi định kỳ (hoặc khi thiết bị mới boot) để cache lại rule xử lý dữ liệu. | `200 OK` kèm object chứa thông tin policy. |

---

## 3. Error case Consumer cần xử lý

Tối thiểu 5 case.

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Request sai schema | B1 ghi log lỗi nghiêm trọng (mapper hỏng) và dừng gửi loại gói tin này để tránh spam B6. |
| 401 | Thiếu token | B1 gọi Identity Service nội bộ để lấy/refresh lại JWT token rồi gửi lại request. |
| 403 | Không đủ quyền | Báo lỗi quyền truy cập, alert cho System Admin kiểm tra cấu hình mạng giữa B1 và B6. |
| 404 | Không tìm thấy resource | B6 không nhận diện được `deviceId`, B1 sẽ từ chối dữ liệu từ thiết bị này. |
| 409 | Xung đột nghiệp vụ | Trùng lặp `eventId`, B1 hiểu là B6 đã nhận data rồi nên sẽ bỏ qua (Skip). |
| 422 | Vi phạm rule nghiệp vụ | Dữ liệu sai logic (vd nhiệt độ < -273°C), B1 log lại và drop gói tin rác này. |

---

## 4. Giả định bổ sung

- Giả định 1: Core Business (B6) có khả năng chịu tải tốt, đáp ứng được ít nhất 1000 request/giây từ Ingestion (B1) mà không bị nghẽn.
- Giả định 2: Các API của Core Business (B6) đều hỗ trợ Idempotency (dựa vào `eventId`) để tránh việc B1 retry mạng gây trùng lặp dữ liệu.
- Giả định 3: Token xác thực (Bearer) giữa các service nội bộ được cấp phát và quản lý bởi một API Gateway hoặc Identity Service chung của Smart Campus.

---

## 5. Câu hỏi cho Provider

1. Giới hạn dung lượng tối đa (Payload Size) cho mỗi lần Ingestion gọi POST `/events` là bao nhiêu KB? Nhóm bạn có hỗ trợ nhận dữ liệu theo mảng (Batching) không?
2. Nếu Core Business (B6) bị sập (trả về 500/503), Ingestion (B1) có nên thực hiện Retry bằng Exponential Backoff không, hay cứ drop dữ liệu?
3. Thời gian sống (TTL) khuyến nghị khi Ingestion (B1) cache lại các `DevicePolicy` là bao lâu (5 phút hay 1 giờ)?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider đổi kiểu dữ liệu | Consumer parse lỗi | Chốt chặt type/format/pattern trong `openapi.yaml`. Bắt buộc Provider phải tuân thủ schema JSON. |
| Provider thiếu mã lỗi | Consumer khó xử lý lỗi | Chuẩn hóa toàn bộ lỗi 4xx/5xx theo chuẩn RFC 7807 (Problem Details). |
| Provider phản hồi chậm (High Latency) | Làm nghẽn luồng xử lý và treo hàng đợi của Consumer | Cài đặt timeout cứng gọn nhẹ (VD: 500ms) ở phía B1, nếu quá hạn thì đẩy data vào Dead-letter Queue. |