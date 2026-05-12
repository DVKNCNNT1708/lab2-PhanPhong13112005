# Phân tích yêu cầu — vai Provider

- Cặp đàm phán: 05
- Product: B
- Provider service: IoT Ingestion (Nhóm 8)
- Consumer service: Core Business (B6)
- Người viết: Phan Lưu Phong
- Ngày: 12-05-2026

---

## 1. Resource chính

| Resource | Mô tả | Thuộc tính bắt buộc | Thuộc tính tùy chọn |
|---|---|---|---|
| `TelemetryPayload` | Gói dữ liệu đo đạc đa hình từ cảm biến | `deviceId`, `timestamp`, `sensorData` | `notes` |

---

## 2. Action/API dự kiến

| Method | Path | Mục đích | Consumer gọi khi nào? |
|---|---|---|---|
| POST | `/api/v1/telemetry` | Nạp dữ liệu | Liên tục theo chu kỳ đo (ví dụ 5s/lần) |
| GET | `/api/v1/telemetry/status` | Health-check | Khi monitor trạng thái hệ thống |
| GET | `/api/v1/devices/{id}/metrics`| Xem cấu hình lấy mẫu | Khi thiết bị boot lên |
| POST | `/api/v1/devices/{id}/sync`| Yêu cầu đồng bộ | Khi cần update firmware/config |

---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống | Response body dự kiến |
|---:|---|---|
| 400 | Payload sai định dạng (thiếu timestamp) | `Problem` |
| 401 | Thiết bị không gửi kèm Token | `Problem` |
| 403 | Token của thiết bị đã bị block | `Problem` |
| 404 | Không tìm thấy cấu hình thiết bị | `Problem` |
| 409 | Request gửi dữ liệu bị trùng lặp | `Problem` |
| 422 | Dữ liệu đúng JSON nhưng nhiệt độ vượt mức logic (ví dụ > 1000 độ) | `Problem` |

---

## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- Giả định 1: Thiết bị IoT gửi dữ liệu dưới dạng JSON thay vì nhị phân (Binary) để dễ parse.
- Giả định 2: Payload tối đa không vượt quá 50KB.

---

## 5. Câu hỏi cho Consumer

1. Thiết bị có hỗ trợ gửi dữ liệu theo dạng mảng (batching) để tiết kiệm băng thông không?
2. Thiết bị dùng chuẩn thời gian nào (ISO hay UNIX)?

---

## 6. Rủi ro tích hợp

| Rủi ro | Tác động | Đề xuất xử lý |
|---|---|---|
| Dữ liệu gửi lên sai chuẩn Polymorphism | Provider không parse được `sensorType` | Trả về lỗi 400 sớm, mô tả rõ `discriminator` cần thiết. |
