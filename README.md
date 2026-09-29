# EKA2L1-S60-Hybrid-Home

> **Dự án thử nghiệm kiến trúc Hybrid Home cho Nokia 5800 / Symbian S60v5 trên EKA2L1.**  
> Mục tiêu là tái tạo lớp **Home / Shell** theo kiểu host-rendered UI, trong khi vẫn sử dụng **dịch vụ, AppArc, ứng dụng và backend Symbian thật** bên trong EKA2L1.

## 1. Mục tiêu dự án

Dự án này nghiên cứu một hướng đi song song với việc cố khởi động toàn bộ Home Screen nguyên bản của Nokia 5800.

Thay vì bắt buộc phải đưa toàn bộ chuỗi Home gốc của RM-356 hoạt động trước, kiến trúc Hybrid Home chia hệ thống thành hai lớp:

```text
┌──────────────────────────────────────┐
│ Hybrid S60 Home / Shell              │
│ Host-rendered UI trên iOS            │
│ - Home Screen                        │
│ - App launcher                       │
│ - Status / clock / shortcuts         │
└──────────────────┬───────────────────┘
                   │ bridge
                   ▼
┌──────────────────────────────────────┐
│ EKA2L1 + Symbian backend thật        │
│ - Menu3 thật                         │
│ - AppArc / app registry thật         │
│ - Window Server / input              │
│ - Central Repository                 │
│ - filesystem / services              │
│ - ứng dụng Symbian thật              │
└──────────────────────────────────────┘
```

Ý tưởng này gần với triết lý của các dự án tái tạo giao diện hệ điều hành cũ như **OldOS**: không nhất thiết phải boot toàn bộ hệ điều hành nguyên bản để tái hiện trải nghiệm sử dụng, nhưng các chức năng phía sau vẫn có thể nối vào backend thật.

## 2. Thiết bị và firmware mục tiêu

- **Thiết bị:** Nokia 5800 XpressMusic
- **Product code:** RM-356
- **Firmware đang dùng để thử nghiệm:** 60.0.003
- **Hệ điều hành:** Symbian S60 5th Edition
- **Frontend thử nghiệm chính:** EKA2L1 trên iOS
- **Thiết bị test hiện tại:** iPhone
- **Ngôn ngữ test ưu tiên:** Tiếng Việt

## 3. Baseline kỹ thuật

Nhánh Hybrid được xây dựng từ dòng tương thích đã chứng minh được Menu3 hoạt động ổn định:

- MENUUI36 WINFOCUS1
- MENUUI35 async input
- MENUUI33 key sound
- MENUUI31 raw pen input
- MENUUI30 stock FEP
- MANIC3
- NOJAVA

Menu3 thật của firmware:

```text
UID 0x101F4CD2
```

Home Screen thật của Nokia 5800:

```text
UID 0x102750F0
```

Hybrid Home **không bắt buộc phải khởi chạy Home UID 0x102750F0**.

## 4. Kiến trúc HYBRIDHOME1

Prototype đầu tiên chứng minh luồng:

```text
Menu3 thật
   ↓
Hybrid Home do frontend iOS dựng
   ↓
bridge::get_apps()
   ↓
danh sách ứng dụng đã đăng ký thật trong EKA2L1
   ↓
launchAppUid(uid)
   ↓
bridge::launch_app(uid)
   ↓
ứng dụng Symbian thật
```

### Điều Hybrid Home giữ lại từ emulator

Hybrid Home không phải một mock app độc lập. Mục tiêu là dùng lại càng nhiều backend thật càng tốt:

- App registry / AppArc
- UID ứng dụng
- launcher EKA2L1
- filesystem của firmware
- Window Server
- input pipeline
- Central Repository
- các service Symbian mà ứng dụng thực sự cần
- tài nguyên và metadata lấy từ RM-356 khi khả thi

### Điều Hybrid Home thay thế

Ở giai đoạn đầu, lớp giao diện Home được dựng ở phía host để tránh phụ thuộc vào toàn bộ startup chain của Home gốc:

- AISCUT
- WidgetRegistry
- Home plugins
- TfxServer / transition dependencies
- các plugin hoặc service phụ chỉ cần để Home gốc khởi động

Đây là **đường kiến trúc song song**, không phải xoá bỏ hướng nghiên cứu Home thật.

## 5. HYBRIDHOME1 — trạng thái hiện tại

HYBRIDHOME1 đã build thành công trên GitHub Actions.

- **Build run:** `36600699450`
- **Kết quả:** GREEN / success
- **Build host commit:** `7325f43f9ef6f9079f20636ec2991138ac84ad06`
- **Source prototype commit:** `83e1326dd25bedec0b8d6d20435484b0ee965b2a`
- **IPA artifact:** `EKA2L1-HYBRIDHOME1-OLDOS-SHELL-IPA`
- **Artifact ID:** `11048733884`
- **IPA:** `EKA2L1-HYBRIDHOME1-OLDOS-SHELL-unsigned.ipa`
- **SHA-256:** `101b3bdbd1d9d56e71126e053d03d720beed11f58afcb3481ae957ba564ad23f`

Prototype hiện có hai thao tác chính:

1. **Trở về Menu3 thật** — gỡ lớp Hybrid Home và quay lại guest Menu3 đang chạy mà không reboot emulator.
2. **Ứng dụng Symbian thật** — đọc app registry của EKA2L1 và khởi chạy ứng dụng Symbian qua launcher thật.

## 6. Tiêu chí PASS HYBRIDHOME1

Prototype được xem là PASS kiến trúc khi thiết bị thật chứng minh được toàn bộ chuỗi:

```text
Menu3 thật
→ Hybrid Home
→ trở lại Menu3 thật
→ Hybrid Home
→ danh sách app thật
→ ứng dụng Symbian thật chạy
```

Các marker log chính:

```text
[HYBRIDHOME1][TRIGGER]
[HYBRIDHOME1][SHOW]
[HYBRIDHOME1][RETURN_MENU3]
[HYBRIDHOME1][APP_CHOOSER]
[HYBRIDHOME1][LAUNCH_REAL_APP]
```

## 7. Lộ trình

### HYBRIDHOME2 — giao diện Nokia 5800

Sau khi HYBRIDHOME1 PASS trên thiết bị:

- dựng bố cục Home gần Nokia 5800 hơn;
- lấy icon ứng dụng thật;
- đọc tên app từ AppArc;
- đưa wallpaper/theme từ firmware vào shell;
- status bar, đồng hồ, pin, sóng;
- shortcut Telephone / Contacts / Messaging;
- portrait 360×640 theo màn hình Nokia 5800.

### HYBRIDHOME3 — resource-driven shell

- đọc resource từ firmware thay vì hardcode;
- ánh xạ shortcut bằng UID / CenRep;
- launcher dạng grid/list;
- trạng thái app đang chạy;
- chuyển qua lại Home ↔ app mà không reboot emulator.

### HYBRIDHOME4+ — mở rộng Shell

Mục tiêu dài hạn là có một lớp S60 Shell đủ dùng để:

- chạy ứng dụng và game Symbian;
- sử dụng Messaging, Contacts, Phone và các system app khi backend hỗ trợ;
- tái hiện trải nghiệm Nokia 5800;
- giảm phụ thuộc vào các thành phần Home gốc mà EKA2L1 chưa mô phỏng đầy đủ.

## 8. Quan hệ với các nhánh EKA2L1 khác

Dự án này được tách riêng để không làm ảnh hưởng các hướng nghiên cứu hiện tại.

### HOMEONLY

HOMEONLY tiếp tục mục tiêu đưa **Home Screen thật của Nokia 5800** chạy đúng trong EKA2L1.

Các nghiên cứu gần đây đã xử lý hoặc chẩn đoán:

- AISCUT USER 11 / descriptor overflow;
- CenRep quoted tokenizer;
- WidgetRegistry / AppList non-native registration;
- MMF audio Publish & Subscribe;
- input/event wake của Home.

### NativeBoot / CompatBoot

Hybrid Home **không dựa vào NativeBoot hoặc CompatBoot**.

Mục tiêu là bắt đầu từ guest environment đã ổn định, đặc biệt là Menu3, rồi dựng lớp Shell mới ở phía trên.

## 9. Nguyên tắc phát triển

1. **Không giả lập lại những gì EKA2L1 đã làm được.** Nếu backend Symbian thật hoạt động thì dùng backend thật.
2. **Host UI chỉ thay phần Shell/Home cần thiết.**
3. **Không hardcode firmware-specific data nếu có thể đọc từ ROM/RPKG/CenRep/AppArc.**
4. **Mỗi thay đổi phải có marker log rõ ràng.**
5. **Giữ đường quay lại Menu3 để đối chiếu.**
6. **Không làm hỏng baseline MENUUI36 / input đã ổn định.**
7. **Ưu tiên incremental build và device test nhỏ, có checkpoint.**

## 10. Scope / Safety clarification

Dự án này chỉ phục vụ:

- giả lập và bảo tồn phần mềm Symbian;
- nghiên cứu khả năng tương thích EKA2L1;
- tái tạo giao diện Nokia/S60;
- chạy firmware và ứng dụng trên emulator.

Dự án **không liên quan đến**:

- khai thác hệ thống;
- xâm nhập thiết bị;
- malware;
- đánh cắp credential;
- persistence;
- bypass bảo mật;
- tấn công mạng.

Mọi patch trong repo phải phục vụ trực tiếp cho emulator compatibility, UI reconstruction hoặc preservation.

## 11. Nguồn liên quan

- Upstream EKA2L1: `EKA2L1/EKA2L1`
- Nhánh nghiên cứu Menu3/Home trước đây: `phai-nguyen/Eka2l1-nhanh-menuu3`
- Build host lịch sử: `phai-nguyen/Eka2l1_bot_menu_simbiam`
- Ý tưởng tham khảo kiến trúc UI recreation: `zzanehip/the-oldos-project`

---

**Trạng thái:** 🟢 HYBRIDHOME1 build GREEN — đang chờ device test để xác nhận kiến trúc trên iPhone.
