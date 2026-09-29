# Upstream iOS baseline — 2026-09-30

## Quyết định kiến trúc

Từ thời điểm này, **EKA2L1/EKA2L1 upstream** là codebase chính để phát triển Hybrid Home.

Repo `phai-nguyen/EKA2L1-S60-Hybrid-Home` được giữ lại làm **research/specification/reference repository**, không còn là nơi chứa toàn bộ source emulator chính.

## Upstream baseline đã kiểm tra

- Repository: `EKA2L1/EKA2L1`
- Branch: `master`
- Baseline commit đã đọc: `c5ae62d9c667f6e7060d76be58a2614122a629a4`
- iOS frontend: SwiftUI + Objective-C++ bridge
- Core bridge:
  - `DeviceStore.swift`
  - `EKA2L1Bridge.swift`
  - `IosEmulator.mm`
- App registry:
  - `applist_server`
  - `rescan_registries()`
  - UID / caption / system-app classification từ registration thật
- Launch path:
  - `launchApp(uid:)`
  - `launchAppWithUID:completion:`
  - `applist_server::launch_app(...)`
- Icon path:
  - `iconPNGData(uid:sizePx:)`
  - `.mif`, `.mbm` và bitwise icon fallback
- System app classification:
  - app từ drive Z được đánh dấu built-in/system app.

## IPA đã device-test

File tham chiếu do người dùng cung cấp:

`EKA2L1-26.9.2-AppStore-VI-ESignMatch-unsigned.ipa`

Thông tin đã xác minh:

- Version: `26.9.2`
- Build: `260972`
- Architecture: `arm64`
- Bundle ID: `app.lavender1865.valley8348`
- MinimumOSVersion trong IPA: `18.0`
- SHA-256 IPA:
  `7db641a3107049dd3e5ee1c48fbce55df1cfab338216eb53319f7ca009d6b0b5`
- Binary SHA-256:
  `738b02a62548565a5b1934a2df9f04d19d1bf2b5bbca56705fb11b25587bf28f`

Binary chứa trực tiếp các symbol/selector quan trọng:

```text
EKA2L1Bridge
rescanApps
launchAppWithUID:completion:
runLaunchAppWithUID:
iconPNGDataForUID:sizePx:
home.showSystemApps
home.hideSystemApps
```

Người dùng đã test thực tế bản này với firmware Nokia 5800 RM-356 và xác nhận **nhiều ứng dụng hệ thống chạy được dù Home Screen nguyên bản không boot được**.

## Phát hiện quan trọng

Upstream đã có đúng mô hình cần cho Hybrid Home:

```text
RM-356 firmware
    ↓
EKA2L1 kernel + HLE services
    ↓
AppList/AppArc registration
    ↓
SwiftUI frontend
    ↓
UID + caption + icon thật
    ↓
launch_app()
    ↓
Symbian application thật
```

Do đó Hybrid Home không cần kéo theo toàn bộ HOMEONLY/MENUUI/NativeBoot/CompatBoot stack cũ.

## HYBRIDHOME1 mới

Mục tiêu proof đầu tiên trên upstream:

```text
Boot RM-356 backend
→ không yêu cầu Nokia Home UID 0x102750F0
→ hiển thị S60 Hybrid Home ở host
→ đọc app registry thật
→ hiển thị tên + icon thật
→ launch Calculator hoặc system app thật
→ app exit
→ quay lại Hybrid Home
```

## Thành phần upstream cần giữ nguyên ban đầu

- Device installation / boot
- DeviceStore
- EKA2L1Bridge
- AppList / AppArc
- icon decoding
- Window Server
- FBS
- CenRep
- View server
- MSV
- Etel
- MMF/audio
- input/touch
- app-exit lifecycle
- `needs_reboot_before_launch` workaround

Không tối ưu hoặc xoá workaround lifecycle trước khi HYBRIDHOME1 device-PASS.

## Phần sẽ phát triển

Ưu tiên sửa frontend, không sửa emulator core nếu không bắt buộc:

```text
src/emu/ios/App/
  S60HybridHomeView.swift
  S60StatusBar.swift
  S60AppGrid.swift
  S60ShortcutBar.swift
  S60SystemAppIcon.swift
```

Có thể chỉnh `ContentView.swift` để chọn Hybrid Home làm home surface của RM-356.

## Repo cũ

Các nhánh HOMEONLY / MENUUI / HYBRIDHOME1 cũ chỉ còn vai trò:

- reverse-engineering reference;
- evidence về RM-356;
- fallback nếu upstream thiếu một compatibility patch cụ thể;
- so sánh log;
- proof-of-concept Host Home + guest app.

Không bulk-port các patch cũ sang upstream.

## Scope

Dự án chỉ phục vụ emulator compatibility, Symbian preservation và UI reconstruction. Không nhằm tấn công mạng, xâm nhập hệ thống, malware, credential theft, persistence trái phép hoặc bypass bảo mật ngoài phạm vi emulator.
