# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: 05
- Product: B
- Consumer service: Core Business (B6)
- Provider service: IoT Ingestion (Nhóm 8 - B1)
- Người viết: Phan Lưu Phong (Hỗ trợ viết thay Consumer)
- Ngày: 2026-05-20

---

## 1. Resource Consumer cần nhận/gửi

| Resource | Consumer dùng để làm gì? | Field bắt buộc với Consumer | Field có thể tùy chọn |
|---|---|---|---|
| `TelemetryPayload` | Dùng để phân tích ngưỡng nhiệt độ/độ ẩm, kích hoạt cảnh báo cháy hoặc điều hòa. | `deviceId`, `timestamp`, `sensorData` | `notes` |
| `DeviceStatus` | Theo dõi để biết khi nào cảm biến bị offline. | `deviceId`, `status` | `batteryLevel` |

---

## 2. API Consumer cần gọi

| Method | Path | Lúc nào gọi? | Kỳ vọng response |
|---|---|---|---|
| POST | `/api/v1/telemetry` | (Consumer đóng vai trò nhận từ Queue, API này để dự phòng HTTP sync) | `201 Created` |
| GET | `/api/v1/telemetry` | Gọi khi hệ thống Consumer bị rớt mạng, cần pull lại data cũ. | `200 OK` kèm danh sách data |

---

## 3. Error case Consumer cần xử lý

Tối thiểu 5 case.

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào? |
|---:|---|---|
| 400 | Request sai schema | Dừng gửi, ghi log lỗi để developer sửa code. |
| 401 | Thiếu token | Tự động gọi API Identity để xin cấp lại token mới. |
| 403 | Không đủ quyền | Báo cảnh báo bảo mật, ngừng truy cập. |
| 404 | Không tìm thấy resource | Bỏ qua thao tác, coi như resource chưa được tạo. |
| 409 | Xung đột nghiệp vụ (Trùng lặp) | Bỏ qua (Skip) vì coi như đã xử lý event này rồi. |
| 422 | Vi phạm rule nghiệp vụ | Đẩy payload vào Dead-Letter Queue để xem xét bằng tay. |

## 4. Giả định bổ sung
- Giả định 1: Token xác thực (Bearer) giữa các service nội bộ được cấp phát và quản lý bởi một API Gateway hoặc Identity Service chung của Smart Campus.
- Giả định 2: Ingestion Service (B1) có cấu hình đủ tài nguyên để lưu trữ lại các message chưa được xử lý vào Dead-Letter Queue (DLQ) để B6 có thể trích xuất xử lý sau khi khắc phục sự cố.

## 5. Câu hỏi cho Provider
1. Giới hạn dung lượng tối đa (Payload Size) cho mỗi gói tin JSON mà Ingestion Service (B1) chịu tải được là bao nhiêu KB?
2. Nếu Core Business (B6) bị sập (trả về 500/503 hoặc không ACK queue), Ingestion (B1) sẽ lưu trữ message đó tối đa bao lâu (TTL) trước khi xóa vĩnh viễn?

## 6. Rủi ro tích hợp
| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Provider (B1) tự ý đổi cấu trúc JSON (đổi tên trường, đổi kiểu dữ liệu) | B6 parse lỗi dữ liệu, dẫn đến không kích hoạt được các luồng cảnh báo khẩn cấp | Chốt chặt cấu trúc type/format/pattern trong file `openapi.yaml`. Mọi thay đổi phá vỡ cấu trúc (breaking changes) phải tuân thủ chính sách Versioning (lên v2). |