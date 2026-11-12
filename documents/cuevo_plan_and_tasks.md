# Kế hoạch triển khai và danh sách task – Cuevo (Flutter)

Ngày: 2026-11-06 · Cập nhật: 2026-11-15 (đồng bộ SRS: D-01–D-19, E-01–E-15) · Tác giả: Luong Yen

## 1. Nguyên tắc lập kế hoạch

Kế hoạch chia công việc thành 6 giai đoạn (0–5) theo độ ưu tiên P0/P1/P2 trong SRS (cuevo_srs.md); mỗi task gắn mã yêu cầu (FR/NFR) để truy vết ngược.

- **Ước lượng theo cỡ:** S ≈ nửa ngày, M ≈ 1–2 ngày, L ≈ 3–5 ngày công của một lập trình viên Flutter. Task lớn hơn L phải tách nhỏ.
- **Chưa có mốc ngày:** cần biết số người, có lập trình native Kotlin/Swift hay không; khi có sẽ quy đổi cỡ sang lịch.
- **Định nghĩa hoàn thành (DoD):** code qua review, có test theo cấp trong mục 9 của tài liệu thiết kế, chạy đúng trên máy thật Android và iOS, không tạo cảnh báo analyzer mới.
- **Rủi ro cao làm trước:** các task đánh dấu ⚠ (spike) để kiểm chứng sớm trước khi cam kết tiến độ.

## 2. Lộ trình và cổng kiểm tra

| Giai đoạn | Mục tiêu | Cổng kiểm tra để sang giai đoạn sau |
| --- | --- | --- |
| 0. Nền tảng và spike | Dựng project, CI, kiểm chứng 5 rủi ro kỹ thuật | Spike camera native, cuộn đạt NFR-01, tiếng Việt, lưu bộ sưu tập/galleryRef, nguồn micro đều đạt; nếu camera native không khả thi thì chốt phương án gói `camera` (SRS 6.1) |
| 1. MVP – nhắc chữ | Soạn kịch bản, nhắc chữ, tùy biến hiển thị | Cuộn đạt NFR-01, không mất kịch bản khi tắt app đột ngột (NFR-05) |
| 2. MVP – quay video | Camera gốc, micro trong và ngoài, ghi hình kèm chữ, thư viện video, xin quyền, phát hành MVP | Video sạch, đồng bộ tiếng, không rớt khung, đạt NFR-13; đạt toàn bộ tiêu chí nghiệm thu MVP (SRS 7.2) và qua TestFlight/Google Play |
| 3. Hoàn thiện | Toàn bộ P1, onboarding, khả dụng, bản cập nhật | Mọi FR P1 đúng trên máy thật, không hồi quy tiêu chí nghiệm thu MVP, bản cập nhật qua TestFlight/Google Play |
| 4. Mở rộng | Cửa sổ nổi Android, điều khiển từ xa, voice-tracking, lưới canh khung (ẩn sau cờ tính năng, chưa phát hành công khai, SRS D-14); xuất/nhập JSON và máy tính bảng phát hành bình thường | Tính năng chạy đúng sau cờ; tùy phản hồi người dùng |
| 5. Bản Pro | Mua một lần, khôi phục, offline, màn Pro; chốt danh sách tính năng Pro và ra mắt cùng các tính năng giai đoạn 4 (làm cuối) | Mua, khôi phục, offline, hoàn tiền đúng với tài khoản thử; danh sách tính năng Pro đã chốt |

## 3. Task giai đoạn 0 và 1

Mỗi dòng có dạng: mã task, nội dung, (yêu cầu liên quan · cỡ · phụ thuộc). ⚠ là spike cần làm sớm.

### Giai đoạn 0: nền tảng và spike

- [ ] **T0-01** Khởi tạo project Flutter theo stack đã chốt (SRS D-19): SDK stable mới nhất khóa bằng FVM, `material_ui`, riverpod 3 (Notifier), drift, Pigeon; cấu trúc thư mục, lint, flavor dev/prod; Android minSdk 26, targetSdk 36; iOS 16 (– · M)
- [ ] **T0-02** CI: analyze, test, build debug Android và iOS (– · M · T0-01)
- [ ] **T0-03** ⚠ Spike cuộn: Ticker + ScrollController, 5.000 từ, đo fps trên máy tầm trung, cả màn 60 Hz và 120 Hz (NFR-01 · M · T0-01)
- [ ] **T0-04** ⚠ Spike camera native: preview + ghi hình bằng CameraX và AVFoundation qua Pigeon; đo độ phân giải, fps, bitrate bằng ffprobe so với hồ sơ mã hóa của máy (FR-47, NFR-13 · L · T0-01)
- [ ] **T0-05** ⚠ Spike tiếng Việt: kiểm tra hiển thị dấu và bản đậm (FR-35) ở các phông chữ dự kiến, tính phần dung lượng thêm vào NFR-12 (FR-32 · S)
- [ ] **T0-06** Theme, localization khung vi/en; ngôn ngữ lần đầu theo hệ thống (Việt/Anh), ngôn ngữ khác dùng tiếng Anh (FR-60 · S · T0-01)
- [ ] **T0-07** ⚠ Spike lưu vào bộ sưu tập: Android MediaStore (lấy URI, truy vấn lại mục do app tạo), iOS quyền chỉ thêm (`.addOnly`) và quyền đọc/giới hạn; xác nhận cách lấy và lưu `galleryRef`; mặc định lưu bằng mã native của plugin, chỉ dùng gói `gal` nếu nó trả được mã tham chiếu (SRS D-19) (FR-49a, FR-49b · M · T0-01)
- [ ] **T0-08** ⚠ Spike nguồn micro: liệt kê và chọn micro trong máy, cắm dây/USB, Bluetooth; chuyển nguồn khi ngắt giữa lúc quay. Android: CameraX nguồn MIC + `setCommunicationDevice` (TYPE_BLUETOOTH_SCO, TYPE_BLE_HEADSET), xác nhận `Recorder.Builder.setAudioSource` công khai hay phải dùng reflection. iOS: `.playAndRecord` + `.allowBluetooth`, `setPreferredInput`, thử AirPods ở chế độ chất lượng cao trên iOS 26. Thử trên Pixel, Samsung, Xiaomi và một máy Android 17. Cũng xác nhận quyền BLUETOOTH_CONNECT trên Android 12+ (xin khi chọn micro Bluetooth, D-17) và iOS không cần quyền riêng. Đạt khi: lệch hình–tiếng trong ±45 ms (NFR-16, micro trong máy và cắm dây), nhạc của ứng dụng khác phát lại được sau khi quay. Kết quả quyết định FR-41b giữ P1 hay hạ P2 (FR-41a, FR-41b, NFR-16 · M · T0-04)

### Giai đoạn 1: MVP nhắc chữ

**Kịch bản**

- [ ] **T1-01** Schema DB và repository Script (FR-01 · M · T0-01)
- [ ] **T1-02** Màn danh sách kịch bản: số từ lấy từ `PrompterDocument.wordCount` (SRS 1.3, D-16; viết bản đầu của `PrompterMarkdown.parse` ngay ở task này: bỏ ký hiệu định dạng và đếm từ, chưa dựng định dạng hiển thị), thời lượng ước tính với WPM mặc định 140 (FR-02, FR-12 · M · T1-01)
- [ ] **T1-03** Màn soạn kịch bản, tự lưu sau 1 giây ngừng gõ và khi app vào nền (FR-01, NFR-05 · M · T1-01)
- [ ] **T1-04** Tìm kiếm kịch bản (FR-03 · S · T1-02)

**Nhắc chữ**

- [ ] **T1-05** Scroll engine: WPM sang px/giây, tạm dừng, tiếp tục (FR-12, NFR-01 · L · T0-03)
- [ ] **T1-06** Màn nhắc chữ: bắt đầu/dừng, chạm kéo thủ công; thả tay giữ trạng thái trước khi kéo (đang chạy thì chạy tiếp, đang dừng thì giữ dừng) (FR-10, FR-11, FR-13 · L · T1-05)
- [ ] **T1-07** Dải đánh dấu dòng đang đọc: mặc định 1/3 trên, chỉnh 10–50%, tắt được (FR-15 · M · T1-06)
- [ ] **T1-08** Lật chữ ngang và dọc, bật/tắt độc lập (FR-19 · S · T1-06)
- [ ] **T1-09** Giữ màn hình sáng khi nhắc chữ (FR-16 · S · T1-06)
- [ ] **T1-10** Tùy biến hiển thị: cỡ chữ 20–120 dp (không nhân cỡ chữ hệ thống), dãn dòng, lề, màu, phông (FR-30, FR-31, FR-32 · M · T0-05, T1-06)
- [ ] **T1-11** Lưu cài đặt bằng shared_preferences: một bộ `PrompterSettings` dùng chung, gồm `sidePadding`, `guideEnabled`, `guidePosition` (FR-30, FR-15 · S · T1-10)
- [ ] **T1-12** Cài đặt chung: ngôn ngữ, giao diện sáng/tối (FR-60 · S · T0-06)
- [ ] **T1-13** Test: unit cho engine và đếm từ, widget cho soạn và nhắc chữ (NFR-11 · M · T1-06)

## 4. Task giai đoạn 2: quay video

**Camera gốc**

- [ ] **T2-01** Hợp đồng Pigeon cho camera (khởi tạo, preview, ghi, cài đặt, khả năng thiết bị) (FR-47 · M · T0-04)
- [ ] **T2-02** Android: CameraX preview và ghi hình, nhận cài đặt (FR-47, FR-48 · L · T2-01)
- [ ] **T2-03** iOS: AVCaptureSession preview và ghi hình, nhận cài đặt (FR-47, FR-48 · L · T2-01)
- [ ] **T2-04** Widget preview (PlatformView/Texture) cùng lớp chữ không nền nằm trên; chuyển camera trước/sau (FR-40, FR-41 · M · T2-02, T2-03, T1-06)
- [ ] **T2-05** Cài đặt camera: flash/đèn pin, bù sáng, lấy nét, zoom, lật xem trước (không ảnh hưởng video ghi ra), chống rung (FR-48 · L · T2-04)
- [ ] **T2-06** Danh sách độ phân giải và fps theo khả năng máy; mặc định 1080p/30 (không hỗ trợ thì lấy mức gần nhất thấp hơn); chọn 4K thì cảnh báo nóng máy và dung lượng (FR-42, NFR-14 · M · T2-02, T2-03)
- [ ] **T2-07** Màn quay: nút quay/dừng, đếm ngược mặc định 3 giây (chỉnh 0–10, có thể hủy) chỉ trước khi quay; đồng bộ một chiều với cuộn chữ (dừng quay thì dừng chữ, hết kịch bản vẫn quay); khóa hướng màn hình khi đang ghi; kiểm tra dung lượng trống (dưới 1 GB thì cảnh báo kèm số phút ước tính); chữ mặc định 100% và có viền (FR-14, FR-45, FR-43, UC-02 · M · T2-04)

**Lưu và thư viện video**

- [ ] **T2-08** Lưu video vào bộ nhớ riêng của ứng dụng, ghi metadata (kể cả `audioSource`) và ảnh thu nhỏ; loại trừ thư mục video khỏi sao lưu iCloud và Android Auto Backup; xóa kịch bản thì đặt `scriptId` rỗng, giữ video (FR-44, NFR-07 · M · T2-07)
- [ ] **T2-09** Màn xem lại SC-05: phát, Lưu vào bộ sưu tập, chia sẻ, xóa (FR-44 · M · T2-08)
- [ ] **T2-10** Màn danh sách video SC-09 dạng lưới, nhóm theo ngày, hiện độ dài và dung lượng tệp; dòng nhắc video chưa lưu sẽ mất khi gỡ app (FR-49, FR-44 · M · T2-08)
- [ ] **T2-11** Chọn nhiều: Lưu vào bộ sưu tập, chia sẻ, xóa hàng loạt, tiến độ (FR-49 · M · T2-10, T2-15)

**Quyền và độ bền**

- [ ] **T2-12** Luồng xin quyền camera, micro và màn lỗi SC-10 cho Android và iOS; tự kiểm tra lại quyền khi quay lại app. Hành động cửa sổ nổi trên SC-10 làm ở T4-01, không thuộc MVP (FR-61, NFR-08 · M · T2-04)
- [ ] **T2-13** Xử lý ngắt quãng: cuộc gọi đến, hết bộ nhớ, app bị đóng, giữ phần video hợp lệ (NFR-06 · M · T2-07)
- [ ] **T2-14** Đo hiệu năng và chất lượng: quay 1080p/30 (NFR-03, NFR-04, máy Android 10 trở lên), 4K/30 (NFR-14, mục tiêu), bitrate bằng ffprobe theo NFR-13, độ lệch hình–tiếng theo NFR-16 bằng video thử vỗ tay và đèn nháy (NFR-03, NFR-04, NFR-13, NFR-14, NFR-16 · M · T2-07)
- [ ] **T2-15** Hành động Lưu vào bộ sưu tập (MediaStore; thư viện ảnh iOS với quyền chỉ thêm), xin quyền lúc lưu, đặt `savedToGalleryAt` và `galleryRef`, dùng cho cả SC-05 và chọn nhiều (FR-44, FR-49 · M · T2-09, T0-07)
- [ ] **T2-16** Biểu tượng trạng thái bộ sưu tập trên SC-09 kèm nhãn đọc màn hình. Android: ba trạng thái, kiểm tra MediaStore khi mở SC-09 và khi app quay lại từ nền. iOS: chỉ chưa lưu/đã lưu. Hộp thoại giải thích và Lưu lại (FR-49, FR-49a · M · T2-15)
- [ ] **T2-17** Nguồn micro có dây: liệt kê và chọn micro trong máy, cắm dây/USB trên SC-04, mặc định micro cắm dây nếu có; rút micro khi đang quay thì chuyển sang micro trong máy, không dừng quay, có thông báo (FR-41a · M · T2-02, T2-03, T0-08)

**Phát hành MVP** (chuyển từ giai đoạn 3 theo SRS D-01)

- [ ] **T2-18** Integration test UC-01 và UC-02 trên máy thật, gồm micro cắm dây và đổi hướng màn hình khi đang ghi (NFR-09 · M · T2-14, T2-17)
- [ ] **T2-19** Tối ưu dung lượng tải về ≤ 40 MB, icon, splash, ảnh cửa hàng (NFR-12 · S)
- [ ] **T2-20** Chính sách quyền riêng tư và mô tả quyền trên cửa hàng (NFR-07, NFR-08 · S)
- [ ] **T2-21** TestFlight và Closed testing trên Google Play, sửa lỗi theo phản hồi; nghiệm thu MVP theo SRS 7.2 (L · T2-18, T2-19, T2-20)

## 5. Task giai đoạn 3 và 4

### Giai đoạn 3: hoàn thiện và phát hành

- [ ] **T3-01** Nhập kịch bản từ .txt, nhân bản, nhãn/thư mục (FR-04, FR-05 · M · T1-01)
- [ ] **T3-02** Thanh tiến độ, thời gian còn lại, nhảy tới đánh dấu (FR-17, FR-18 · M · T1-06, T3-15)
- [ ] **T3-03** Căn lề văn bản và preset cài đặt (FR-33, FR-34 · S · T1-10)
- [ ] **T3-04** Điều khiển bằng phím âm lượng và remote/tai nghe Bluetooth. Android: mọi màn nhắc chữ. iOS: phím âm lượng chỉ ở màn quay, từ iOS 17.2; các màn khác dùng remote Bluetooth dạng bàn phím/phím media (FR-50 · M · T1-06)
- [ ] **T3-05** Onboarding: 3 màn hướng dẫn và 1 màn xin quyền camera, micro, có nút Bỏ qua; bỏ qua thì xin lại đúng lúc cần (FR-62, NFR-08 · S · T2-12)
- [ ] **T3-06** Khả dụng: cỡ chữ hệ thống, tương phản, nhãn đọc màn hình; không áp dụng cho lớp chữ nhắc (NFR-10 · S)
- [ ] **T3-11** Vị trí lưu video (FR-63): Pigeon `StorageHostApi` (Android `getExternalFilesDirs`, chỉ nhận ổ thẻ nhớ đang gắn, nghe gắn/rút thẻ), cột `volume` trong `Recordings`, mục chọn ở SC-07 (ẩn trên iOS), quay vào ổ đã chọn, xử lý thẻ bị rút, cảnh báo dung lượng và thẻ chậm; hoàn thành trước T3-13 (FR-63 · M · T2-07, T2-10)
- [ ] **T3-12** iOS: trạng thái đã bị xóa khỏi bộ sưu tập. Nút Kiểm tra trạng thái trong hộp thoại giải thích, xin quyền đọc thư viện kèm lý do, quyền giới hạn hoặc bị từ chối thì không báo đã xóa (FR-49b · M · T2-16, T0-07)
- [ ] **T3-13** Hồi quy tiêu chí MVP, phát hành bản cập nhật qua TestFlight và Google Play (L · T3-01 đến T3-06, T3-11, T3-12, T3-14, T3-15)
- [ ] **T3-15** Bộ hiển thị Markdown tự viết: mở rộng `PrompterMarkdown.parse` (đã có từ T1-02) để phân tích một lượt thành văn bản sạch, đoạn định dạng và vị trí tiêu đề; dựng một `TextSpan`; dùng cho SC-03, lớp chữ SC-04, xem trước SC-02; tiêu đề chỉ in đậm, không đổi màu (D-15); unit test và đo thời gian 5.000/20.000 từ (FR-35, FR-18, NFR-15 · M · T1-02, T1-05, T1-10)
- [ ] **T3-14** Micro Bluetooth: chọn trên SC-04 (không tự chọn), xin quyền BLUETOOTH_CONNECT trên Android 12+ lúc chọn (từ chối thì tắt mục Bluetooth kèm giải thích, vẫn quay bằng micro khác, D-17), dòng nhắc về chất lượng và âm thanh phát qua tai nghe. Android: `setCommunicationDevice` khi bắt đầu quay, `clearCommunicationDevice()` khi dừng quay hoặc rời SC-04. iOS: HFP qua `setPreferredInput`, bật chế độ AirPods chất lượng cao trên iOS 26, hủy kích hoạt phiên âm thanh có báo cho ứng dụng khác khi xong. Ngắt Bluetooth khi đang quay thì chuyển micro trong máy (FR-41b · M · T2-17, T0-08)

### Giai đoạn 4: mở rộng

Các task FR-20, FR-46, FR-51, FR-52 (T4-01, T4-03, T4-04, T4-05, T4-07) làm xong nhưng ẩn sau cờ tính năng `proFeaturesEnabled`, chưa phát hành công khai trước bản Pro (SRS D-14). T4-02 và T4-06 phát hành bình thường.

- [ ] **T4-01** Cửa sổ nổi chỉ hiển thị (không nhận chạm) nhắc chữ trên ứng dụng khác, chỉ Android; thêm hành động Cấp quyền hiển thị trên ứng dụng khác và Tiếp tục với cửa sổ nổi vào SC-10 (FR-51, FR-61 · L · T2-12)
- [ ] **T4-02** Xuất/nhập sao lưu kịch bản JSON (FR-06 · S · T1-01)
- [ ] **T4-03** Lưới canh khung và chống rung nâng cao (FR-46 · M · T2-05)
- [ ] **T4-04** ⚠ Spike voice-tracking: nhận dạng giọng để cuộn theo lời đọc (FR-20 · L)
- [ ] **T4-05** Điều khiển từ thiết bị thứ hai qua Bluetooth/Wi-Fi cục bộ (FR-52 · L)
- [ ] **T4-06** Bản máy tính bảng (M)
- [ ] **T4-07** Điều khiển cửa sổ nổi qua thông báo (Tạm dừng/Tiếp tục, Tùy chỉnh, Đóng) và bảng điều khiển SC-11: vị trí, kích thước, màu chữ, màu nền, độ đậm nhạt, tốc độ; xin quyền thông báo từ Android 13 (FR-51 · L · T4-01)

### Giai đoạn 5: bản Pro (làm cuối)

- [ ] **T5-01** Tạo sản phẩm Pro không tiêu hao trên App Store Connect và Google Play Console, tài khoản thử, file cấu hình StoreKit để thử cục bộ (FR-70 · S)
- [ ] **T5-02** `ProService` dùng `in_app_purchase`: lấy giá, mua, xử lý giao dịch chờ, acknowledge trên Android, kiểm chữ ký giao dịch Android; xác nhận plugin lộ được `revocationDate` và danh sách quyền hiện có trên iOS, nếu thiếu thì thêm cầu nối native nhỏ qua Pigeon (FR-70, FR-72 · M · T5-01)
- [ ] **T5-03** Khôi phục tự động khi mở app và nút Khôi phục giao dịch mua; lưu `ProStatus` làm bộ nhớ đệm offline; khóa lại khi hoàn tiền hoặc thu hồi (FR-71, FR-73 · M · T5-02)
- [ ] **T5-04** Màn SC-12 và mục Pro trong SC-07; biểu tượng Pro trên tính năng bị khóa, mở SC-12 khi chạm (FR-74, FR-75 · M · T5-02)
- [ ] **T5-05** Chốt danh sách tính năng Pro (E-14 là đề xuất xuất phát), khóa theo danh sách đó và bật cờ các tính năng giai đoạn 4 cùng lúc ra bản Pro; cập nhật chính sách quyền riêng tư, mô tả cửa hàng; thử mua, khôi phục, offline, hoàn tiền trên máy thật (FR-75, NFR-07 · M · T5-03, T5-04)

## 6. Phụ thuộc và thứ tự ưu tiên

Đường găng (chuỗi task quyết định ngày xong) đi qua camera native, nên spike T0-04, T0-07 và T0-08 phải xong trước khi cam kết lịch giai đoạn 2.

1. T0-01 → T0-03 → T1-05 → T1-06: dựng project, spike cuộn, engine, màn nhắc chữ.
2. T0-04 → T2-01 → T2-02 và T2-03 → T2-04 → T2-07: camera native đến màn quay (hai nhánh Android và iOS chạy song song).
3. T2-07 → T2-08 → T2-10 → T2-15 → T2-11 và T2-16: lưu video, danh sách, lưu ra bộ sưu tập, chọn nhiều và biểu tượng trạng thái.
4. T2-14 → T2-18 → T2-21: đo hiệu năng và chất lượng, test tích hợp, thử nghiệm trên cửa hàng và nghiệm thu MVP.
5. T0-07 → T2-15 → T2-16 → T3-12 và T0-08 → T2-17 → T3-14: lưu bộ sưu tập, trạng thái bộ sưu tập, nguồn micro.

**Có thể làm song song:** nhóm kịch bản (T1-01 đến T1-04) với nhóm nhắc chữ (T1-05 đến T1-12); nhóm camera Android và iOS nếu có hai người biết native.

**Nếu cần cắt phạm vi:** bỏ T2-11 (chọn nhiều) và T2-05 phần chống rung trước; không cắt T2-12 (xin quyền), T2-13 (ngắt quãng), T2-14 (đo chất lượng) và T2-17 (nguồn micro, theo SRS D-09) vì đây là chỗ dễ gây mất video hoặc lỗi phát hành.
