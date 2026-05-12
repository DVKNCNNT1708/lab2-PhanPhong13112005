# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: 05 (IoT Ingestion - Core Business)
- Product: B
- Provider: IoT Ingestion (Phan Lưu Phong, Trần Phong)
- Consumer: Core Business (Ngô Văn Quân, Trần Thanh Bình)
- Phiên: v1.0
- Ngày: 2026-05-12

---

## Issue #1: Định dạng thời gian
- Raised by: Provider
- Concern: Dùng Unix timestamp hay ISO 8601?
- Proposal: Dùng ISO 8601 `date-time` để đồng bộ với các service khác.
- Resolution: Accepted
- Rationale: Dễ đọc và không cần convert múi giờ ở các service tiêu thụ.
- Impact: Schema `timestamp` dùng `format: date-time`.

## Issue #2: Cấu trúc báo lỗi
- Raised by: Consumer
- Concern: Cần chuẩn hóa lỗi để Consumer dễ xử lý tự động.
- Proposal: Áp dụng RFC 7807 (Problem Details).
- Resolution: Accepted
- Rationale: Là chuẩn công nghiệp giúp mô tả lỗi chi tiết và nhất quán.
- Impact: Thêm `application/problem+json` vào các response 4xx.

## Issue #3: Cơ chế phân trang
- Raised by: Provider
- Concern: Dữ liệu sensor rất lớn, dùng Offset pagination sẽ chậm.
- Proposal: Dùng Cursor-based pagination.
- Resolution: Accepted
- Rationale: Hiệu suất cao hơn đối với dữ liệu time-series.
- Impact: Thêm field `cursor` và `nextCursor`.

## Issue #4: Xử lý đa hình sensor
- Raised by: Provider
- Concern: Làm sao để gửi nhiều loại sensor (Nhiệt độ, Độ ẩm) chung 1 endpoint?
- Proposal: Dùng `oneOf` kèm `discriminator` trên trường `sensorType`.
- Resolution: Accepted
- Rationale: Giúp API mở rộng linh hoạt mà không cần thêm path mới.
- Impact: Schema `SensorData` sử dụng `discriminator`.

## Issue #5: Trường ghi chú (Notes)
- Raised by: Consumer
- Concern: Một số sensor không có tính năng ghi chú, field này có thể null không?
- Proposal: Dùng union type `[string, "null"]`.
- Resolution: Accepted
- Rationale: OpenAPI 3.1 ưu tiên union type thay vì `nullable: true`.
- Impact: Cập nhật field `notes` trong `TemperatureData`.

## Issue #6: Quy tắc đặt tên field
- Raised by: Consumer
- Concern: Dùng camelCase hay snake_case?
- Proposal: Thống nhất dùng camelCase cho toàn bộ project B.
- Resolution: Accepted
- Rationale: Phù hợp với convention của Node.js/TypeScript mà nhóm đang dùng.
- Impact: Toàn bộ field trong YAML dùng camelCase.

---

# Chốt hợp đồng v1.0
Provider sign-off: Phan Lưu Phong  
Consumer sign-off: Ngô Văn Quân  
Witness (GV/TA):    
Date: 12-05-2026