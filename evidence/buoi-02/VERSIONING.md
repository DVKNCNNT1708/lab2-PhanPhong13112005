# Chính sách Versioning API — Smart Campus

## 1. Cơ chế Versioning
Hệ thống IoT Ingestion (B1) sử dụng **URI Versioning**.
Phiên bản API được đặt trực tiếp trong URL.
Ví dụ: `https://api.campus.local/api/v1/telemetry`

## 2. Backward-compatible changes (Không làm tăng version)
Các thay đổi sau được xem là an toàn và API vẫn giữ ở `/v1/`:
- Thêm một endpoint mới (VD: `/api/v1/telemetry/stats`).
- Thêm một optional field mới vào Request Body hoặc Response.
- Thêm một giá trị enum mới nhưng không phá vỡ logic cũ.

## 3. Breaking changes (Tăng version lên /v2/)
Nếu có những thay đổi sau, Ingestion Service bắt buộc phải tạo version mới `/v2/`:
- Đổi tên hoặc xóa một field đang là `required`.
- Đổi kiểu dữ liệu của một field (VD: từ `integer` sang `string`).
- Đổi định dạng của URL path parameters.

## 4. Quá trình Deprecation (Loại bỏ API cũ)
Khi chuyển sang `/v2/`, phiên bản `/v1/` sẽ không bị tắt ngay.
- Đánh dấu `deprecated: true` trong file `openapi.yaml` ở các endpoint cũ.
- Trả về HTTP Header `Sunset` trong response của `/v1/` để báo ngày chính thức khai tử (ít nhất 3 tháng kể từ lúc ra `/v2/`).