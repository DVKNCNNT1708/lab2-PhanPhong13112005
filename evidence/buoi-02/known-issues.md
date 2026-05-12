# Known Issues — Lab 02

Ghi các lỗi còn tồn tại nếu chưa xử lý xong.

| Lỗi | Ảnh hưởng | Cách xử lý dự kiến | Người phụ trách |
|---|---|---|---|
| Chưa cấu hình webhook | Thiếu tính năng push notification ngược lại thiết bị | Bổ sung phần `webhooks` trong cấu trúc OpenAPI ở bài tập về nhà Lab 03. | Phan Lưu Phong |
| Chưa có xác thực bảo mật | Các endpoint hiện tại đang mở (public), dễ bị spam data | Sẽ bổ sung khối `securitySchemes` (Bearer Token) vào `openapi.yaml`. | Trần Văn Phong |
