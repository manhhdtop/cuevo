# Tài liệu thiết kế kỹ thuật – Cuevo (Flutter)

Ngày: 2026-11-06 · Cập nhật: 2026-11-15 (đồng bộ SRS: D-01–D-19, E-01–E-15) · Tác giả: Luong Yen

## 1. Quyết định kiến trúc và quy ước code

Tài liệu này là bản thiết kế để lập trình viên bắt tay vào code, dựa trên SRS (cuevo_srs.md) và kế hoạch task (cuevo_plan_and_tasks.md). Mã mẫu là khung để tham khảo, cần điều chỉnh theo phiên bản thư viện khi thực hiện.

| Chủ đề | Lựa chọn | Lý do | Phương án đã loại |
| --- | --- | --- | --- |
| State management | flutter_riverpod 3.x (API Notifier thống nhất; không dùng riverpod_generator, không dùng provider legacy) | Không phụ thuộc BuildContext, dễ test, dễ override khi test | Bloc: nhiều mã khung hơn cho ứng dụng nhỏ |
| Điều hướng | go_router | Khai báo route, tham số đường dẫn; gói đang ở chế độ bảo trì nhưng đủ cho khoảng 10 route, đặt sau một lớp router mỏng | Navigator 1.0 thủ công |
| Lưu kịch bản | drift (SQLite) | Truy vấn có kiểu, migration rõ ràng, tìm kiếm đơn giản | Isar: cần kiểm tra thêm về tình trạng bảo trì |
| Lưu cài đặt | shared_preferences | Dữ liệu nhỏ, dạng JSON có số phiên bản | Bảng DB riêng: thừa |
| Camera | Plugin native riêng qua Pigeon, bọc CameraX và AVFoundation | Ghi thẳng bằng API gốc, không mã hóa lại, thông số theo hồ sơ mã hóa của máy (NFR-13); chủ động các cài đặt sáng, flash, zoom, nguồn micro | Plugin `camera`: dùng làm dự phòng nếu spike T0-04 thất bại |
| Cuộn chữ | Ticker tự viết + ScrollController | Tốc độ theo thời gian thực, độc lập tần số quét màn hình | AnimationController dài: lệch khi kéo tay |
| Model bất biến | freezed + json_serializable | copyWith, so sánh giá trị, đọc/ghi JSON cài đặt | Viết tay: dễ sai |
| Giao diện | material_ui (Material 3) | Bản trong SDK đã đóng băng từ Flutter 3.44 và sắp bị đánh dấu lỗi thời | `package:flutter/material.dart` |
| Lưu vào thư viện | Mã native trong plugin (MediaStore, PHPhotoLibrary) | Cần `galleryRef` (FR-49a, FR-49b) | Gói `gal`: dự phòng nếu T0-07 cho thấy đủ dùng |
| Mua Pro (giai đoạn 5) | in_app_purchase | Plugin chính thức, bọc StoreKit 2 và Google Play Billing, không cần server (SRS D-13) | RevenueCat: cần dịch vụ bên thứ ba, chỉ là phương án dự phòng |

**Quy ước code**

- Thư mục theo tính năng (feature-first); mỗi tính năng có `presentation`, `application`, `domain`, `data` khi cần.
- Lớp `domain` không import `flutter/*`; chỉ widget mới đụng đến BuildContext.
- Mọi chuỗi hiển thị qua file ARB (vi, en); không mã cứng màu, dùng theme.
- Lint: flutter_lints; `flutter analyze` phải sạch khi merge.
- Đặt tên file snake_case; provider kết thúc bằng `Provider`; notifier kết thúc bằng `Controller` hoặc `Notifier`.

## 2. Cấu trúc project, route, phân lớp

Mã phân lớp theo thứ tự gọi: widget → provider/notifier → repository hay service (interface) → implementation (drift, Pigeon, shared_preferences). Không đảo chiều.

```text
lib/
  main.dart
  app/
    app.dart                 # MaterialApp.router
    router.dart              # go_router
    theme.dart
    l10n/                    # app_vi.arb, app_en.arb
  core/
    result.dart              # Result<T>, AppError
    permissions/             # PermissionService (interface + impl)
    utils/                   # formatters.dart (số từ do PrompterMarkdown tính)
  features/
    scripts/
      domain/                # Script, ScriptRepository
      data/                  # drift tables, ScriptRepositoryImpl
      application/           # ScriptListController, ScriptEditController
      presentation/          # SC-01, SC-02
    prompter/
      domain/                # PrompterSettings, PrompterMarkdown
      application/           # PrompterController, ScrollEngine
      presentation/          # SC-03, SC-06, widgets
    recorder/
      domain/                # CameraCapabilities, RecordingConfig
      data/                  # camera_api.g.dart (Pigeon), CameraRepositoryImpl
      application/           # RecorderController
      presentation/          # SC-04, SC-10
    videos/
      domain/                # Recording, RecordingRepository
      data/
      application/           # VideoLibraryController (gom nhóm theo ngày, chọn nhiều)
      presentation/          # SC-05, SC-09
    settings/                # SC-07
    onboarding/              # SC-08
    pro/                     # ProService, SC-12 (giai đoạn 5)
    overlay/                 # cửa sổ nổi Android (giai đoạn 4)
android/app/src/main/kotlin/.../camera/   # CameraX
ios/Runner/Camera/                        # AVFoundation
pigeons/camera_api.dart
```

| Route | Màn hình | Ghi chú |
| --- | --- | --- |
| `/` | SC-01 Danh sách kịch bản | Trang chủ, có thanh điều hướng tới `/videos` |
| `/scripts/:id/edit` | SC-02 Soạn kịch bản | `id = new` khi tạo mới |
| `/scripts/:id/prompt` | SC-03 Nhắc chữ | Toàn màn hình, ẩn thanh hệ thống |
| `/scripts/:id/record` | SC-04 Quay video | Kiểm tra quyền trước khi vào |
| `/videos` | SC-09 Danh sách video | Lưới theo ngày |
| `/videos/:id` | SC-05 Xem lại video | |
| `/settings` | SC-07 Cài đặt | |
| `/settings/display` | SC-06 Tùy biến hiển thị | Cũng mở được từ SC-03 dạng bottom sheet |
| `/onboarding` | SC-08 Giới thiệu | Chỉ hiện lần đầu; 3 màn hướng dẫn và 1 màn xin quyền camera, micro (FR-62) |
| `/permission-error` | SC-10 Lỗi thiếu quyền | Nhận tham số quyền nào thiếu |
| `/pro` | SC-12 Bản Pro | Giai đoạn 5, mở từ SC-07 và khi chạm tính năng Pro chưa mở khóa |
| SC-11 | Bảng điều khiển cửa sổ nổi | Activity riêng nền trong suốt trên Android (giai đoạn 4), mở từ thông báo, không đi qua go_router |

## 3. Mô hình dữ liệu và schema lưu trữ

Kịch bản và video lưu trong SQLite (drift); cài đặt nhắc chữ lưu dạng JSON trong shared_preferences với khóa `prompter_settings_v1`. File video nằm trong thư mục của ứng dụng và được sao chép vào thư viện máy khi người dùng lưu (FR-44, FR-49).

```dart
class Scripts extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(max: 200)();
  TextColumn get content => text()();
  IntColumn get wordCount => integer().withDefault(const Constant(0))();
  DateTimeColumn get createdAt => dateTime()();
  DateTimeColumn get updatedAt => dateTime()();
}

class Recordings extends Table {
  IntColumn get id => integer().autoIncrement()();
  IntColumn get scriptId =>
      integer().nullable().references(Scripts, #id, onDelete: KeyAction.setNull)();
  TextColumn get filePath => text()();
  TextColumn get thumbnailPath => text().nullable()();
  IntColumn get durationMs => integer()();
  IntColumn get fileSizeBytes => integer()();
  IntColumn get width => integer()();
  IntColumn get height => integer()();
  IntColumn get fps => integer()();
  TextColumn get volume => text().withDefault(const Constant('internal'))();   // internal | removable
  DateTimeColumn get savedToGalleryAt => dateTime().nullable()();
  TextColumn get galleryRef => text().nullable()();   // URI (Android) hoặc localIdentifier (iOS)
  TextColumn get audioSource => text().nullable()();  // builtIn | wired | usb | bluetooth (FR-41a, FR-41b)
  DateTimeColumn get createdAt => dateTime()();
}
```

- Danh sách video (SC-09) đọc `Recordings` sắp xếp `createdAt` giảm dần rồi gom theo ngày ở lớp application; chỉ hiển thị thời lượng và dung lượng tệp cho mỗi ô. Khi còn video chưa lưu, đầu danh sách có dòng nhắc rằng gỡ ứng dụng hoặc xóa dữ liệu sẽ mất các video này (FR-44).
- Nếu file video không còn (người dùng xóa ngoài ứng dụng), đánh dấu bản ghi là mất file và dọn khi mở SC-09, không để ô trống.
- **Vị trí lưu video (FR-63).** Cột `volume` ghi video nằm ở `internal` hay `removable` (thẻ nhớ). Video trên thẻ mà thẻ chưa gắn thì không bị coi là mất file: không dọn bản ghi, ô hiện trạng thái Thẻ nhớ chưa gắn và vô hiệu hóa xem, chia sẻ, lưu vào bộ sưu tập (xóa bản ghi vẫn được). Ảnh thu nhỏ luôn lưu ở bộ nhớ trong để danh sách vẫn hiện khi thẻ bị rút.
- Đếm từ theo định nghĩa SRS 1.3, chỉ có một nơi tính: `PrompterMarkdown.parse(content).wordCount` (D-16). Bộ phân tích bỏ ký hiệu định dạng (`#`, `##`, `###`, dấu đầu dòng, `1.`, `**`, `*`, `_`, `\`), tách theo khoảng trắng và chỉ tính chuỗi có ít nhất một chữ cái hoặc chữ số (`RegExp(r'[\p{L}\p{N}]', unicode: true)`); tiếng Việt mỗi âm tiết là một từ. Cột `Scripts.wordCount` ghi giá trị này mỗi lần lưu. Bộ phân tích có từ giai đoạn 1 (T1-02) nên số từ không đổi khi FR-35 hiển thị định dạng ở giai đoạn 3, không cần migration.
- Xóa kịch bản: `scriptId` của video thành null (`KeyAction.setNull`), video vẫn giữ (SRS E-11).
- **Loại trừ khỏi sao lưu (NFR-07):** iOS đặt `isExcludedFromBackup = true` cho thư mục video; Android loại thư mục video trong `data_extraction_rules.xml` (API 31 trở lên) và `full_backup_content.xml` (API 30 trở xuống), cho cả miền `file` và `external`. Kịch bản (DB) và cài đặt vẫn theo sao lưu hệ thống.
- Migration: `schemaVersion` bắt đầu từ 1; mỗi thay đổi bảng phải có bước migration và test.
- Nhãn/thư mục kịch bản (FR-05) sẽ thêm bảng `Tags` ở giai đoạn 3.
- Video chỉ nằm trong bộ nhớ riêng của ứng dụng cho đến khi người dùng bấm Lưu vào bộ sưu tập. Hành động này gọi `RecordingRepository.saveToGallery(ids)`, sao chép file sang thư viện máy (MediaStore trên Android, thư viện ảnh trên iOS), đặt `savedToGalleryAt` và báo tiến độ. Xóa trong ứng dụng không xóa bản đã lưu ra bộ sưu tập.
- **Biểu tượng trạng thái bộ sưu tập (SC-09).** Trạng thái tính từ `savedToGalleryAt` và kết quả kiểm tra `galleryRef`, không lưu vào DB (`galleryStatus` trong SRS 6.3 là giá trị tính lúc chạy):
  - `notSaved`: `savedToGalleryAt` rỗng, biểu tượng `Icons.save_alt_outlined` màu nhạt.
  - `saved`: có `galleryRef` và bản trong thư viện còn tồn tại, biểu tượng `Icons.check_circle`.
  - `removed`: có `galleryRef` nhưng bản trong thư viện không còn, biểu tượng `Icons.error_outline` màu cảnh báo; chạm vào sẽ mở hộp thoại Lưu lại.
- **Kiểm tra bản trong thư viện:** chỉ kiểm tra các video đã có `galleryRef`, chạy nền khi mở SC-09 và khi ứng dụng quay lại (`resumed`), kết quả giữ trong bộ nhớ (`Map<int, GalleryStatus>`).
  - Android (FR-49a, MVP): truy vấn `ContentResolver` theo URI trong `galleryRef`; mục do ứng dụng tạo nên không cần xin thêm quyền (xác nhận ở T0-07, nhất là Android 8–9).
  - iOS trong MVP: chỉ có quyền chỉ thêm (`.addOnly`), không đọc được thư viện, nên chỉ hiện `notSaved` hoặc `saved` dựa trên `savedToGalleryAt`.
  - iOS từ giai đoạn 3 (FR-49b, T3-12): khi quyền đọc đã có, hoặc người dùng chọn Kiểm tra trạng thái (lúc đó mới gọi `PHPhotoLibrary.requestAuthorization(for: .readWrite)`), gọi `PHAsset.fetchAssets(withLocalIdentifiers:)`. Bị từ chối, hoặc quyền giới hạn không thấy video, thì trạng thái là `unknown` và hiển thị như `saved`, không báo đã xóa.
- Biểu tượng có `Semantics(label: ...)` và tooltip ("Chưa lưu", "Đã lưu vào bộ sưu tập", "Bản trong bộ sưu tập đã bị xóa") để không phụ thuộc vào màu (NFR-10).

**PrompterSettings** (JSON, một bộ dùng chung cho mọi kịch bản): `fontSize` (dp, không nhân `textScaler`), `lineSpacing`, `sidePadding`, `textColor`, `bgColor`, `fontFamily`, `align`, `wpm` (mặc định 140), `mirrorH`, `mirrorV`, `guideEnabled` (mặc định true), `guidePosition` (0.10–0.50, mặc định 0.33), `countdownSec`, `textOpacity` (mặc định 1.0), `textOutline` (mặc định true).

**AppSettings** (JSON, khóa `app_settings_v1`, tách khỏi `PrompterSettings`): `videoStorage` (`internal` | `removable`, mặc định `internal`). Giữ nguyên lựa chọn kể cả khi thẻ đang rút, chỉ đổi cách quay tạm thời (xem mục 6).

**OverlaySettings** (JSON, khóa `overlay_settings_v1`, giai đoạn 4, chỉ Android): `anchor` (lưới 3×3), `offsetX`, `offsetY`, `widthPct`, `heightPct`, và dùng chung `textColor`, `bgColor`, `textOpacity`, `fontSize`, `wpm` với `PrompterSettings`. Service và SC-11 cùng đọc/ghi qua `OverlaySettingsRepository`.

**ProStatus** (JSON, khóa `pro_status_v1`, giai đoạn 5, xem mục 10): `isPro`, `productId`, `transactionId` (iOS) hoặc `purchaseToken` (Android), `verifiedAt`, `pending`.

## 4. State management

| Provider | Loại | Giữ gì | Phụ thuộc |
| --- | --- | --- | --- |
| `databaseProvider` | Provider | Thể hiện drift | – |
| `scriptRepositoryProvider` | Provider | `ScriptRepository` | databaseProvider |
| `scriptListProvider` | StreamProvider | Danh sách kịch bản, từ khóa tìm kiếm | scriptRepositoryProvider |
| `scriptEditControllerProvider` | Notifier tự hủy (`NotifierProvider.autoDispose.family`, id nhận qua hàm dựng) | Tiêu đề, nội dung, trạng thái lưu nháp | scriptRepositoryProvider |
| `settingsProvider` | Notifier | `PrompterSettings`, tự lưu khi đổi | shared_preferences |
| `prompterControllerProvider` | Notifier tự hủy (`NotifierProvider.autoDispose`) | `running`, `progress`, `wpm` | settingsProvider, ScrollEngine |
| `cameraRepositoryProvider` | Provider | Cầu nối Pigeon | – |
| `recorderControllerProvider` | Notifier tự hủy (`NotifierProvider.autoDispose`) | `idle / countingDown / preparing / recording / saving / error`, thời gian quay, đếm ngược trước khi ghi | cameraRepositoryProvider, permissionServiceProvider |
| `permissionServiceProvider` | Provider | Trạng thái quyền camera, micro, thư viện; quyền hiển thị trên ứng dụng khác từ giai đoạn 4 | – |
| `audioInputProvider` | Notifier | Danh sách nguồn micro, nguồn đang dùng; tự chọn micro cắm dây, không tự chọn Bluetooth (FR-41b), chọn Bluetooth thì xin quyền trước (D-17); cập nhật khi cắm/rút hoặc kết nối/ngắt Bluetooth | cameraRepositoryProvider |
| `videoLibraryControllerProvider` | Notifier | Danh sách nhóm theo ngày, tập đang chọn | recordingRepositoryProvider |
| `storageProvider` | AsyncNotifier | Danh sách ổ lưu (trong, thẻ nhớ) kèm dung lượng trống, ổ đang chọn; tự cập nhật khi gắn/rút thẻ | storageRepositoryProvider, shared_preferences |

```dart
enum RecorderStatus { idle, countingDown, preparing, recording, saving, error }

class RecorderController extends Notifier<RecorderState> {   // đăng ký bằng NotifierProvider.autoDispose (Riverpod 3)
  @override
  RecorderState build() {
    ref.onDispose(() => ref.read(cameraRepositoryProvider).dispose());
    return const RecorderState.idle();
  }

  Future<void> start(RecordingConfig config) async {
    final perm = await ref.read(permissionServiceProvider).ensureCameraAndMic();
    if (!perm.granted) { state = RecorderState.permissionDenied(perm); return; }
    state = const RecorderState.preparing();
    await ref.read(cameraRepositoryProvider).startRecording(config);
    state = const RecorderState.recording();
  }
}
```

Quy tắc (FR-45, đồng bộ một chiều): khi `RecorderState` chuyển sang `recording`, `PrompterController` tự chạy; khi chuyển khỏi `recording`, tự dừng. Đồng bộ bằng cách `PrompterController` lắng nghe `recorderControllerProvider`, không gọi ngược: tạm dừng chữ, kéo tay hay hết kịch bản (`finished`) đều không ảnh hưởng việc ghi.

Lưu ý Riverpod 3: provider lỗi lúc khởi tạo tự thử lại theo mặc định; tắt thử lại (tham số `retry`) cho provider camera, quyền và lưu video để lỗi hiện ngay. Listener của widget không hiển thị bị tạm dừng, nên `PrompterController` nghe `recorderControllerProvider` bằng `ref.listen` ngay trong notifier, không dựa vào widget; kiểm lại khi code.

Khi bắt đầu ghi, `RecorderController` khóa hướng màn hình hiện tại bằng `SystemChrome.setPreferredOrientations` và trả lại cấu hình cũ khi dừng ghi.

## 5. Scroll engine

Tốc độ cuộn tính theo thời gian thực giữa hai khung hình, nên ổn định trên máy 60 Hz hay 120 Hz. Quy đổi WPM sang px/giây dùng chiều cao văn bản đo thực tế:

```latex
v_{px/s} = \frac{WPM \times H_{text} / N_{words}}{60}
```

Trong đó `H_text` là chiều cao văn bản sau khi bày chữ với cỡ chữ và độ rộng hiện tại (đo bằng `TextPainter.layout`), `N_words` là số từ. Phải tính lại khi đổi cỡ chữ, lề hoặc xoay màn hình. Kết quả là tốc độ trung bình; đoạn có nhiều dòng trống sẽ cuộn hơi nhanh hơn đoạn đặc.

```dart
class ScrollEngine {
  ScrollEngine({required this.controller, required TickerProvider vsync, this.onEnd}) {
    _ticker = vsync.createTicker(_onTick);
  }

  final ScrollController controller;
  final VoidCallback? onEnd;
  late final Ticker _ticker;
  Duration _last = Duration.zero;
  double pxPerSecond = 0;

  bool get running => _ticker.isActive;

  static double speedFor({required int wpm, required double textHeight, required int words}) =>
      words == 0 ? 0 : wpm * (textHeight / words) / 60;

  void start() {
    _last = Duration.zero;          // Ticker.start() đặt lại elapsed về 0
    if (!_ticker.isActive) _ticker.start();
  }

  void pause() => _ticker.stop();

  void _onTick(Duration elapsed) {
    final dt = (elapsed - _last).inMicroseconds / 1e6;
    _last = elapsed;
    if (!controller.hasClients) return;
    final max = controller.position.maxScrollExtent;
    final next = controller.offset + pxPerSecond * dt;
    if (next >= max) {
      controller.jumpTo(max);
      pause();
      onEnd?.call();
      return;
    }
    controller.jumpTo(next);
  }

  void dispose() => _ticker.dispose();
}
```

- **Kéo tay:** khi người dùng chạm xuống (`Listener.onPointerDown`) gọi `pause()`, nhấc tay (`onPointerUp`) thì `start()` lại nếu trước đó đang chạy (FR-13).
- **Lật chữ:** bọc khối chữ bằng `Transform.scale(scaleX: -1)` (ngang) hoặc `scaleY: -1` (dọc); không động vào preview camera.
- **Hiệu năng:** khối chữ nằm trong `RepaintBoundary`; không gọi `setState` mỗi khung, để `ScrollController` tự vẽ lại. Thanh tiến độ chỉ cập nhật mỗi 100 ms.
- **Định dạng Markdown cơ bản (FR-35, NFR-15):** `PrompterMarkdown.parse(String)` nằm ở `features/prompter/domain`, Dart thuần, không import Flutter. Duyệt từng dòng một lượt, trả về `PrompterDocument { String plainText; List<StyledRun> runs; List<int> headingOffsets; int wordCount }`; `StyledRun` gồm vị trí đầu, cuối và cờ kiểu (đậm, nghiêng, tiêu đề, dấu đầu dòng). Lớp presentation dựng một `TextSpan` từ `runs` và kiểu chữ hiện tại, gộp các đoạn liền nhau cùng kiểu. Kết quả phân tích lưu theo nội dung kịch bản; đổi cỡ chữ, màu hay viền chỉ dựng lại `TextSpan`, không phân tích lại. Tiêu đề chỉ in đậm, không đổi cỡ chữ hay màu (D-15), để `speedFor` vẫn đúng và không cần thêm cài đặt màu. Ở giai đoạn 1 (T1-02) hàm này chỉ điền `plainText` và `wordCount` trong cùng một lượt duyệt; `runs` và `headingOffsets` được thêm ở T3-15. Nhảy mốc (FR-18) dùng `TextPainter.getOffsetForCaret` tại `headingOffsets`. Nếu kịch bản 20.000 từ vượt NFR-15 thì chuyển bước phân tích sang `Isolate.run`.
- **Chữ không nền khi quay:** khối chữ không dùng `Container` có màu; thêm viền hoặc bóng bằng `TextStyle.shadows` hoặc `Paint` nét viền; mặc định độ đậm 100% và bật viền (FR-43).
- **Cỡ chữ nhắc:** bọc khối chữ bằng `MediaQuery.withNoTextScaling` để cỡ chữ 20–120 dp không nhân theo cỡ chữ hệ thống (FR-30).
- **Đếm ngược (FR-14), chỉ dành cho quay video.** `RecorderController` đếm `settings.countdownSec` giây (mặc định 3, dải 0–10, 0 là bỏ qua) trước khi gọi `startRecording`; về 0 thì bắt đầu ghi hình và gọi `PrompterController.start()` cùng lúc. Chạm màn hình khi đang đếm thì hủy, về `idle`.
  - `PrompterController` không có pha đếm ngược (các pha: `idle, running, paused, finished`): Bắt đầu, Tiếp tục và thả tay sau khi kéo thủ công (FR-13) đều gọi `engine.start()` ngay.
  - Số đếm vẽ ở lớp trên cùng, chữ lớn không nền; dùng `Semantics(liveRegion: true)` để trình đọc màn hình đọc số.

## 6. Camera native

Flutter chỉ gửi lệnh và cài đặt qua Pigeon; ghi hình do CameraX (Android) và AVFoundation (iOS) làm và ghi thẳng ra file, không đi qua Dart. Preview đưa lên Flutter bằng Texture để lớp chữ ở trên vẽ rẻ; nếu gặp lỗi trên một số máy thì đổi sang PlatformView. Pigeon: mã Dart, Kotlin và Swift phải sinh bằng cùng một phiên bản (29.x ở thời điểm chốt), dùng kênh nền tảng chuẩn (`@async` sinh `suspend` và `async`), không dùng chế độ FFI/JNI còn thử nghiệm; file định nghĩa nằm cùng repo, không tách gói.

```dart
// pigeons/camera_api.dart
enum LensFacing { back, front }

class VideoFormat {
  VideoFormat({required this.width, required this.height, required this.fps});
  int width; int height; int fps;
}

class CameraCapabilities {
  CameraCapabilities({required this.formats, required this.hasTorch,
    required this.minZoom, required this.maxZoom,
    required this.minExposure, required this.maxExposure,
    required this.supportsStabilization, required this.hasFront});
  List<VideoFormat?> formats;
  bool hasTorch; double minZoom; double maxZoom;
  double minExposure; double maxExposure;
  bool supportsStabilization; bool hasFront;
}

enum AudioInputType { builtIn, wired, usb, bluetooth }

class AudioInput {
  AudioInput({required this.id, required this.name, required this.type});
  String id; String name; AudioInputType type;
}

class RecordingConfig {
  RecordingConfig({required this.format, required this.outputPath,
    this.audioInputId, required this.targetBitrate});   // luôn ghi kèm tiếng (D-10)
  VideoFormat format; String outputPath;
  String? audioInputId;   // null = nguồn hệ thống đang chọn
  int targetBitrate;      // từ hồ sơ mã hóa của máy (NFR-13)
}

class RecordingResult {
  RecordingResult({required this.path, required this.durationMs, required this.sizeBytes});
  String path; int durationMs; int sizeBytes;
}

@HostApi()
abstract class CameraHostApi {
  @async CameraCapabilities initialize(LensFacing lens);
  int createPreviewTexture();               // trả về textureId
  void setTorch(bool on);
  void setZoom(double ratio);
  void setExposure(double ev);
  void setFocusPoint(double x, double y);   // 0..1
  void setStabilization(bool on);
  List<AudioInput?> listAudioInputs();
  void setAudioInput(String id);
  @async void startRecording(RecordingConfig config);
  @async RecordingResult stopRecording();
  void dispose();
}

@FlutterApi()
abstract class CameraFlutterApi {
  void onInterrupted(String reason);        // cuộc gọi, thiếu bộ nhớ, mất camera
  void onError(String code, String message);
  void onAudioInputsChanged(List<AudioInput?> inputs, String activeId);   // cắm/rút, Bluetooth
}
```

| Cài đặt | Android (CameraX) | iOS (AVFoundation) |
| --- | --- | --- |
| Ghi hình | `Recorder` + `VideoCapture`, `FileOutputOptions` | `AVCaptureMovieFileOutput` |
| Đèn pin | `CameraControl.enableTorch` | `AVCaptureDevice.torchMode` (khóa cấu hình khi đổi) |
| Zoom | `CameraControl.setZoomRatio` | `videoZoomFactor` |
| Ánh sáng (bù sáng) | `setExposureCompensationIndex` | `setExposureTargetBias` |
| Lấy nét | `FocusMeteringAction` | `focusPointOfInterest` |
| Chống rung | Bật ổn định hình ảnh nếu thiết bị hỗ trợ | `preferredVideoStabilizationMode` |
| Độ phân giải, fps | `QualitySelector`; fps qua `Camera2Interop` | `activeFormat` + `activeVideoMin/MaxFrameDuration` |
| Lật xem trước (camera trước) | Lật Texture ở Flutter (`Transform`) | Lật Texture ở Flutter (`Transform`) |
| Nguồn micro | Liệt kê bằng `AudioManager.getDevices(GET_DEVICES_INPUTS)`. CameraX thu âm bằng `AudioRecord` với nguồn MIC; micro cắm dây/USB được hệ thống tự định tuyến (FR-41a). Bluetooth (FR-41b): `setCommunicationDevice` với thiết bị `TYPE_BLUETOOTH_SCO` hoặc `TYPE_BLE_HEADSET` (API 31 trở lên; cũ hơn dùng `startBluetoothSco`), cần xác nhận với CameraX ở T0-08 (chưa có kết quả thử trên máy thật); gọi `clearCommunicationDevice()` khi dừng quay hoặc rời SC-04. `Recorder.Builder.setAudioSource` có thể chưa công khai, xác nhận ở T0-08 | `automaticallyConfiguresApplicationAudioSession = false`, category `.playAndRecord` với `.allowBluetooth` (HFP), `setPreferredInput` theo `availableInputs`; chọn micro HFP thì đầu ra cũng chuyển sang tai nghe. iOS 26 trở lên: bật `configuresApplicationAudioSessionForBluetoothHighQualityRecording` (hoặc tùy chọn `.bluetoothHighQualityRecording`) cho AirPods. Dừng quay thì `setActive(false, options: .notifyOthersOnDeactivation)` |

**Chất lượng video (FR-47, NFR-13)**

- Định dạng mặc định 1080p/30; không hỗ trợ thì lấy mức gần nhất thấp hơn trong `CameraCapabilities.formats` (FR-42).
- Bitrate mục tiêu lấy từ hồ sơ mã hóa của máy: Android `EncoderProfiles` (API 31 trở lên) hoặc `CamcorderProfile`, truyền vào `Recorder.Builder.setTargetVideoEncodingBitRate`; iOS lấy mốc so sánh từ `recommendedVideoSettingsForAssetWriter` cho cùng định dạng, `AVCaptureMovieFileOutput` dùng cài đặt mặc định của hệ thống. T0-04 xác nhận hai giá trị iOS này khớp nhau.
- Không chèn bước xử lý khung hình (filter, vẽ lên video, đọc pixel về Dart).
- Video ghi ra không lật gương; chỉ xem trước có thể lật.
- Trước khi phát hành, đo độ phân giải, fps và bitrate bằng ffprobe theo điều kiện của NFR-13 (60 giây, cảnh nhiều chuyển động, đủ sáng) (T2-14). Không so với ứng dụng camera của hãng (SRS D-04).
- Micro ngoài bị ngắt khi đang quay: phía native chuyển sang micro trong máy, không dừng ghi, rồi báo `onAudioInputsChanged` để Flutter hiện thông báo (FR-41a, FR-41b).
- Ngắt quãng (cuộc gọi, thiếu bộ nhớ, mất camera): phía native đóng file hợp lệ rồi báo `onInterrupted`; Flutter lưu phần đã ghi và hiện thông báo (NFR-06).

**Vị trí lưu video: bộ nhớ trong hoặc thẻ nhớ (FR-63)**

```dart
// pigeons/storage_api.dart
class StorageVolume {
  StorageVolume({required this.id, required this.isRemovable, required this.rootPath,
    required this.freeBytes, required this.totalBytes, required this.mounted});
  String id;            // 'internal' | 'removable'
  bool isRemovable;
  String rootPath;      // thư mục riêng của ứng dụng trên ổ đó
  int freeBytes; int totalBytes;
  bool mounted;
}

@HostApi()
abstract class StorageHostApi {
  List<StorageVolume?> listVolumes();
}

@FlutterApi()
abstract class StorageFlutterApi {
  void onVolumesChanged(List<StorageVolume?> volumes);   // gắn hoặc rút thẻ
}
```

- **Android:** dùng `Context.getExternalFilesDirs(null)`: phần tử đầu là bộ nhớ trong, các phần tử sau là ổ ngoài. Chỉ nhận ổ có `Environment.isExternalStorageRemovable(dir)` là true và `getExternalStorageState(dir)` là `MEDIA_MOUNTED`; bỏ qua ổ giả lập. Thư mục này là vùng riêng của ứng dụng nên không cần xin quyền, và bị xóa khi gỡ ứng dụng. Lắng nghe `ACTION_MEDIA_MOUNTED`, `ACTION_MEDIA_EJECT` và `ACTION_MEDIA_REMOVED` để gọi `onVolumesChanged`.
- **iOS:** không có thẻ nhớ; `listVolumes()` chỉ trả về ổ trong và SC-07 ẩn mục Vị trí lưu video.
- **Chọn nơi quay:** `StorageRepository.outputDirFor(videoStorage)` trả thư mục đích truyền vào `RecordingConfig.outputPath`. Nếu chọn `removable` mà thẻ chưa gắn, dùng `internal` cho lần quay đó và hiện thông báo; không tự đổi lựa chọn đã lưu.
- **Dung lượng:** trước khi quay, nếu ổ đích còn dưới 1 GB thì cảnh báo kèm số phút ước tính (dung lượng trống chia bitrate mục tiêu) và gợi ý đổi ổ, người dùng vẫn có thể quay (UC-02); hết dung lượng giữa chừng được xử lý như ngắt quãng (`onInterrupted`, NFR-06).
- **Thẻ chậm:** quay 4K hoặc 60 fps vào thẻ chậm có thể rơi khung hình. Khi chọn thẻ nhớ mà độ phân giải/fps cao hơn 1080p30, SC-07 và SC-04 hiện lời nhắc nên dùng thẻ UHS-I U3 trở lên hoặc bộ nhớ trong.
- **Video cũ:** không tự chuyển giữa hai ổ; đổi vị trí chỉ áp dụng cho video quay sau đó. Chuyển video có sẵn là tính năng riêng, chưa nằm trong phạm vi.

## 7. Luồng xin quyền

Sơ đồ luồng xin quyền (Android và iOS): vào màn quay → kiểm tra camera và micro → đủ quyền thì vào SC-04; thiếu quyền thì hiện màn giải thích lý do, gọi hệ thống xin quyền; bị từ chối thì hiện SC-10 (nêu quyền thiếu, nút mở Cài đặt hệ thống). Quay lại ứng dụng thì kiểm tra lại mọi quyền. Lần đầu mở, màn giới thiệu (FR-62, giai đoạn 3) có thêm 1 màn xin quyền camera và micro bằng cùng `ensureCameraAndMic()`; bỏ qua thì xin lại khi vào màn quay (D-18). Từ giai đoạn 4 (FR-51, ngoài MVP), SC-10 trên Android có thêm nhánh kiểm tra quyền Hiển thị trên ứng dụng khác: đã cấp thì cho tiếp tục ở chế độ cửa sổ nổi, chưa cấp thì cảnh báo rồi vẫn cho nhắc chữ trong ứng dụng.

Camera và micro là quyền bắt buộc cho việc quay trong app trên cả hai nền tảng; thiếu một trong hai là hiện SC-10, không có chế độ quay không tiếng (SRS D-10). `RecordingConfig` không có cờ tắt tiếng: ghi hình luôn kèm tiếng.

**Đặc tả `PermissionService`**

- `ensureCameraAndMic()` trả về `granted`, `denied` hoặc `permanentlyDenied` cho từng quyền; chỉ gọi hệ thống hỏi sau khi đã hiện màn giải thích lý do.
- `canDrawOverlays()` (chỉ Android, giai đoạn 4): kiểm tra `Settings.canDrawOverlays`; `openOverlaySettings()` mở `ACTION_MANAGE_OVERLAY_PERMISSION` kèm gói của ứng dụng.
- `openAppSettings()`: mở trang cài đặt của ứng dụng; khi quay lại (`AppLifecycleState.resumed`) tự kiểm tra lại mọi quyền.
- Trên iOS, `permanentlyDenied` nghĩa là hệ thống sẽ không hỏi lại, nên SC-10 chỉ còn nút mở Cài đặt.
- `ensureGallerySave()`: chỉ gọi khi người dùng bấm Lưu vào bộ sưu tập. Android 10 trở lên ghi qua MediaStore nên thường không cần quyền; iOS xin quyền chỉ thêm bằng `PHPhotoLibrary.requestAuthorization(for: .addOnly)` (`NSPhotoLibraryAddUsageDescription`).
- `ensureGalleryRead()` (chỉ iOS, giai đoạn 3, FR-49b): chỉ gọi khi người dùng chọn Kiểm tra trạng thái; xin `.readWrite` (`NSPhotoLibraryUsageDescription`), chấp nhận cả `.limited`. Bị từ chối thì báo lỗi tại chỗ, video vẫn nằm trong ứng dụng.
- `ensureBluetoothConnect()` (chỉ Android 12 trở lên, giai đoạn 3, FR-41b): chỉ gọi khi người dùng chọn micro Bluetooth, sau lời giải thích; Android 11 trở xuống không cần xin lúc chạy. Bị từ chối thì mục Bluetooth trên SC-04 tắt kèm giải thích và nút mở Cài đặt, không dẫn tới SC-10 và không chặn quay (D-17). iOS không cần quyền riêng cho đường âm thanh HFP (xác nhận ở T0-08).
- `canPostNotifications()` và `requestNotifications()` (chỉ Android 13 trở lên, giai đoạn 4, dùng cho cửa sổ nổi): thiếu quyền này thì SC-10 hiện thêm hành động xin quyền thông báo, xem mục 8.

## 8. Cửa sổ nổi trên Android

Chế độ này thuộc giai đoạn 4 (task T4-01, FR-51) và chỉ có trên Android; iOS không cho vẽ lên trên ứng dụng khác. Người dùng dùng ứng dụng camera khác để quay, cửa sổ nổi chỉ hiện kịch bản.

- **Quyền:** khai báo `SYSTEM_ALERT_WINDOW`; người dùng cấp thủ công ở trang cài đặt (xem mục 7).
- **Service:** một foreground service giữ cửa sổ sống, kèm thông báo điều khiển với ba action (xem mục Điều khiển bên dưới). Từ Android 14 phải khai báo loại foreground service phù hợp; cần đối chiếu tài liệu Android hiện hành khi làm.
- **Cửa sổ:** thêm bằng `WindowManager` với loại `TYPE_APPLICATION_OVERLAY`, cờ FLAG_NOT_FOCUSABLE và FLAG_NOT_TOUCHABLE, nên cửa sổ chỉ hiển thị: không nhận chạm, không kéo thả, mọi thao tác chạm đi thẳng tới ứng dụng camera bên dưới.
- **Hiển thị:** Ưu tiên dùng một Flutter engine phụ trong service để tái dùng `ScrollEngine` và giao diện nhắc chữ (có thể thử gói flutter_overlay_window trước khi tự viết). Nếu không ổn, vẽ bằng View native với bộ cuộn riêng.
- **Điều khiển:** cửa sổ không có nút nào. Mọi điều khiển đi qua thông báo của foreground service với ba action: Tạm dừng/Tiếp tục, Tùy chỉnh, Đóng. Tùy chỉnh mở bảng điều khiển gọn SC-11 (activity nền trong suốt dạng bottom sheet) có vị trí (lưới 3×3 hoặc thanh trượt), kích thước, màu chữ, màu nền, độ đậm nhạt và tốc độ; đóng bảng thì quay lại ứng dụng camera. Thông báo Android chỉ hiện tối đa ba action nên tốc độ nằm trong bảng. Trạng thái đồng bộ giữa service và ứng dụng chính qua message port hoặc MethodChannel.
- **Giới hạn cần thử:** kiểm tra trên Android 12 trở lên rằng cửa sổ không nhận chạm thật sự cho thao tác đi xuyên xuống ứng dụng camera; một số hãng máy có trình quản lý pin tự tắt service nền. Spike trên ít nhất 3 hãng máy trước khi cam kết.
- **Cửa hàng:** Google Play yêu cầu giải thích mục đích dùng quyền này trong mô tả và chính sách quyền riêng tư (task T2-20, bổ sung khi làm T4-01).
- Video quay bởi ứng dụng khác không đi qua `Recordings`, nên không xuất hiện trong SC-09.
- **Quyền thông báo:** từ Android 13 cần quyền `POST_NOTIFICATIONS`. Thiếu quyền này thì không có điều khiển nào, nên xin cùng lúc với quyền hiển thị trên ứng dụng khác và chỉ bật chế độ cửa sổ nổi khi có đủ cả hai (bổ sung vào `PermissionService` và SC-10).
- **Luồng thao tác:** người dùng chạm action trên thông báo → service nhận `PendingIntent` → đổi trạng thái (tạm dừng/tiếp tục, đóng) hoặc mở SC-11; cài đặt đổi trong SC-11 được gửi sang service và cửa sổ cập nhật ngay, không khởi động lại.

## 9. Kiểm thử, CI/CD, cấu hình build

| Cấp | Phạm vi | Lệnh/công cụ |
| --- | --- | --- |
| Unit | `ScrollEngine.speedFor`, đếm từ, `PrompterMarkdown.parse` (từng cú pháp, ký hiệu không đóng, ký tự thoát, tiếng Việt, đo thời gian NFR-15), gom video theo ngày, repository (drift in-memory), `PermissionService` (mock) | `flutter test` |
| Widget | SC-02, SC-03 (tạm dừng, đổi tốc độ, lật chữ, dải đánh dấu dòng), SC-09 (chọn nhiều, trạng thái bộ sưu tập), SC-10 | `flutter test`, golden test cho SC-03 |
| Native | Hợp đồng Pigeon: gọi thử từng lệnh với bản giả | Test đơn vị Kotlin/Swift |
| Integration | UC-01 và UC-02 trên máy thật | `integration_test` |
| Hiệu năng | Cuộn theo NFR-01 (60 Hz và 120 Hz), quay 1080p/30 (NFR-03, NFR-04 trên máy Android 10 trở lên), 4K/30 (NFR-14), bitrate theo NFR-13, độ lệch hình–tiếng theo NFR-16 | `flutter run --profile`, DevTools, ffprobe |
| Thủ công | Cuộc gọi đến, xoay màn hình (cả khi đang ghi), từ chối quyền, sắp hết và hết bộ nhớ, cắm/rút micro ngoài và ngắt Bluetooth khi đang quay, rút thẻ nhớ | Danh sách kiểm trên thiết bị |

**CI** chạy trên mỗi pull request: `flutter pub get`, `dart format --set-exit-if-changed .`, `flutter analyze`, `flutter test --coverage`, build debug Android; build iOS khi có runner macOS. Độ phủ mục tiêu ≥ 70% cho lớp domain và application (NFR-11).

**Cấu hình build**

- Flavor `dev` và `prod` (applicationId/bundle id khác nhau).
- Android: `minSdk 26`, `targetSdk` và `compileSdk` 36 (Google Play yêu cầu target API 36 từ 2026-08-31, xác nhận trên Play Console); plugin Kotlin của app theo hướng dẫn chuyển sang Kotlin tích hợp của AGP 9 (xác nhận khi khởi tạo); quyền `CAMERA`, `RECORD_AUDIO`, `BLUETOOTH_CONNECT` (giai đoạn 3, Android 12 trở lên; thêm `BLUETOOTH` với `maxSdkVersion=30` nếu `startBluetoothSco` cần, xác nhận ở T0-08), `SYSTEM_ALERT_WINDOW` (giai đoạn 4, kèm POST_NOTIFICATIONS và quyền foreground service); lưu vào thư viện qua MediaStore, không xin quyền ghi bộ nhớ ngoài trên Android 10 trở lên.
- Thẻ nhớ (Android): ghi vào thư mục riêng của ứng dụng trên thẻ, không thêm quyền nào; thử trên máy có thẻ microSD và trên máy dùng bộ nhớ trong chia phân vùng (giả lập thẻ) để chắc bộ lọc ổ hoạt động.
- iOS: deployment target 16.0 (SRS D-08; Flutter 3.47 trở đi yêu cầu tối thiểu 15); build bằng Xcode có SDK iOS 26 (Apple yêu cầu cho bản tải lên từ 2026-04-28); Flutter 3.44 trở đi mặc định dùng Swift Package Manager, plugin chưa hỗ trợ thì dùng CocoaPods; `Info.plist` có `NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, `NSPhotoLibraryAddUsageDescription` (và `NSPhotoLibraryUsageDescription` từ giai đoạn 3, FR-49b) bằng tiếng Việt và Anh, nêu rõ lý do.
- Sao lưu: Android khai báo `android:dataExtractionRules` và `android:fullBackupContent` để loại thư mục video (NFR-07).
- Ký bản phát hành: keystore Android và chứng chỉ iOS lưu ở kho bí mật của CI, không đưa vào repo.
- Phát hành thử: TestFlight và Closed testing trên Google Play (task T2-21 cho MVP, T3-13 cho bản cập nhật).

**Trước khi merge một task**

- [ ] Đạt định nghĩa hoàn thành trong kế hoạch (review, test, máy thật, analyzer sạch).
- [ ] Mọi chuỗi hiển thị mới có cả bản vi và en.
- [ ] Không đọc pixel camera về Dart và không thêm bước xử lý khung hình.
- [ ] Đã xử lý trường hợp từ chối quyền và ngắt quãng nếu task dính đến camera.

## 10. Mở khóa Pro (giai đoạn 5, FR-70–FR-75)

Không có server và không dùng RevenueCat (SRS D-13): mọi việc đi qua plugin `in_app_purchase`, bọc StoreKit 2 (iOS) và Google Play Billing (Android).

- **`ProService`** (`features/pro/`): lấy thông tin sản phẩm và giá từ cửa hàng, gọi mua, nghe luồng cập nhật giao dịch, và cung cấp `proStatusProvider` (Notifier) cho cả ứng dụng. Tính năng Pro chỉ hỏi `proStatusProvider`, không gọi plugin trực tiếp.
- **iOS:** giao dịch đã được StoreKit 2 xác minh chữ ký trước khi trả về; khi mở app đọc quyền đã mua để khôi phục tự động; nút Khôi phục gọi `AppStore.sync()` rồi đọc lại. Giao dịch có `revocationDate` thì khóa lại; cần xác nhận ở T5-02 là plugin lộ được `revocationDate`, nếu không thì thêm cầu nối native nhỏ qua Pigeon.
- **Android:** kiểm chữ ký `originalJson` bằng khóa công khai Base64 của ứng dụng trên Play Console; acknowledge giao dịch ngay sau khi mua; khi mở app gọi `queryPurchasesAsync` để khôi phục. Giao dịch không còn trong kết quả thì chưa khóa ngay mà theo quy tắc ở mục Offline bên dưới (hai lần kiểm có mạng liên tiếp).
- **Offline và lỗi tạm thời:** lưu `ProStatus` (JSON trong shared_preferences, khóa `pro_status_v1`). Nếu cửa hàng trả danh sách rỗng mà lần xác minh gần nhất là Pro, giữ Pro và kiểm lại ở lần mở sau, chỉ khóa khi có giao dịch hoàn tiền rõ ràng hoặc hai lần kiểm có mạng liên tiếp đều không thấy giao dịch.
- **Thử nghiệm:** file cấu hình StoreKit cho Xcode, tài khoản thử Sandbox và tài khoản thử giấy phép trên Play Console; kịch bản thử gồm mua, chờ thanh toán, hủy, khôi phục trên máy mới, offline, hoàn tiền.
- **Cờ tính năng (SRS D-14):** FR-20, FR-46, FR-51, FR-52 được làm ở giai đoạn 4 sau cờ `proFeaturesEnabled`, tắt trong bản phát hành công khai; giai đoạn 5 bật cờ cùng lúc ra bản Pro. Cờ là hằng số lúc build, không đọc từ máy chủ.