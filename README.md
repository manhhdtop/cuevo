# Cuevo

Ứng dụng máy nhắc chữ (teleprompter) trên điện thoại Android và iOS, viết bằng Flutter. Người dùng soạn kịch bản, chữ tự cuộn theo tốc độ đã chọn, và có thể quay video bằng camera gốc của hệ thống trong lúc đọc. Mọi dữ liệu lưu trên máy, không cần máy chủ.

## Tài liệu

| File | Nội dung | Đọc khi |
| --- | --- | --- |
| [cuevo_srs.md](documents/cuevo_srs.md) | Đặc tả yêu cầu: chức năng (FR), màn hình (SC), phi chức năng (NFR), rủi ro, lộ trình | Cần biết ứng dụng phải làm gì |
| [cuevo_plan_and_tasks.md](documents/cuevo_plan_and_tasks.md) | Kế hoạch 6 giai đoạn (0–5) và danh sách task (T0-01 đến T5-05), phụ thuộc, đường găng | Cần biết làm gì trước, làm gì sau |
| [cuevo_technical_design.md](documents/cuevo_technical_design.md) | Thiết kế để code: kiến trúc, schema, provider, scroll engine, cầu nối Pigeon, quyền, cửa sổ nổi, mở khóa Pro, CI | Bắt tay vào lập trình |

Thứ tự đọc gợi ý: SRS → kế hoạch → thiết kế. Mã yêu cầu (FR/NFR/SC) nối ba tài liệu với nhau, mỗi task trong kế hoạch ghi rõ yêu cầu liên quan.

## Tóm tắt sản phẩm

- **Nhắc chữ:** cuộn tự động theo WPM (mặc định 140), định dạng Markdown cơ bản tự viết (tiêu đề, đậm, nghiêng, danh sách), tạm dừng/tiếp tục, kéo tay, lật chữ, dải đánh dấu dòng, tùy biến cỡ chữ, màu, phông.
- **Quay video:** dùng đường ghi hình gốc (CameraX, AVFoundation), Flutter chỉ truyền cài đặt như sáng, đèn, zoom, lật xem trước, nguồn micro; mặc định 1080p/30, không mã hóa lại; micro trong máy, cắm dây (MVP) hoặc Bluetooth (giai đoạn 3, chất lượng thoại); chữ nhắc nằm trên preview, không vào video. Đếm ngược mặc định 3 giây, chỉ áp dụng khi quay.
- **Thư viện video:** lưới theo ngày, chọn nhiều, Lưu vào bộ sưu tập của máy, biểu tượng trạng thái lưu. Video chỉ nằm trong ứng dụng cho đến khi người dùng lưu.
- **Vị trí lưu:** bộ nhớ trong hoặc thẻ nhớ (Android, khi có thẻ).
- **Quyền:** màn giới thiệu lần đầu có 1 màn xin quyền camera và micro; thiếu camera hoặc micro thì hiện màn lỗi trên cả hai nền tảng; quyền Bluetooth chỉ xin khi chọn micro Bluetooth; từ giai đoạn 4, Android có thêm hành động cấp quyền hiển thị trên ứng dụng khác.
- **Cửa sổ nổi (chỉ Android, giai đoạn 4):** chỉ hiển thị, mọi điều khiển đi qua thông báo.

## Công nghệ

Android 8.0+ và iOS 16+. Flutter stable mới nhất khi khởi tạo (3.50 ngày 2026-11-15), khóa bằng FVM; `material_ui`, flutter_riverpod 3 (Notifier), go_router, drift (SQLite), shared_preferences, freezed, plugin camera và lưu thư viện viết native qua Pigeon, in_app_purchase (bản Pro, giai đoạn 5). Chi tiết ở SRS 6.1, quyết định D-19.

## Lộ trình

| Giai đoạn | Nội dung |
| --- | --- |
| 0 | Dựng project, CI, spike cuộn, camera native, tiếng Việt, lưu bộ sưu tập, nguồn micro |
| 1 | MVP – nhắc chữ |
| 2 | MVP – quay video, thư viện, xin quyền, phát hành MVP |
| 3 | Hoàn thiện P1, vị trí lưu video, onboarding (kèm màn xin quyền), trạng thái bộ sưu tập trên iOS, bản cập nhật |
| 4 | Cửa sổ nổi Android, điều khiển từ xa, voice-tracking, lưới canh khung (ẩn sau cờ tính năng đến khi ra bản Pro) |
| 5 | Bản Pro mua một lần qua App Store và Google Play, không server; ra mắt cùng các tính năng giai đoạn 4 |

## Việc còn mở

- Chốt giá bán và danh sách tính năng bản Pro (mua một lần, SRS D-13; E-14 là đề xuất xuất phát, tính năng giai đoạn 4 ẩn đến khi chốt, D-14). Không làm đồng bộ đám mây (SRS D-12).
- Chưa có mốc ngày; cần biết số người và có lập trình viên native Kotlin/Swift hay không.

Chi tiết câu hỏi mở nằm ở mục 7.5 của SRS.
