# SRS – Ứng dụng Máy nhắc chữ (Teleprompter) trên di động – Flutter

Dự án: **Cuevo** · Ngày: 2026-11-06 · Cập nhật: 2026-11-15 · Tác giả: Luong Yen

## 1. Giới thiệu

Tài liệu này đặc tả yêu cầu cho **Cuevo**, ứng dụng máy nhắc chữ trên điện thoại (Android và iOS) viết bằng Flutter, giúp người dùng đọc kịch bản trôi mượt trong lúc quay video. Tên dự án, tên ứng dụng và tên hiển thị trên cửa hàng ứng dụng đều là **Cuevo**.

### 1.1 Mục đích

Làm cơ sở thống nhất giữa chủ sản phẩm, thiết kế, lập trình và kiểm thử về những gì ứng dụng phải làm, ở mức đủ để ước lượng, phát triển và nghiệm thu bản MVP.

### 1.2 Phạm vi

- **Trong phạm vi:** soạn và quản lý kịch bản; nhắc chữ cuộn tự động; quay video bằng camera kèm chữ nhắc; điều khiển tốc độ; tùy biến hiển thị; lưu trữ cục bộ.
- **Ngoài phạm vi MVP:** nhận dạng giọng nói để tự cuộn theo lời đọc (voice-tracking, FR-20) và thu phí trong ứng dụng (bản Pro, FR-70–75). Hai mục này nằm ở lộ trình giai đoạn sau (mục 7).
- **Không nằm trong lộ trình:** đồng bộ đám mây và máy chủ riêng (D-12), cộng tác nhiều người, chỉnh sửa video nâng cao.

### 1.3 Định nghĩa và viết tắt

| Thuật ngữ | Ý nghĩa |
| --- | --- |
| Teleprompter | Màn hình hiển thị chữ cuộn để người nói vừa nhìn vừa đọc |
| Kịch bản (Script) | Văn bản người dùng soạn để đọc |
| WPM | Words per minute, số từ đọc mỗi phút, đếm theo định nghĩa "Từ" bên dưới |
| Từ | Chuỗi ký tự liền nhau giữa hai khoảng trắng, có ít nhất một chữ cái hoặc chữ số. Tiếng Việt mỗi âm tiết tính là một từ ("máy nhắc chữ" = 3 từ); số như "2026" là một từ; chuỗi chỉ gồm dấu câu và ký hiệu mốc `##` không tính; đếm trên văn bản đã bỏ ký hiệu định dạng của FR-35, kể cả ở các bản chưa hiển thị định dạng, để số từ không đổi khi FR-35 ra mắt (D-16) |
| Lật chữ (Mirror) | Lật lớp chữ nhắc theo chiều ngang và/hoặc dọc, dùng khi phản chiếu qua kính nhắc chữ vật lý (FR-19); khác với Lật xem trước |
| Lật xem trước | Lật hình xem trước của camera trước như soi gương; chỉ ảnh hưởng hình trên màn hình, không ảnh hưởng video ghi ra (FR-48) |
| FR / NFR | Yêu cầu chức năng / phi chức năng |
| MVP | Bản sản phẩm khả dụng tối thiểu |

### 1.4 Tài liệu tham chiếu

- Tài liệu Flutter và Dart chính thức (flutter.dev, dart.dev)
- Tài liệu CameraX (Android), AVFoundation (Apple), Pigeon và các plugin permission_handler, wakelock_plus, in_app_purchase trên pub.dev
- Chuẩn trình bày SRS theo tinh thần ISO/IEC/IEEE 29148 (thay thế IEEE 830)

## 2. Mô tả tổng quan

Ứng dụng hoạt động hoàn toàn offline trên điện thoại: người dùng nhập kịch bản, mở chế độ nhắc chữ, chữ tự cuộn với tốc độ đã chọn, và có thể đồng thời quay video bằng camera.

### 2.1 Nhóm người dùng

| Nhóm | Nhu cầu chính |
| --- | --- |
| Người sáng tạo nội dung (TikTok, Reels, YouTube) | Quay nhanh, tự cuộn để nhìn camera liên tục |
| Giảng viên, MC, diễn giả | Chữ lớn, dễ đọc ở khoảng cách xa, kịch bản dài |
| Doanh nghiệp nhỏ, bán hàng livestream | Lưu nhiều kịch bản mẫu, điều chỉnh tốc độ khi đang quay |

### 2.2 Môi trường vận hành

- Android 8.0 (API 26) trở lên; iOS 16 trở lên.
- Hỗ trợ điện thoại, có thể mở rộng sang máy tính bảng ở giai đoạn sau.
- Ngôn ngữ giao diện: tiếng Việt và tiếng Anh. Lần đầu mở, ứng dụng theo ngôn ngữ hệ thống nếu là tiếng Việt hoặc tiếng Anh, ngôn ngữ khác thì dùng tiếng Anh; sau đó theo lựa chọn ở FR-60. Hỗ trợ đầy đủ dấu tiếng Việt (Unicode).
- Flutter stable mới nhất khi khởi tạo dự án (3.50 ở thời điểm 2026-11-15; 3.47.x là phương án lùi), Dart đi kèm bản đó; khóa SDK bằng FVM và ràng buộc `environment` trong pubspec; một mã nguồn cho cả hai nền tảng (D-19).

### 2.3 Ràng buộc

- Cuộn chữ phải mượt, kể cả khi đang quay video; ngưỡng đo ở NFR-01, NFR-03 và NFR-14.
- Quyền camera, micro và lưu trữ do hệ điều hành quản lý; ứng dụng phải xử lý cả trường hợp người dùng từ chối.
- Không có máy chủ riêng (D-12), nên mọi dữ liệu lưu trên máy; riêng việc mua Pro đi qua App Store và Google Play.

### 2.4 Giả định

- Người dùng quay dọc (portrait) là chính, quay ngang (landscape) là phụ.
- Kịch bản thông thường dưới 20.000 từ; kịch bản dài hơn vẫn phải mở được nhưng không cam kết hiệu năng tối ưu.
- Mô hình kinh doanh: ứng dụng miễn phí, có bản Pro mua một lần ở giai đoạn 5 (D-13); giá bán và danh sách tính năng Pro chốt sau.

## 3. Yêu cầu chức năng

Mỗi yêu cầu có mã duy nhất (FR-xx) và độ ưu tiên: **P0** = bắt buộc cho MVP, **P1** = nên có sớm sau MVP, **P2** = giai đoạn sau.

### 3.1 Quản lý kịch bản

| Mã | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| FR-01 | Tạo, sửa, xóa kịch bản gồm tiêu đề và nội dung văn bản | P0 |
| FR-02 | Danh sách kịch bản hiển thị tiêu đề, ngày sửa, số từ, thời lượng ước tính theo WPM | P0 |
| FR-03 | Tìm kiếm kịch bản theo tiêu đề và nội dung | P0 |
| FR-04 | Nhập kịch bản bằng cách dán văn bản hoặc nhập từ tệp .txt | P1 |
| FR-05 | Nhân bản, sắp xếp, gắn nhãn/thư mục cho kịch bản | P1 |
| FR-06 | Xuất/nhập sao lưu kịch bản dạng tệp JSON | P2 |

### 3.2 Chế độ nhắc chữ (cốt lõi)

| Mã | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| FR-10 | Hiển thị kịch bản toàn màn hình, chữ cuộn từ dưới lên trên | P0 |
| FR-11 | Bắt đầu, tạm dừng, tiếp tục bằng một chạm lên màn hình hoặc nút | P0 |
| FR-12 | Điều chỉnh tốc độ cuộn trong lúc chạy, dải 10–300 WPM, bước 5 WPM; mặc định 140 WPM (cũng dùng để tính thời lượng ước tính ở FR-02) | P0 |
| FR-13 | Người dùng vuốt/kéo để cuộn thủ công. Thả tay: nếu trước khi kéo chữ đang tự cuộn thì tự cuộn tiếp ngay từ vị trí mới; nếu đang tạm dừng thì giữ tạm dừng | P0 |
| FR-14 | Chỉ ở chế độ quay video: đếm ngược khi bấm nút quay, trước khi bắt đầu ghi hình và cuộn chữ; mặc định 3 giây, chỉnh 0–10 giây (0 là tắt); số đếm hiện lớn giữa màn hình, chạm để hủy. Nhắc chữ không quay video thì không đếm; kéo tay chữ (FR-13), tạm dừng và tiếp tục cũng không đếm | P0 |
| FR-15 | Dải đánh dấu dòng đang đọc (reading guide); mặc định ở 1/3 chiều cao màn hình tính từ trên, người dùng chỉnh vị trí trong khoảng 10–50% hoặc tắt dải ở SC-06 | P0 |
| FR-16 | Giữ màn hình luôn sáng khi đang nhắc chữ | P0 |
| FR-17 | Hiển thị thanh tiến độ và thời gian còn lại | P1 |
| FR-18 | Nhảy tới đoạn/mục bằng dấu mốc: mỗi dòng tiêu đề theo FR-35 là một mốc | P1 |
| FR-19 | Lật chữ (mirror) theo chiều ngang và chiều dọc, bật/tắt độc lập; chỉ tác động lên lớp chữ | P0 |
| FR-20 | Cuộn theo giọng nói (voice-tracking) | P2 |

### 3.3 Tùy biến hiển thị

| Mã | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| FR-30 | Chỉnh cỡ chữ nhắc 20–120 dp (không nhân theo cỡ chữ hệ thống để giữ đúng dải), khoảng cách dòng, lề hai bên; cỡ chữ hệ thống vẫn áp dụng cho phần giao diện còn lại (NFR-10) | P0 |
| FR-31 | Chọn màu chữ và màu nền (mặc định chữ trắng trên nền đen); ở chế độ quay chữ hiển thị không có nền | P0 |
| FR-32 | Chọn phông chữ từ danh sách có sẵn, đảm bảo hiển thị đúng dấu tiếng Việt; mỗi phông có bản đậm để dùng cho FR-35 | P0 |
| FR-33 | Căn lề văn bản: trái, giữa, phải | P1 |
| FR-34 | Lưu các bộ cài đặt thành preset và áp dụng lại nhanh | P1 |
| FR-35 | Hiển thị định dạng cơ bản trong kịch bản bằng bộ hiển thị Markdown tự viết, ưu tiên nhanh và nhẹ. Chỉ hỗ trợ: dòng bắt đầu bằng `#`, `##`, `###` là tiêu đề (ẩn ký hiệu, chỉ in đậm, không đổi màu và không đổi cỡ chữ để tốc độ cuộn đều; D-15); `**đậm**`; `*nghiêng*` hoặc `_nghiêng_`; dòng bắt đầu bằng `- `, `* ` hoặc `1. ` là danh sách, hiện dấu đầu dòng; `\` để gõ ký hiệu như chữ thường. Xuống dòng giữ nguyên như người dùng gõ, dòng trống tách đoạn. Cú pháp khác (liên kết, ảnh, bảng, khối code, HTML) hiện nguyên như chữ thường. Không dùng thư viện hiển thị Markdown bên ngoài: kịch bản được chuyển thành một khối chữ có định dạng duy nhất, nên bộ cuộn đo, lật chữ và vẽ viền như với chữ thường. Áp dụng cho SC-03, lớp chữ trên SC-04 và phần xem trước ở SC-02 | P1 |

### 3.4 Quay video kèm nhắc chữ

| Mã | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| FR-40 | Xem trước camera, chuyển camera trước/sau | P0 |
| FR-41 | Quay video kèm tiếng, chữ nhắc hiển thị trực tiếp trên khung camera, chữ không có nền (không khung, không mảng màu phía sau); chữ nhắc **không** bị ghi vào video | P0 |
| FR-41a | Nguồn micro có dây: micro trong máy và micro cắm dây (jack 3.5 mm, USB-C, Lightning). Mặc định dùng micro cắm dây nếu đang cắm, nếu không thì dùng micro trong máy; người dùng chọn nguồn trên SC-04, danh sách theo các nguồn hệ điều hành báo về. Micro cắm dây bị rút khi đang quay thì chuyển sang micro trong máy, không dừng quay, có thông báo | P0 |
| FR-41b | Micro Bluetooth, hỗ trợ trong khả năng của thiết bị. Tai nghe Bluetooth thông thường (HFP) cho chất lượng thoại, lấy mẫu 8–16 kHz, có thể lệch tiếng với hình, và trong lúc quay mọi âm thanh của máy phát qua tai nghe. Chất lượng cao hơn khi dùng AirPods trên iOS 26 trở lên (chế độ ghi âm chất lượng cao) hoặc tai nghe LE Audio trên Android 13 trở lên. Micro Bluetooth không bao giờ được tự chọn: người dùng phải chọn trên SC-04, khi đó SC-04 hiện dòng nhắc về chất lượng tiếng và việc âm thanh phát qua tai nghe. Android 12 trở lên cần quyền BLUETOOTH_CONNECT: chỉ xin khi người dùng chọn micro Bluetooth, kèm giải thích; bị từ chối thì mục Bluetooth tắt kèm lời giải thích và nút mở Cài đặt, việc quay vẫn dùng micro khác và không hiện SC-10 (D-17). iOS không cần quyền riêng cho đường âm thanh này (xác nhận ở T0-08). Ngắt Bluetooth khi đang quay thì chuyển sang micro trong máy, không dừng quay, có thông báo. Dừng quay hoặc rời SC-04 thì trả lại định tuyến âm thanh như trước để ứng dụng khác phát tiếp bình thường | P1 |
| FR-42 | Chọn độ phân giải và tốc độ khung hình theo danh sách thiết bị báo về (ví dụ 1080p, 4K, 30/60 fps); mặc định 1080p/30 fps, nếu máy không hỗ trợ thì lấy mức gần nhất thấp hơn; các mức cao hơn chỉ hiện khi máy báo hỗ trợ | P0 |
| FR-43 | Đặt độ đậm nhạt của chữ nhắc (10–100%), kèm tùy chọn viền hoặc bóng chữ để dễ đọc khi không có nền; mặc định 100% và bật viền chữ | P0 |
| FR-44 | Video quay xong chỉ lưu trong bộ nhớ riêng của ứng dụng, không tự vào thư viện ảnh của máy; xem lại, chia sẻ, và có hành động Lưu vào bộ sưu tập của máy; khi còn video chưa lưu, SC-09 nhắc rằng gỡ ứng dụng hoặc xóa dữ liệu ứng dụng sẽ mất các video này | P0 |
| FR-45 | Đồng bộ cuộn chữ với ghi hình: hết đếm ngược thì ghi hình và cuộn chữ bắt đầu cùng lúc; dừng quay thì cuộn chữ dừng. Chiều ngược lại không áp dụng: tạm dừng cuộn chữ, kéo tay chữ (FR-13) hay hết kịch bản đều không dừng ghi hình; hết kịch bản thì chữ đứng ở cuối và vẫn quay đến khi người dùng bấm dừng | P0 |
| FR-46 | Lưới canh khung, chống rung nếu máy hỗ trợ | P2 |
| FR-47 | Camera dùng đường ghi hình gốc của hệ điều hành (CameraX trên Android, AVFoundation trên iOS); Flutter không xử lý hay mã hóa lại video. Thông số ghi (độ phân giải, fps, bitrate, codec) lấy theo hồ sơ mã hóa máy báo về (EncoderProfiles/CamcorderProfile trên Android, cấu hình đề xuất của AVFoundation trên iOS). Không cam kết hình ảnh giống ứng dụng camera của hãng máy, vì xử lý riêng của hãng (HDR, xử lý ảnh, chống rung nâng cao) có thể không mở cho ứng dụng bên thứ ba | P0 |
| FR-48 | Lớp bao (wrapper) chỉ điều khiển cài đặt camera: đèn flash/đèn pin, bù sáng (phơi sáng), lấy nét, zoom, lật xem trước của camera trước (không ảnh hưởng video ghi ra), chống rung; không chèn bước xử lý ảnh nào | P0 |
| FR-49 | Danh sách video đã quay dạng lưới ảnh thu nhỏ giống bộ sưu tập của hệ thống, nhóm theo ngày quay (mới nhất trên cùng); mỗi video chỉ hiển thị độ dài và dung lượng tệp; chạm để mở SC-05; nhấn giữ để chọn nhiều video, rồi Lưu vào bộ sưu tập của máy (thư viện ảnh/video), chia sẻ hoặc xóa hàng loạt, có nút chọn tất cả và báo tiến độ khi lưu; video chỉ nằm trong ứng dụng cho đến khi người dùng lưu, xóa trong ứng dụng không xóa bản đã lưu ra bộ sưu tập. Mỗi ô video có biểu tượng trạng thái bộ sưu tập: chưa lưu, đã lưu, hoặc bản trong bộ sưu tập đã bị xóa (trạng thái thứ ba theo FR-49a, FR-49b); chạm vào biểu tượng để xem giải thích và lưu (lại). Biểu tượng có nhãn đọc màn hình, không chỉ dựa vào màu | P0 |
| FR-49a | Android: phát hiện bản đã lưu bị xóa khỏi bộ sưu tập bằng cách kiểm tra mục do ứng dụng tạo trong MediaStore (theo galleryRef), không xin thêm quyền; kiểm tra khi mở SC-09 và khi ứng dụng quay lại từ nền. iOS trong MVP chỉ hiện hai trạng thái chưa lưu/đã lưu. Cách lấy và lưu galleryRef trên hai nền tảng xác nhận ở giai đoạn 0 | P0 |
| FR-49b | iOS: phát hiện bản đã lưu bị xóa. Nếu quyền đọc thư viện đã được cấp, hoặc người dùng chọn Kiểm tra trạng thái trong phần giải thích của biểu tượng (lúc đó mới xin quyền đọc, kèm lý do), thì hiện thêm trạng thái đã bị xóa. Khi không xác định được (bị từ chối quyền đọc, hoặc quyền giới hạn không thấy video) thì không báo là đã xóa. Lưu (lại) luôn dùng được, không phụ thuộc quyền đọc | P1 |

### 3.5 Điều khiển từ xa và phụ trợ

| Mã | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| FR-50 | Điều khiển bằng phím âm lượng hoặc remote/tai nghe Bluetooth (bắt đầu/dừng, tăng/giảm tốc). Android: dùng được ở mọi màn nhắc chữ. iOS không có API công khai để bắt phím âm lượng nói chung: chỉ dùng được ở màn quay video từ iOS 17.2 qua API sự kiện nút chụp của hệ thống (xác nhận khi làm); các màn khác chỉ nhận remote Bluetooth dạng bàn phím hoặc phím media | P1 |
| FR-51 | Chỉ Android: chế độ cửa sổ nổi nhắc chữ trên ứng dụng khác (cần quyền Hiển thị trên ứng dụng khác và quyền thông báo từ Android 13). Cửa sổ chỉ để hiển thị: không nhận chạm, không có nút hay kéo thả, để người dùng thao tác trực tiếp trên ứng dụng camera bên dưới. Mọi điều khiển (tạm dừng/tiếp tục, tốc độ, vị trí, kích thước, màu chữ, màu nền, đóng) đi qua thông báo của ứng dụng, mở thêm bảng điều khiển SC-11 khi cần chỉnh chi tiết. Nếu người dùng không cấp quyền, hiện cảnh báo nhưng vẫn cho tiếp tục với chế độ nhắc chữ trong ứng dụng. Chế độ này dùng được cả khi camera của ứng dụng bị từ chối, vì việc quay do ứng dụng camera khác đảm nhiệm, video không nằm trong danh sách video của ứng dụng. Từ giai đoạn này, SC-10 trên Android có thêm hành động Cấp quyền hiển thị trên ứng dụng khác (mở thẳng trang quyền này); nếu camera bị từ chối nhưng quyền này đã được cấp, vẫn cho tiếp tục ở chế độ cửa sổ nổi để nhắc chữ trên ứng dụng camera khác. iOS không hỗ trợ chế độ này | P2 |
| FR-52 | Điều khiển từ thiết bị thứ hai qua Bluetooth/Wi-Fi cục bộ | P2 |

### 3.6 Cài đặt chung

| Mã | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| FR-60 | Chọn ngôn ngữ Việt/Anh, giao diện sáng/tối | P0 |
| FR-61 | Xin quyền camera, micro khi cần dùng (quyền lưu vào thư viện chỉ xin khi người dùng chọn Lưu vào bộ sưu tập), kèm giải thích lý do; nếu bị từ chối, hiện màn hình lỗi (SC-10) trên cả Android và iOS, nêu rõ quyền nào còn thiếu, có nút mở Cài đặt hệ thống; quay lại ứng dụng thì tự kiểm tra lại quyền. Quyền đọc thư viện trên iOS chỉ xin khi người dùng chọn Kiểm tra trạng thái (FR-49b, sau MVP). Quyền Bluetooth (Android 12 trở lên) chỉ xin khi người dùng chọn micro Bluetooth (FR-41b) và không chặn việc quay. Các hành động liên quan cửa sổ nổi trên SC-10 (chỉ Android) thuộc FR-51, làm ở giai đoạn 4, không nằm trong MVP | P0 |
| FR-62 | Trợ giúp lần đầu (onboarding) gồm 3 màn hướng dẫn và 1 màn xin quyền camera, micro (D-18): màn này giải thích lý do rồi mới xin, có nút Bỏ qua; bỏ qua thì quyền được xin lại đúng lúc cần dùng (NFR-08). Quyền lưu thư viện, đọc thư viện và Bluetooth không xin ở đây | P1 |
| FR-63 | Cài đặt Vị trí lưu video: Bộ nhớ trong (mặc định) hoặc Thẻ nhớ. Tùy chọn Thẻ nhớ chỉ hiện trên Android khi máy có thẻ nhớ đang gắn; iOS không có thẻ nhớ nên ẩn mục này. Mỗi lựa chọn hiện dung lượng còn trống. Áp dụng cho video quay mới, video cũ giữ nguyên chỗ. Nếu thẻ bị rút, video quay mới tự dùng bộ nhớ trong kèm thông báo; video đang nằm trên thẻ hiện trạng thái Thẻ nhớ chưa gắn (không bị xóa khỏi danh sách) và dùng lại được khi gắn thẻ | P1 |

### 3.7 Luồng xử lý chính (use case)

**UC-01 – Đọc kịch bản với nhắc chữ:**

1. Người dùng chọn kịch bản trong danh sách.
2. Ứng dụng mở màn nhắc chữ (SC-03) ở trạng thái chờ, chữ dừng ở đầu kịch bản, áp dụng tùy biến đã lưu.
3. Người dùng nhấn Bắt đầu; ứng dụng cuộn chữ ngay, không đếm ngược.
4. Người dùng có thể tạm dừng, đổi tốc độ, kéo lại đoạn đã đọc; bấm Tiếp tục thì cuộn tiếp ngay.
5. Đến cuối kịch bản, ứng dụng tự dừng và hiện nút Đọc lại.

**UC-02 – Quay video với nhắc chữ:**

1. Người dùng mở chế độ quay từ kịch bản; ứng dụng xin quyền camera và micro nếu chưa có.
2. Người dùng chọn camera, độ phân giải và fps, độ sáng, đèn, lật xem trước, lật chữ, nguồn micro (FR-41a, FR-41b).
3. Nhấn nút quay; ứng dụng kiểm tra dung lượng trống, đếm ngược (FR-14), rồi bắt đầu ghi hình và cuộn chữ cùng lúc (FR-45). Hướng màn hình khóa theo hướng lúc bắt đầu ghi.
4. Nếu hết kịch bản trước khi dừng quay: chữ đứng ở cuối, vẫn tiếp tục ghi (FR-45).
5. Nhấn dừng; video lưu trong bộ nhớ riêng của ứng dụng, ứng dụng mở màn hình xem lại.
6. Ngoại lệ: dung lượng trống dưới 1 GB khi bấm quay → cảnh báo kèm ước tính số phút quay được ở độ phân giải đang chọn, người dùng vẫn có thể quay. Hết bộ nhớ khi đang quay → dừng quay, giữ phần đã ghi, báo lỗi rõ ràng.
7. Ngoại lệ: người dùng từ chối camera hoặc micro → hiện SC-10, không vào màn quay; không có chế độ quay không tiếng (D-10). Riêng Android, từ giai đoạn 4 (FR-51, ngoài MVP): SC-10 có hành động Cấp quyền hiển thị trên ứng dụng khác; nếu quyền này đã cấp thì vẫn cho tiếp tục ở chế độ cửa sổ nổi trên ứng dụng camera khác dù camera của ứng dụng bị chặn; nếu quyền này cũng bị từ chối thì cảnh báo rồi vẫn cho tiếp tục nhắc chữ trong ứng dụng.

### 3.8 Bản Pro (mua một lần, không server)

Mở khóa Pro theo tài khoản cửa hàng (Apple ID trên iOS, tài khoản Google trên Android), chỉ dùng App Store và Google Play, không có máy chủ riêng và không dùng dịch vụ quản lý mua hàng bên thứ ba (D-13). Mua trên một nền tảng không mở khóa trên nền tảng kia. Phần này làm ở giai đoạn cuối (giai đoạn 5).

| Mã | Yêu cầu | Ưu tiên |
| --- | --- | --- |
| FR-70 | Mua bản Pro một lần, trọn đời (sản phẩm không tiêu hao, non-consumable) qua App Store và Google Play bằng plugin `in_app_purchase`. Giá và đơn vị tiền tệ lấy từ cửa hàng, không ghi cứng. Giao dịch đang chờ (Ask to Buy trên iOS, thanh toán chờ xử lý trên Android) hiện trạng thái chờ, chưa mở khóa | P2 |
| FR-71 | Khôi phục: khi mở ứng dụng, tự đọc quyền đã mua từ cửa hàng (iOS `Transaction.currentEntitlements`, Android `queryPurchasesAsync`) để mở khóa trên máy mới hoặc sau khi cài lại, không cần thao tác. Có nút Khôi phục giao dịch mua trên SC-12 và SC-07 (iOS gọi `AppStore.sync()`) | P2 |
| FR-72 | Xác minh giao dịch trên máy: iOS dùng giao dịch đã ký do StoreKit 2 xác minh; Android kiểm chữ ký giao dịch bằng khóa công khai của ứng dụng trên Google Play và xác nhận (acknowledge) giao dịch ngay sau khi mua để Google không tự hoàn tiền. Không xác minh qua server (chấp nhận rủi ro ở mục 7.3) | P2 |
| FR-73 | Dùng được khi offline: trạng thái Pro lấy từ bộ nhớ đệm giao dịch của cửa hàng trên máy, kèm bản lưu trạng thái lần xác minh gần nhất; khi có mạng thì kiểm lại. Giao dịch bị hoàn tiền hoặc thu hồi (iOS có `revocationDate`, Android không còn trả về giao dịch) thì khóa lại tính năng Pro, không xóa dữ liệu người dùng đã tạo | P2 |
| FR-74 | Màn Pro (SC-12): danh sách tính năng Pro, giá theo cửa hàng, nút Mua, nút Khôi phục giao dịch mua, liên kết Điều khoản sử dụng và Chính sách quyền riêng tư. Mở từ SC-07 và khi người dùng chạm vào tính năng Pro chưa mở khóa | P2 |
| FR-75 | Danh sách tính năng Pro còn mở, chốt trước khi bắt đầu giai đoạn 5 (E-14 là đề xuất xuất phát). Quy tắc: tính năng đã phát hành miễn phí (MVP, giai đoạn 3 và mọi bản đã ra) giữ miễn phí cho mọi người dùng; các tính năng giai đoạn 4 thuộc danh sách ứng viên Pro (FR-20, FR-46, FR-51, FR-52) được làm xong nhưng ẩn sau cờ tính năng và không phát hành công khai trước bản Pro (D-14). Tính năng Pro chưa mở khóa vẫn hiện trong giao diện, có biểu tượng Pro và nhãn đọc màn hình | P2 |

## 4. Yêu cầu giao diện

Giao diện ưu tiên thao tác một tay, nút lớn nằm vùng ngón cái với tới, và dùng Material 3 từ gói `material_ui` làm nền (D-19).

### 4.1 Danh sách màn hình

| Mã | Màn hình | Nội dung chính |
| --- | --- | --- |
| SC-01 | Danh sách kịch bản (trang chủ) | Danh sách, ô tìm kiếm, nút tạo mới |
| SC-02 | Soạn kịch bản | Ô nhập tiêu đề, văn bản, đếm từ, thời lượng ước tính, nút xem trước định dạng (FR-35) |
| SC-03 | Nhắc chữ | Chữ cuộn toàn màn hình, thanh điều khiển ẩn/hiện khi chạm |
| SC-04 | Quay video | Xem trước camera, chữ không nền, nút quay/dừng, số đếm ngược, thanh cài đặt camera (độ phân giải và fps, sáng, đèn, lật xem trước, lật chữ), chọn nguồn micro (FR-41a, FR-41b), dòng nhắc khi chọn micro Bluetooth |
| SC-05 | Xem lại video | Phát lại, Lưu vào bộ sưu tập, chia sẻ, xóa |
| SC-06 | Tùy biến hiển thị | Cỡ chữ, màu, phông, tốc độ, lật chữ, vị trí dải đánh dấu dòng, preset, số giây đếm ngược khi quay |
| SC-07 | Cài đặt | Ngôn ngữ, giao diện, vị trí lưu video (Android có thẻ nhớ, FR-63), quyền, Pro và Khôi phục giao dịch mua (FR-71, FR-74), thông tin ứng dụng |
| SC-08 | Giới thiệu lần đầu | 3 màn hình hướng dẫn và 1 màn xin quyền camera, micro (có nút Bỏ qua) |
| SC-09 | Danh sách video đã quay | Lưới ảnh thu nhỏ nhóm theo ngày, mỗi ô ghi độ dài, dung lượng tệp và biểu tượng trạng thái bộ sưu tập; khi còn video chưa lưu có dòng nhắc video sẽ mất nếu gỡ ứng dụng; mở từ thanh điều hướng của trang chủ, chạm một video để sang SC-05; nhấn giữ để vào chế độ chọn nhiều, thanh hành động gồm Lưu vào bộ sưu tập, Chia sẻ, Xóa |
| SC-10 | Lỗi thiếu quyền | Hiện khi người dùng từ chối camera hoặc micro: nêu quyền thiếu và lý do cần, nút mở Cài đặt, nút quay lại; có bản Android và iOS, theo ngôn ngữ đang chọn của ứng dụng. Từ giai đoạn 4 (FR-51, chỉ Android): thêm trường hợp thiếu quyền thông báo (Android 13 trở lên) và thêm nút Cấp quyền hiển thị trên ứng dụng khác; nếu quyền này đã cấp thì hiện nút Tiếp tục với cửa sổ nổi dù camera bị chặn; nếu quyền này bị từ chối thì chỉ là cảnh báo có nút Tiếp tục |
| SC-11 | Bảng điều khiển cửa sổ nổi (Android) | Mở từ action Tùy chỉnh trên thông báo, dạng bảng gọn trên nền trong suốt: vị trí, kích thước, màu chữ, màu nền, độ đậm nhạt, tốc độ; đóng bảng thì quay về ứng dụng camera |
| SC-12 | Bản Pro | Danh sách tính năng Pro, giá theo cửa hàng, nút Mua, Khôi phục giao dịch mua, trạng thái chờ thanh toán, liên kết điều khoản và quyền riêng tư (FR-70–FR-75, giai đoạn 5) |

### 4.2 Quy ước giao diện

- Trong màn hình nhắc chữ, thanh điều khiển tự ẩn sau 3 giây không chạm, chạm một lần để hiện lại.
- Vùng chạm tối thiểu 48×48 dp cho mọi nút.
- Hỗ trợ xoay màn hình dọc/ngang; chữ nhắc dàn lại mà không mất vị trí đang đọc. Khi đang ghi hình, hướng màn hình khóa theo hướng lúc bắt đầu quay để video không đổi hướng giữa chừng.
- Giao diện sáng và tối; màn nhắc chữ mặc định nền đen để giảm chói.
- Mọi thông báo lỗi dùng ngôn ngữ đơn giản kèm hành động tiếp theo (ví dụ "Cấp quyền camera").

## 5. Yêu cầu phi chức năng

| Mã | Nhóm | Yêu cầu đo được |
| --- | --- | --- |
| NFR-01 | Hiệu năng | Cuộn đạt ≥ 58 fps trung bình trên màn 60 Hz, và ≥ 95% tần số quét trên màn tần số cao (ví dụ ≥ 114 fps ở 120 Hz); đo trên máy tầm trung (RAM 4 GB), kịch bản 5.000 từ |
| NFR-02 | Hiệu năng | Khởi động nguội ≤ 2 giây; mở kịch bản 5.000 từ ≤ 500 ms |
| NFR-03 | Hiệu năng | Khi quay 1080p/30 fps (mặc định), không rớt quá 1% khung hình do lớp nhắc chữ |
| NFR-04 | Tài nguyên | Quay 10 phút liên tục ở 1080p/30 fps, độ sáng màn hình 50%, trên máy chuẩn: trạng thái nhiệt của hệ điều hành không vượt mức trung bình (Android thermal status ≤ MODERATE, iOS thermalState ≤ fair); pin giảm không quá 10%. Máy Android dùng để đo phải chạy Android 10 trở lên, vì API trạng thái nhiệt chỉ có từ API 29 |
| NFR-05 | Độ tin cậy | Kịch bản tự lưu sau 1 giây ngừng gõ và ngay khi ứng dụng chuyển sang nền; ứng dụng bị đóng đột ngột thì mất tối đa phần gõ trong 1 giây cuối, không bao giờ hỏng kịch bản đã lưu |
| NFR-06 | Độ tin cậy | Cuộc gọi đến hoặc ngắt ứng dụng khi đang quay: lưu phần video đã ghi hợp lệ |
| NFR-07 | Bảo mật/riêng tư | Dữ liệu kịch bản và video chỉ lưu trên máy; không gửi đi ngoài nếu không có hành động của người dùng. Thư mục video được loại trừ khỏi sao lưu hệ thống (iCloud trên iOS, Auto Backup trên Android); kịch bản và cài đặt theo cài đặt sao lưu hệ thống của người dùng. Mua Pro chỉ giao tiếp với App Store và Google Play, không gửi dữ liệu cho bên thứ ba |
| NFR-08 | Bảo mật/riêng tư | Chỉ xin quyền khi cần dùng (just-in-time), có giải thích lý do trước khi xin. Ngoại lệ có chủ đích: màn xin quyền camera và micro trong giới thiệu lần đầu (FR-62, D-18), người dùng có thể bỏ qua |
| NFR-09 | Tương thích | Android 8.0+ và iOS 16+; chạy đúng trên màn hình từ 5" đến 7" |
| NFR-10 | Khả dụng | Hỗ trợ cỡ chữ hệ thống, độ tương phản màu ≥ 4.5:1 ở giao diện mặc định, nhãn cho trình đọc màn hình. Không áp dụng cho lớp chữ nhắc trên hình camera (FR-43) và cỡ chữ nhắc (FR-30) |
| NFR-11 | Bảo trì | Chia lớp rõ ràng, độ phủ kiểm thử đơn vị ≥ 70% cho logic nghiệp vụ |
| NFR-12 | Phát hành | Dung lượng tải về ≤ 40 MB cho một máy cụ thể (theo báo cáo kích thước của Google Play Console và App Store Connect), tính cả phông chữ đóng gói; đạt yêu cầu của Google Play và App Store |
| NFR-13 | Chất lượng video | Video ghi ra đúng độ phân giải và fps đã chọn. Bitrate mục tiêu lấy từ hồ sơ mã hóa máy báo về (Android: EncoderProfiles/CamcorderProfile; iOS: recommendedVideoSettings của AVFoundation); bitrate trung bình đo bằng ffprobe nằm trong khoảng ±10% mục tiêu, với điều kiện đo: quay 60 giây cảnh nhiều chuyển động, đủ sáng. Không mã hóa lại sau khi ghi |
| NFR-14 | Hiệu năng | Khi quay 4K/30 fps trên máy báo hỗ trợ: cuộn chữ ≥ 55 fps trung bình, không rớt quá 1% khung hình do lớp nhắc chữ, quay 5 phút liên tục chưa bị hệ điều hành giảm xung; 4K/60 fps không cam kết hiệu năng. Đây là mục tiêu, không chặn nghiệm thu MVP: máy báo hỗ trợ 4K vẫn cho chọn, kèm cảnh báo về nóng máy và dung lượng |
| NFR-15 | Hiệu năng | Bộ hiển thị định dạng (FR-35) chuyển kịch bản 5.000 từ thành khối chữ có định dạng trong ≤ 50 ms và 20.000 từ trong ≤ 200 ms trên máy tầm trung; thời gian này nằm trong ngưỡng mở kịch bản của NFR-02; cuộn chữ có định dạng vẫn đạt NFR-01; không thêm thư viện ngoài |
| NFR-16 | Chất lượng video | Độ lệch giữa hình và tiếng trong video ghi ra ≤ ±45 ms với micro trong máy và micro cắm dây, đo bằng video thử có tiếng vỗ tay kèm đèn nháy (so khung hình đèn sáng với dạng sóng tiếng vỗ) trên máy chuẩn; micro Bluetooth không cam kết (FR-41b) |

Các con số trên là mục tiêu đề xuất; cần đo thực tế trên thiết bị chuẩn khi đội kiểm thử chốt.

## 6. Kiến trúc kỹ thuật Flutter

Ứng dụng chia ba lớp: giao diện (widget), nghiệp vụ (state + service) và dữ liệu (local), theo hướng feature-first để mỗi tính năng độc lập, dễ kiểm thử.

```text
Giao diện (widget)  ->  Nghiệp vụ (state + service)  ->  Dữ liệu (repository, camera, hệ điều hành)
```

Widget chỉ gọi lớp nghiệp vụ; lớp này đọc và ghi dữ liệu qua repository, camera và hệ điều hành, không để widget chạm trực tiếp vào chúng.

### 6.1 Công nghệ và package đề xuất

| Nhu cầu | Package | Ghi chú |
| --- | --- | --- |
| Quản lý trạng thái | flutter_riverpod 3.x | API Notifier thống nhất; không dùng riverpod_generator và provider legacy; dễ kiểm thử, tách logic khỏi widget |
| Giao diện Material 3 | material_ui | Thay cho `package:flutter/material.dart`, đã đóng băng từ Flutter 3.44; `cupertino_ui` chỉ thêm khi cần |
| Điều hướng | go_router | Khai báo route, deep link; gói đang ở chế độ bảo trì (chỉ sửa lỗi), đủ cho khoảng 10 route, đặt sau một lớp router mỏng để thay được |
| Lưu kịch bản | drift (SQLite) | Tìm kiếm và sắp xếp nhanh; dùng kèm drift_flutter, drift_dev, build_runner (chung với freezed) |
| Lưu cài đặt, preset | shared_preferences | Dữ liệu nhỏ dạng khóa-giá trị |
| Camera và quay video | camera | Chỉ là phương án dự phòng nếu giai đoạn 0 cho thấy wrapper native không khả thi; nếu dùng, phải đánh giá lại FR-47, FR-48, NFR-13 và ghi thành quyết định ở mục 7.6. Lớp chữ là widget nằm trên, không vào file video |
| Quyền | permission_handler | Camera, micro, lưu trữ/ảnh |
| Giữ màn hình sáng | wakelock_plus | Bật khi nhắc chữ, tắt khi thoát |
| Lưu vào thư viện ảnh | Mã native trong plugin riêng (MediaStore trên Android, PHPhotoLibrary trên iOS) | Cần `galleryRef` cho FR-49a và FR-49b, trong khi tài liệu của gói `gal` không nêu giá trị trả về là mã tham chiếu; `gal` chỉ là phương án dự phòng nếu T0-07 cho thấy đủ dùng |
| Chia sẻ | share_plus | Chia sẻ video |
| Nhập tệp .txt | file_picker | FR-04 |
| Hiển thị định dạng | Tự viết, không dùng package | FR-35; không dùng `flutter_markdown` (đã ngừng phát triển) hay `flutter_markdown_plus`, vì chúng dựng mỗi đoạn thành một widget và hỗ trợ nhiều cú pháp không cần cho máy nhắc chữ |
| Mua Pro | in_app_purchase | FR-70–FR-75; plugin chính thức, bọc StoreKit 2 và Google Play Billing; không dùng RevenueCat (D-13) |
| Đa ngôn ngữ | intl, gen_l10n (ARB) | Tiếng Việt và Anh; delegate của Material lấy từ `material_ui` (xác nhận cách nối khi khởi tạo) |
| Camera chất lượng gốc | Plugin native riêng bọc CameraX (Android) và AVFoundation (iOS), nối với Flutter qua Pigeon | Chỉ truyền cài đặt (sáng, flash, zoom, gương, độ phân giải); việc ghi hình do hệ điều hành xử lý, không mã hóa lại (FR-47, NFR-13) |

Stack đã chốt ngày 2026-11-15 (D-19) sau khi tra pub.dev và tài liệu Flutter. Dòng phiên bản chính: riverpod 3.x, go_router 18.x, drift 2.3x (sqlite3 3.x), permission_handler 12.x, share_plus 13.x, file_picker 12.x, wakelock_plus 1.x, Pigeon 29.x, in_app_purchase 3.x (StoreKit 2 là mặc định trên iOS), material_ui 1.x. Phiên bản chính xác khóa trong `pubspec.lock`; chỉ nâng giữa các giai đoạn, sau khi chạy lại bộ test và đo NFR-01. Mức hỗ trợ iOS và Android tối thiểu của từng plugin vẫn phải kiểm khi thêm vào dự án.

### 6.2 Cấu trúc thư mục

```text
lib/
  main.dart
  app/            # router, theme, localization
  core/           # tiện ích, hằng số, lỗi chung
  features/
    scripts/      # danh sách, soạn, nhập/xuất kịch bản
    prompter/     # màn nhắc chữ, bộ cuộn, điều khiển
    recorder/     # camera, ghi hình, lớp phủ chữ
    videos/       # danh sách video, lưu vào bộ sưu tập, vị trí lưu
    overlay/      # cửa sổ nổi Android (giai đoạn 4)
    settings/     # tùy biến, preset, quyền
    onboarding/
  data/           # repository, database, model
test/             # unit + widget test
integration_test/
pigeons/          # định nghĩa cầu nối Pigeon (camera, lưu trữ)
android/, ios/    # plugin native: CameraX, AVFoundation, dịch vụ cửa sổ nổi
```

### 6.3 Mô hình dữ liệu

| Thực thể | Trường chính |
| --- | --- |
| Script | id, title, content, wordCount (lấy từ PrompterDocument.wordCount, D-16), createdAt, updatedAt, folderId, tags (FR-05) |
| PrompterSettings | fontSize, lineSpacing, sidePadding, textColor, bgColor, fontFamily, align, wpm, mirrorH, mirrorV, guideEnabled, guidePosition, countdownSec, textOpacity, textOutline. Một bộ dùng chung cho mọi kịch bản; đổi nhanh bằng preset (FR-34) |
| Preset | id, name, settings (PrompterSettings) |
| Recording | id, scriptId (để trống khi kịch bản đã bị xóa, video vẫn giữ), filePath, audioSource (FR-41a, FR-41b), volume (bộ nhớ trong/thẻ nhớ, FR-63), thumbnailPath, duration, fileSize, resolution, fps, createdAt, savedToGalleryAt, galleryRef, galleryStatus (chưa lưu/đã lưu/đã bị xóa/không xác định; tính lúc chạy từ savedToGalleryAt và kết quả kiểm tra galleryRef, không lưu vào DB) |
| ProStatus | isPro, productId, transactionId (iOS) hoặc purchaseToken (Android), verifiedAt, pending; lưu trên máy làm bộ nhớ đệm, nguồn đúng luôn là cửa hàng (FR-73) |

### 6.4 Cơ chế cuộn chữ

- Dùng một `Ticker` cập nhật vị trí cuộn theo thời gian thực (đơn vị px/giây nhân độ trễ giữa hai khung), không cộng cố định mỗi khung, để tốc độ ổn định dù máy 60 hay 120 Hz.
- Quy đổi WPM sang px/giây dựa trên đo thực tế chiều cao văn bản sau khi bày chữ (`TextPainter`) chia cho số từ, nhờ đó WPM đúng với mọi cỡ chữ và độ rộng màn hình.
- Khi người dùng chạm/kéo, tạm ngắt Ticker, cập nhật offset theo ngón tay, rồi tiếp tục từ vị trí mới.
- Lật chữ bằng `Transform` (scale -1) trên lớp chữ, không đụng đến camera.
- Tránh `setState` toàn màn hình mỗi khung: chỉ lớp chữ được vẽ lại, bọc trong `RepaintBoundary`.
- Định dạng (FR-35): phân tích kịch bản một lần khi mở (một lượt duyệt theo dòng, không dùng biểu thức chính quy phức tạp) thành văn bản đã bỏ ký hiệu kèm danh sách đoạn định dạng và vị trí tiêu đề; dựng một `TextSpan` duy nhất, gộp các đoạn liền nhau cùng kiểu; chỉ dựng lại khi nội dung đổi, còn đổi cỡ chữ hay màu chỉ dựng lại kiểu chữ. Vị trí tiêu đề quy ra tọa độ cuộn bằng `TextPainter` để nhảy mốc (FR-18).

## 7. Kiểm thử, nghiệm thu, rủi ro và lộ trình

MVP gồm toàn bộ yêu cầu P0, tức giai đoạn 1 (nhắc chữ) và giai đoạn 2 (quay video) ở mục 7.4; bản được nghiệm thu khi đạt các tiêu chí dưới đây trên ít nhất hai máy Android và hai máy iPhone.

### 7.1 Chiến lược kiểm thử

| Cấp độ | Nội dung | Công cụ |
| --- | --- | --- |
| Unit test | Quy đổi WPM, đếm từ, repository kịch bản, xử lý preset, bộ phân tích định dạng FR-35 (từng cú pháp, ký hiệu không đóng, ký tự thoát, tiếng Việt có dấu) và đo thời gian theo NFR-15 | flutter_test, mocktail |
| Widget test | Màn soạn, màn nhắc chữ (tạm dừng, đổi tốc độ, mirror) | flutter_test, golden test |
| Integration test | UC-01 và UC-02 đầu cuối trên thiết bị thật | integration_test |
| Thử hiệu năng | NFR-01 khi cuộn; NFR-03, NFR-04 khi quay 1080p/30; NFR-14 khi quay 4K/30; bitrate theo NFR-13; độ lệch hình–tiếng theo NFR-16 | Flutter DevTools (profile mode), ffprobe, trạng thái nhiệt của hệ điều hành |
| Thử thủ công | Cuộc gọi đến, xoay màn hình (cả khi đang ghi), cắm/rút micro ngoài và ngắt Bluetooth khi đang quay, từ chối quyền, sắp hết và hết bộ nhớ, rút thẻ nhớ | Danh sách kiểm trên thiết bị thật |

Truy vết: mỗi FR P0 và P1 có ít nhất một test case mang mã `TC-<mã FR>-<số>` (ví dụ TC-FR13-01) trong kế hoạch kiểm thử; mỗi NFR được kiểm bằng cách đo ghi ở dòng Thử hiệu năng hoặc Thử thủ công, trên thiết bị chuẩn do đội kiểm thử chốt.

### 7.2 Tiêu chí nghiệm thu MVP

- [ ] Toàn bộ FR ưu tiên P0 hoạt động đúng trên thiết bị thật Android và iOS.
- [ ] Đạt các ngưỡng NFR-01 đến NFR-06, NFR-12, NFR-13 và NFR-16 (NFR-14 cho 4K là mục tiêu, không chặn nghiệm thu MVP).
- [ ] Video quay ra không chứa lớp chữ nhắc, đồng bộ hình và tiếng (NFR-16), lưu được trong bộ nhớ riêng của ứng dụng và có thể lưu ra bộ sưu tập của máy.
- [ ] Tiếng Việt có dấu hiển thị đúng ở mọi phông chữ cung cấp.
- [ ] Không còn lỗi mức nghiêm trọng (crash, mất dữ liệu, mất video) đang mở.
- [ ] Đã chạy thử trên TestFlight và Closed testing của Google Play, không còn lỗi chặn phát hành.
- [ ] Video quay đạt NFR-13: độ phân giải và fps đúng lựa chọn, bitrate khớp hồ sơ mã hóa của máy; danh sách video hiển thị đúng mọi video đã quay.

### 7.3 Rủi ro

| Rủi ro | Mức độ | Giảm thiểu |
| --- | --- | --- |
| Micro Bluetooth chất lượng thấp hoặc lệch tiếng với hình; hành vi định tuyến âm thanh khác nhau giữa các hãng Android | Trung bình | Thử với micro cắm dây và Bluetooth phổ biến ở giai đoạn 0; đo độ lệch hình–tiếng trong thử thủ công; Bluetooth tách thành FR-41b (P1), không chặn nghiệm thu MVP; ghi rõ giới hạn chất lượng trong FR-41b |
| Giật khung khi vừa quay vừa cuộn trên máy yếu | Cao | Thử sớm trên máy cấu hình thấp; giữ mặc định 1080p/30 fps (FR-42); giảm độ phức tạp lớp phủ |
| Hành vi camera khác nhau giữa các hãng máy Android | Cao | Thử trên nhiều hãng; chỉ bật tùy chọn máy hỗ trợ |
| Wrapper native chưa hỗ trợ hết tính năng camera (60 fps, 4K, chống rung) trên mọi máy | Trung bình | Lấy danh sách khả năng từ thiết bị, chỉ hiện tùy chọn máy hỗ trợ; kiểm thử trên nhiều hãng |
| iOS chỉ cấp quyền thêm vào thư viện nên không kiểm tra được video đã lưu còn hay bị xóa | Trung bình | MVP: Android kiểm tra qua MediaStore (FR-49a), iOS chỉ hiện chưa lưu/đã lưu; sau MVP iOS kiểm tra khi có quyền đọc (FR-49b); giai đoạn 0 xác nhận galleryRef và hành vi quyền giới hạn |
| iOS/Android hạn chế chạy nền, cửa sổ nổi | Trung bình | Cửa sổ nổi chỉ làm cho Android ở giai đoạn 4, iOS không hỗ trợ; spike trên nhiều hãng máy trước khi cam kết |
| Pro bị bẻ khóa trên Android vì chỉ xác minh trên máy (bản APK bị chỉnh sửa) | Thấp | Chấp nhận với giá bán thấp; kiểm chữ ký giao dịch; nếu thất thoát đáng kể thì chuyển sang xác minh qua RevenueCat, giao dịch cũ vẫn khôi phục được vì nằm ở cửa hàng |
| StoreKit 2 đôi khi trả danh sách quyền rỗng dù đã mua (lỗi đã được báo trên một số bản iOS 26) | Trung bình | Không khóa ngay khi danh sách rỗng mà dùng trạng thái lần xác minh gần nhất (FR-73); nút Khôi phục giao dịch mua luôn có trên SC-07 và SC-12 |
| Video chỉ nằm trong ứng dụng nên mất khi gỡ ứng dụng hoặc xóa dữ liệu | Trung bình | Nhắc ở SC-09 khi còn video chưa lưu (FR-44); biểu tượng trạng thái bộ sưu tập trên từng video (FR-49) |
| Nền tảng đổi nhanh: Flutter ra bản mới mỗi quý, Material và Cupertino tách khỏi SDK (bản trong SDK dự kiến bị đánh dấu lỗi thời từ bản tháng 11), Riverpod 3 đổi API, go_router chỉ còn sửa lỗi | Trung bình | Khóa SDK và package (FVM, pubspec.lock); dùng `material_ui` ngay từ đầu; chỉ nâng Flutter giữa các giai đoạn; giữ lớp điều hướng mỏng để thay go_router được |
| Chính sách của cửa hàng ứng dụng về quyền riêng tư và quyền đặc biệt (hiển thị trên ứng dụng khác, foreground service từ Android 14 cho FR-51) | Thấp | Viết chính sách quyền riêng tư và mô tả dùng quyền rõ ràng; khai báo loại foreground service và lý do dùng khi làm FR-51 |

### 7.4 Lộ trình phát triển

| Giai đoạn | Phạm vi | Cổng kiểm tra |
| --- | --- | --- |
| 0. Nền tảng và spike | Chưa có FR; khung dự án, spike camera native (FR-47, FR-48) và nguồn micro (FR-41a, FR-41b), lưu vào bộ sưu tập và galleryRef (FR-49a, FR-49b), đo hiệu năng cuộn | Wrapper native được xác nhận khả thi; nếu không, quyết định dùng gói camera (mục 6.1) |
| 1. MVP – nhắc chữ | FR-01–03, FR-10–13, FR-15–16, FR-19, FR-30–32, FR-60 | Cuộn đạt NFR-01, soạn và lưu kịch bản ổn định (NFR-05) |
| 2. MVP – quay video | FR-14, FR-40–45 (gồm FR-41a), FR-47–49, FR-49a, FR-61 | Video sạch, đồng bộ tiếng, không rớt khung; nghiệm thu MVP theo mục 7.2 |
| 3. Hoàn thiện | FR-04–05, FR-41b, FR-17–18, FR-35, FR-33–34, FR-49b, FR-50, FR-62, FR-63 | Mọi FR P1 hoạt động đúng trên thiết bị thật; không hồi quy tiêu chí nghiệm thu MVP (mục 7.2) |
| 4. Mở rộng | FR-06, FR-20, FR-46, FR-51–52, bản máy tính bảng. FR-20, FR-46, FR-51, FR-52 làm xong nhưng ẩn sau cờ tính năng, chưa phát hành công khai (D-14); FR-06 và bản máy tính bảng phát hành bình thường | Tính năng chạy đúng sau cờ; tùy phản hồi người dùng |
| 5. Bản Pro | FR-70–75 (làm cuối, D-13); chốt danh sách tính năng Pro và bật cờ các tính năng giai đoạn 4 cùng lúc ra bản Pro | Mua, khôi phục, offline, hoàn tiền hoạt động đúng với tài khoản thử của App Store và Google Play; danh sách tính năng Pro đã chốt |

Chưa có mốc ngày: sẽ gán sau khi xác định quy mô đội ngũ.

### 7.5 Câu hỏi mở

- Giá bán bản Pro và danh sách tính năng Pro. E-14 là đề xuất xuất phát; FR-20, FR-46, FR-51, FR-52 giữ ẩn đến khi chốt (D-14). Nếu muốn khóa thêm tính năng của MVP hoặc giai đoạn 3 thì phải chốt trước khi phát hành tính năng đó (xem FR-75).

### 7.6 Quyết định đã chốt (2026-11-12 đến 2026-11-15)

| # | Quyết định | Ảnh hưởng |
| --- | --- | --- |
| D-01 | MVP gồm cả nhắc chữ và quay video: toàn bộ P0, tức giai đoạn 1 và 2 | Mục 7, 7.4 |
| D-02 | Hành động cửa sổ nổi trên SC-10 thuộc FR-51 (P2, giai đoạn 4), không nằm trong MVP | FR-51, FR-61, UC-02, SC-10 |
| D-03 | Mặc định quay 1080p/30 fps; mức cao hơn chỉ hiện khi máy hỗ trợ, 4K có ngưỡng riêng | FR-42, NFR-03, NFR-04, NFR-14, 7.3 |
| D-04 | Chất lượng video đo bằng thông số (độ phân giải, fps, bitrate theo hồ sơ mã hóa của máy), không so với ứng dụng camera của hãng | FR-47, NFR-13, 6.1, 7.2 |
| D-05 | Trạng thái "đã bị xóa khỏi bộ sưu tập" làm theo hướng lai: Android kiểm tra qua MediaStore, iOS chỉ kiểm tra khi có quyền đọc thư viện; cách làm xác nhận ở giai đoạn 0 | FR-49, FR-49a, FR-49b, FR-61, 7.3 |
| D-06 | "Camera của hệ thống" là API camera gốc của máy (CameraX, AVFoundation), không phải ứng dụng camera mặc định | FR-47, 6.1 |
| D-07 | Tên dự án, tên ứng dụng và tên hiển thị trên cửa hàng đều là Cuevo | Mục 1, 2.4 |
| D-08 | iOS tối thiểu 16 (chủ sản phẩm xác nhận 2026-11-13); quyền chỉ thêm vào thư viện đã có từ iOS 14 | 2.2, NFR-09, FR-49a, FR-49b |
| D-09 | Hỗ trợ micro ngoài (2026-11-13): micro cắm dây là P0 (FR-41a); micro Bluetooth là P1, hỗ trợ trong khả năng thiết bị (FR-41b), theo đánh giá khả thi cùng ngày | FR-41a, FR-41b, SC-04, 7.1, 7.3, 7.4 |
| D-10 | Thiếu quyền micro thì không cho quay, kể cả khi có quyền camera: hiện SC-10 như thiếu camera. Không có chế độ quay video không tiếng, vì Cuevo là máy nhắc chữ, video không có lời đọc không đúng mục đích (2026-11-13) | FR-61, UC-02, SC-10 |
| D-11 | Tự viết bộ hiển thị Markdown cơ bản, ưu tiên nhanh và nhẹ; khi nhắc chữ ẩn ký hiệu `##`, tiêu đề chỉ in đậm, không đổi màu (D-15), cùng cỡ chữ (2026-11-13) | FR-18, FR-35, NFR-15, 6.1, 6.4 |
| D-12 | Không làm đồng bộ đám mây và không có máy chủ riêng ở thời điểm này, vì không muốn vận hành server (2026-11-13) | 1.2, 2.3, NFR-07 |
| D-13 | Bản Pro mua một lần, mở khóa theo tài khoản cửa hàng, chỉ dùng App Store và Google Play, không server, không RevenueCat; làm ở giai đoạn cuối; danh sách tính năng chốt sau (2026-11-13) | 3.8, SC-12, NFR-07, 6.1, 7.3, 7.4 |
| D-14 | Tính năng giai đoạn 4 thuộc danh sách ứng viên Pro (FR-20, FR-46, FR-51, FR-52) không phát hành công khai trước bản Pro: làm xong ở giai đoạn 4, ẩn sau cờ tính năng, ra mắt cùng giai đoạn 5. Danh sách tính năng Pro vẫn còn mở (2026-11-13) | FR-75, 7.4, 7.5, E-14 |
| D-15 | Tiêu đề Markdown chỉ in đậm, không đổi màu, nên không thêm cài đặt màu tiêu đề (2026-11-13) | FR-35, FR-32, D-11 |
| D-16 | Số từ chỉ lấy từ PrompterDocument.wordCount, bộ phân tích viết từ giai đoạn 1; chưa có mã nên không cần migration (2026-11-13) | 1.3, FR-02, 6.3 |
| D-17 | Quyền Bluetooth (Android 12 trở lên: BLUETOOTH_CONNECT) chỉ xin khi người dùng chọn micro Bluetooth, từ chối không chặn quay (2026-11-13) | FR-41b, FR-61 |
| D-18 | Giới thiệu lần đầu gồm 3 màn hướng dẫn và 1 màn xin quyền camera, micro; là ngoại lệ có chủ đích của NFR-08 (2026-11-13) | FR-62, SC-08, NFR-08 |
| D-19 | Stack chốt (2026-11-15): Flutter stable mới nhất khi khởi tạo (3.50; lùi 3.47.x nếu bản .0 lỗi), khóa bằng FVM; giao diện dùng `material_ui` thay cho `package:flutter/material.dart`; flutter_riverpod 3.x với API Notifier; go_router; drift; freezed; Pigeon 29.x; lưu thư viện bằng mã native thay cho `gal`; Android target API 36, iOS 16 | 2.2, 4, 6.1, 7.3 |

### 7.7 Đề xuất đã duyệt (2026-11-13)

Các đề xuất dưới đây do nhóm phân tích đưa ra trong lúc review và đã được chủ sản phẩm duyệt ngày 2026-11-13; có giá trị như quyết định ở mục 7.6. E-05 được sửa khi duyệt. Giữ mã E- để các tài liệu khác tham chiếu không đổi.

| # | Đề xuất | Ảnh hưởng |
| --- | --- | --- |
| E-01 | FR-45 nâng lên P0, chỉ đồng bộ một chiều: dừng quay thì dừng chữ; tạm dừng chữ hay hết kịch bản không dừng quay | FR-45, UC-02 |
| E-02 | NFR-14 (4K) là mục tiêu, không chặn nghiệm thu MVP; máy báo hỗ trợ 4K vẫn cho chọn kèm cảnh báo | NFR-14, 7.2 |
| E-03 | Tách trạng thái bộ sưu tập: Android (FR-49a) là P0, iOS (FR-49b) là P1 | FR-49a, FR-49b, FR-61, 7.4 |
| E-04 | Cách đếm "từ" theo âm tiết tiếng Việt; WPM mặc định 140 | 1.3, FR-02, FR-12 |
| E-05 | Ngôn ngữ lần đầu theo hệ thống (Việt/Anh), ngôn ngữ khác thì dùng tiếng Anh (chủ sản phẩm sửa khi duyệt) | 2.2, SC-10 |
| E-06 | Dải đánh dấu dòng mặc định ở 1/3 trên, chỉnh 10–50% hoặc tắt | FR-15, SC-06 |
| E-07 | Video loại trừ khỏi sao lưu iCloud và Android Auto Backup | NFR-07 |
| E-08 | Cảnh báo khi dung lượng trống dưới 1 GB lúc bấm quay | UC-02 |
| E-09 | Ngưỡng NFR-04: trạng thái nhiệt không vượt mức trung bình, pin giảm ≤ 10% sau 10 phút | NFR-04 |
| E-10 | Một bộ PrompterSettings dùng chung cho mọi kịch bản | 6.3 |
| E-11 | Xóa kịch bản không xóa video đã quay từ kịch bản đó | 6.3 |
| E-12 | Mặc định tự chọn micro cắm dây khi đang cắm; micro Bluetooth không tự chọn; micro ngoài bị ngắt khi đang quay thì chuyển sang micro trong máy, không dừng quay | FR-41a, FR-41b |
| E-13 | FR-35 là P1 (giai đoạn 3, cùng FR-18); ngưỡng NFR-15: 50 ms cho 5.000 từ, 200 ms cho 20.000 từ | FR-35, NFR-15, 7.4 |
| E-14 | Tính năng Pro đề xuất xuất phát (danh sách còn mở theo D-14), chỉ gồm tính năng chưa phát hành miễn phí: cuộn theo giọng nói (FR-20), cửa sổ nổi Android (FR-51), điều khiển từ thiết bị thứ hai (FR-52), lưới canh khung (FR-46). Phần cốt lõi (soạn, nhắc chữ, quay video, xuất sao lưu) giữ miễn phí | FR-75, SC-12 |
| E-15 | Thêm NFR-16: độ lệch hình–tiếng ≤ ±45 ms với micro trong máy và micro cắm dây, đưa vào tiêu chí nghiệm thu MVP | NFR-16, 7.1, 7.2 |
