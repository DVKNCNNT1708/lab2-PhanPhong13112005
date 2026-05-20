# Biên bản đàm phán hợp đồng API

- Cặp đàm phán: 05
- Product: B
- Provider: IoT Ingestion (Nhóm 8 - B1)
- Consumer: Core Business (Nhóm 11- B6)
- Phiên: v1.0
- Ngày: 20-05-2026

---

## Issue #1

- Raised by: Consumer
- Endpoint: POST `/api/v1/telemetry`
- Concern: Cấu trúc dữ liệu nhiệt độ và độ ẩm gộp chung rất khó validate từng loại riêng biệt.
- Proposal: Áp dụng cơ chế đa hình (Polymorphism) dùng `oneOf` và `discriminator` để tách riêng `TemperatureData` và `HumidityData`.
- Resolution: Accepted
- Rationale: Giúp API Contract rõ ràng hơn, tự động validate chính xác theo từng loại cảm biến.
- Impact: Provider phải sửa lại lược đồ Schema trong OpenAPI.

---

## Issue #2

- Raised by: Provider
- Endpoint: GET `/api/v1/telemetry`
- Concern: Bảng dữ liệu có hàng triệu record, nếu Consumer dùng phân trang `page/limit` sẽ gây chậm DB.
- Proposal: Đổi sang cơ chế phân trang Cursor-based (dùng `cursor` và `nextCursor`).
- Resolution: Accepted
- Rationale: Cursor-based pagination có độ trễ O(1), không bị chậm khi lùi sâu vào các trang cuối.
- Impact: Consumer phải thay đổi logic lấy danh sách, không dùng số trang nữa.

---

## Issue #3

- Raised by: Consumer
- Endpoint: ALL
- Concern: Định dạng timestamp đang gửi không đồng nhất múi giờ, có thể gây sai lệch logic cảnh báo.
- Proposal: Bắt buộc ép chuẩn RFC 3339 / ISO 8601 UTC (ví dụ: `2026-05-20T08:30:00Z`).
- Resolution: Accepted
- Rationale: Hệ thống microservices cần một chuẩn múi giờ chung là UTC để đồng bộ.
- Impact: Cả hai bên phải thêm hàm format Date sang UTC trước khi gửi/nhận.

---

## Issue #4

- Raised by: Consumer
- Endpoint: ALL
- Concern: Khi xảy ra lỗi HTTP 400 hoặc 422, thông báo lỗi bằng chuỗi text thông thường rất khó parse bằng code.
- Proposal: Áp dụng chuẩn RFC 7807 (Problem Details for HTTP APIs).
- Resolution: Accepted
- Rationale: Giúp Consumer dễ dàng lấy mã lỗi cụ thể từ trường `type` hoặc `status`.
- Impact: Provider phải định nghĩa lại Response Error schema thành `Problem`.

---

## Issue #5

- Raised by: Consumer
- Endpoint: Hàng đợi Message (Queue Async)
- Concern: Nếu mạng lỗi và Provider gửi lại (retry) thông điệp 2 lần, Consumer sẽ tính toán sai.
- Proposal: Bổ sung `eventId` dưới dạng UUID bắt buộc vào Payload để Consumer làm khóa chống trùng (Idempotency).
- Resolution: Accepted
- Rationale: Đảm bảo tính nhất quán dữ liệu (Exactly-once processing) khi dùng queue.
- Impact: Provider phải sinh UUID cho mỗi gói tin. Consumer phải check DB trước khi xử lý.

---

## Issue #6

- Raised by: Provider
- Endpoint: Hàng đợi Message (Queue Async)
- Concern: Nếu Consumer sập quá lâu, hàng đợi của Provider sẽ bị đầy bộ nhớ.
- Proposal: Gắn TTL (Time To Live) cho message là 24 giờ. Quá 24 giờ chưa xử lý sẽ đẩy vào Dead Letter Queue (DLQ).
- Resolution: Accepted
- Rationale: Tránh sập toàn bộ hệ thống môi giới tin nhắn (Message Broker).
- Impact: Consumer cần theo dõi DLQ để xử lý bù dữ liệu bị rớt.

---
Provider sign-off: Phan Lưu Phong (Đại diện Nhóm 8)
Consumer sign-off: 
Witness: (Giảng viên/Mentor)