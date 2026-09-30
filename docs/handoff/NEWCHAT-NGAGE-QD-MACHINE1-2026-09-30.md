# NEW CHAT HANDOFF — EKA2L1 iOS / HYBRIDHOME3 / VPL / N-Gage QD MACHINE1

**Thời điểm chốt:** 2026-09-30 16:20 (+07:00)

## 0. Mục tiêu của HANDOFF này

Tài liệu này chốt toàn bộ trạng thái còn đọc được của cuộc trò chuyện hiện tại để có thể mở chat mới và tiếp tục ngay, không nghiên cứu lại từ đầu.

Có hai nhánh công việc đang tồn tại song song:

1. **Nhánh sản phẩm EKA2L1 iOS hiện tại**
   - phát triển trực tiếp từ official `EKA2L1/EKA2L1`;
   - Hybrid Home riêng cho Nokia 5800 RM-356;
   - VPL firmware importer;
   - tiếng Việt;
   - ESign-compatible;
   - JIT/Dynarmic tắt trên device của user.

2. **Nhánh nghiên cứu mới**
   - lấy cảm hứng từ CloudpilotEmu full-system/hardware-oriented emulation;
   - không bắt đầu bằng RM-356 vì quá phức tạp;
   - target đầu tiên đã chốt: **Nokia N-Gage QD RH-29 / S60v1 / EKA1**;
   - file `SYM.ROM` cho RH-29 V04.10 đã được dựng trong chat này;
   - **chưa device-test file ROM vừa dựng**;
   - **chưa tạo branch/repo `NGAGE-MACHINE1`**.

---

# 1. Scope / safety clarification

Đây là dự án **emulator compatibility / system emulation** cho Symbian/Nokia firmware.

Mục tiêu:
- nghiên cứu kiến trúc máy;
- cài và chạy firmware hợp lệ trong EKA2L1;
- chạy guest applications;
- mô phỏng phần cứng/phần mềm guest cần thiết.

Không phải dự án:
- credential theft;
- malware;
- persistence trên hệ thống người khác;
- network exploitation;
- xâm nhập hệ thống;
- bypass bảo mật ngoài phạm vi tương thích emulator.

---

# 2. User environment cần nhớ

- Ngôn ngữ: **Tiếng Việt**.
- Thiết bị test chính: **iPhone 12 Pro Max (iPhone13,4), iOS 18.7**.
- Sideload/sign: **ESign**.
- PC phụ: Windows 7 64-bit, RAM 2 GB, không có Mac.
- User thích test thực tế trên iPhone rồi gửi log/video.
- Không gọi một bản là DEVICE PASS nếu user chưa test thực tế.

---

# 3. Repo / branch hiện tại

## 3.1 Build/controller repo

`phai-nguyen/-EKA2L1-iOS-fixed`

Active branch:

`hybridhome-upstream1`

Official upstream pinned baseline:

`EKA2L1/EKA2L1@c5ae62d9c667f6e7060d76be58a2614122a629a4`

Nguyên tắc:

> upstream chạy được cái gì thì không viết lại cái đó.

Workflow:

`.github/workflows/build-hybridhome-upstream1.yml`

## 3.2 Reference/spec repo

`phai-nguyen/EKA2L1-S60-Hybrid-Home`

Tài liệu kiến trúc:
- `docs/ARCHITECTURE.md`
- `docs/research/UPSTREAM-IOS-BASELINE-2026-09-30.md`
- handoff hiện tại này.

## 3.3 Repo cũ / POC

Các repo/nhánh NativeBoot/Menu3 cũ vẫn là reference, không phải baseline mới:
- `phai-nguyen/Eka2l1_bot_menu_simbiam`
- `phai-nguyen/Eka2l1-nhanh-menuu3`
- DirectHome / CompatBoot / HOMEONLY / MENUUI.

Không quay lại các nhánh này trừ khi cần lấy bằng chứng/ý tưởng.

---

# 4. Ràng buộc iOS đã xác nhận

## 4.1 JIT / Dynarmic phải OFF cho ESign

Crash iOS trước đây xảy ra ngay trong JIT probe:
- `host_can_jit()`
- executable page / CODESIGNING
- `EXC_BAD_ACCESS` / invalid page

Baseline bắt buộc:

```text
EKA2L1_IOS_DYNARMIC=OFF
```

Backend device hiện dùng Dyncom.

## 4.2 Bundle ID phải ESign-match để Files picker hoạt động đúng

Bundle ID cần giữ:

```text
app.lavender1865.valley8348
```

Bundle ID upstream mặc định `com.eka2l1.emulator` từng mở Files được nhưng không trả usable URL đúng trong cấu hình ESign của user.

---

# 5. HYBRIDHOME hiện tại

## 5.1 Kiến trúc

```text
RM-356 firmware
      ↓
official EKA2L1 core/services
      ↓
AppList / AppArc / icon registry
      ↓
EKA2L1Bridge
      ↓
SwiftUI Hybrid Home
      ↓
real Symbian app
```

Nokia Home thật UID `0x102750F0` không bắt buộc.

Hybrid Home chỉ kích hoạt khi:

```swift
firmwareCode.caseInsensitiveCompare("RM-356") == .orderedSame
```

Các máy khác tiếp tục dùng upstream app-grid bình thường.

## 5.2 UID RM-356 đã xác nhận từ log thật

- Nokia Home: `0x102750F0`
- Menu: `0x101F4CD2`
- Calculator: `0x10005902`
- Telephone: `0x100058B3`
- Contacts: `0x101F4CCE`
- Messaging: `0x100058C5`

## 5.3 HYBRIDHOME3

Commit:

`acb2775a652f44104c841a6ed08557801a3aadbe`

Message:

`feat(hybridhome): add localized searchable RM-356 launcher`

Tính năng:
- Home/Menu shell host-rendered;
- icon + app name thật từ AppList/AppArc;
- shortcut Telephone/Contacts/Messaging bằng UID thật;
- search theo tên app;
- search theo UID dạng `0x...`;
- nút refresh AppList;
- no-results state;
- Hybrid Home strings đi qua localization, không còn hard-code tiếng Việt;
- launch qua:
  `NavigationLink(destination: EmulatorView(uid: app.uid))`;
- exit/app dismissal dựa trên upstream `EmulatorView` behavior.

Log marker đã đổi sang `HYBRIDHOME3`.

## 5.4 Build HYBRIDHOME3 mới nhất

GitHub Actions:

`#36663115510`

Status:

**GREEN / success**

HEAD:

`acb2775a652f44104c841a6ed08557801a3aadbe`

Artifact:
- name: `EKA2L1-HYBRIDHOME-UPSTREAM1-IPA`
- artifact ID: `11074824884`
- size: `10,062,898` bytes
- digest:
  `sha256:7114802b238fe90cf5b16e9290952c966b97a5c1c4574185112785504de6458e`

Run URL:

https://github.com/phai-nguyen/-EKA2L1-iOS-fixed/actions/runs/36663115510

**Lưu ý:** build GREEN, chưa ghi nhận device test riêng cho HYBRIDHOME3 trong đoạn hội thoại sau build.

---

# 6. Vietnamese localization

Đã hoàn thiện merge tiếng Việt lên current upstream catalogs.

Reference files/controller:
- `localization/Localizable.vi-reference.xcstrings`
- `localization/InfoPlist.vi-reference.xcstrings`
- `scripts/apply_vietnamese_ios.py`

User đã xác nhận UI tiếng Việt hoạt động.

Build localization GREEN trước đó:

`#36650452787`

Commit:

`4568a376395f3fc89586fc66278a2fc1b412286b`

Coverage mục tiêu:
- Localizable: 245/245
- InfoPlist: 5/5

HYBRIDHOME3 thêm localized strings cho:
- shortcuts;
- menu title;
- app count;
- Home;
- Menu;
- Phone;
- Contacts;
- search;
- no results;
- refresh.

---

# 7. Build optimization hiện tại

Đã có:
- Xcode incremental build cache;
- ccache với path đúng;
- shallow upstream checkout;
- FFmpeg cache.

Mốc trước:
- full compile ~30+ phút;
- sau cache: ~10–11 phút total;
- build step khoảng ~8 phút.

Không xóa cache workflow nếu không có lý do.

---

# 8. VPL firmware importer — trạng thái DEVICE PASS

User yêu cầu thêm lại chức năng Firmware VPL như bản EKA trước đây.

Đã triển khai vào current official-iOS-based app.

## 8.1 UI

Trong Install Device có:
- loose ROM/RPKG;
- archive;
- **Firmware VPL**.

VPL có 2 cách:
1. Chọn cả thư mục firmware;
2. Chọn nhiều file firmware cùng lúc.

Flow multi-file:
- security-scoped URLs;
- copy vào temp staging;
- tìm `.vpl`;
- gọi bridge;
- bridge gọi core `eka2l1::install_firmware(...)`.

Không reimplement parser firmware ở Swift.

## 8.2 Commit VPL ban đầu

`5c5f0224e5112e4eaf8461f9fc579a33eee3a8fa`

`feat(ios): restore VPL firmware import flow`

Build:

`#36660104360` — GREEN.

## 8.3 Crash device ban đầu

Khi user chọn nhiều file VPL, app crash ngay.

Hai crash logs cùng root cause:

```text
EXC_BREAKPOINT
SIGTRAP
_dispatch_assert_queue_fail
swift_task_isCurrentExecutorWithFlagsImpl
ImportDeviceView.findVPL(in:)
ImportDeviceView.install()
```

Nguyên nhân:
- helper static nằm trong SwiftUI View;
- Swift 6 main-actor isolation;
- helper được gọi từ background queue.

## 8.4 Fix

Hai helper thành:

```swift
nonisolated private static func findVPL(...)
nonisolated private static func stageFirmwareFiles(...)
```

Commit fix:

`8fff5a38202a253009ad35f8a15217cde0e94ae5`

CI guard:

`62bf761c44fb52111206e19a2d1a4a1b3fbb6fa7`

Build:

`#36661834297` — GREEN.

## 8.5 Device test result

User xác nhận bản sửa hoạt động.

Video cho thấy:
- Firmware VPL;
- chọn 8 file RM-612;
- install progress chạy;
- app không crash;
- C6-00/RM-612 xuất hiện trong device switcher;
- AppList/icons thật đọc được.

Chốt:

**VPL multi-file import: DEVICE PASS**

**VPL no-crash fix: DEVICE PASS**

Non-RM-356 như C6-00 vẫn dùng upstream app-grid, không dùng Hybrid Home.

---

# 9. CloudpilotEmu research — kết luận cần giữ

Repo nghiên cứu:

`cloudpilot-emu/cloudpilot-emu`

Kết luận:
- Cloudpilot không chỉ reimplement Palm APIs;
- Palm OS 1–4 dựa trên POSE/DragonBall-oriented machine emulation;
- Palm OS 5/Tungsten E2 dùng uARM với SoC/device model rất sâu;
- họ dựng CPU/MMU/RAM/ROM/MMIO/IRQ/timer/DMA/GPIO/LCD/storage/audio/touch/UART/etc.;
- ROM thật chạy trên machine model;
- nhưng họ không cố 100% transistor-accurate:
  - vẫn có patch;
  - stubs;
  - selective interception;
  - paravirtualized components/platform.

Bài học quan trọng:

```text
device-specific machine
+
real ROM execution
+
accurate-enough MMIO/timing
+
selective hardware emulation
+
targeted patches/paravirtualization
```

Đây là hướng kiến trúc đáng học cho một nhánh full-system Symbian.

Không nên port code Palm/PXA trực tiếp sang Nokia; chỉ học cấu trúc machine model.

---

# 10. Quyết định chiến lược mới: không bắt đầu full-machine bằng RM-356

User nhận định Nokia 5800 quá phức tạp và muốn chọn S60 đơn giản hơn.

Target đã chốt:

# **Nokia N-Gage QD — RH-29**

Lý do:
- S60 1st Edition;
- Symbian OS 6.1 / EKA1;
- ARM9/ARM920T-class target đơn giản hơn ARM11 RM-356;
- màn 176×208;
- không touch;
- không camera;
- ít startup stack/platform-security complexity hơn S60v5;
- EKA2L1 đã có nhiều support/compatibility riêng cho N-Gage;
- EKA2L1 device map đã biết:
  - NEM-4 = original N-Gage;
  - RH-29 = N-Gage QD.

Thứ tự nghiên cứu dự kiến:

```text
RH-29 N-Gage QD
→ NEM-4 original N-Gage
→ N70 / S60v2
→ S60v3
→ cuối cùng mới cân nhắc quay lại RM-356 full-machine
```

Hybrid Home RM-356 không bị bỏ; nó vẫn là nhánh sản phẩm chạy được.

---

# 11. Firmware RH-29 user vừa cung cấp

User upload:

`Nokia_N-Gage_QD_RH-29.zip`

Runtime path trong chat hiện tại:

`/mnt/data/Nokia_N-Gage_QD_RH-29.zip`

**Không giả định path này còn tồn tại ở chat mới. Nếu cần bytes trong chat mới, retrieve attachment/Library hoặc yêu cầu re-upload.**

SHA-256 original ZIP:

`6d2e365d7afb343415e81656971e4007ad34bc001bde3745336f7c964a8a1c4f`

ZIP chứa:

```text
Nokia_N-Gage_QD_RH-29/Firmware/RH2941.026
  size: 1,386,311

Nokia_N-Gage_QD_RH-29/Firmware/RH2941.0C1
  size: 18,454,871
```

Ngoài ra có Credits/URL/Driver links.

---

# 12. SYM.ROM RH-29 đã dựng trong chat này

Từ firmware package trên, trong chat đã dựng được:

`SYM.ROM`

Runtime path hiện tại:

`/mnt/data/SYM.ROM`

Kết quả đã chốt:
- target: Nokia N-Gage QD RH-29;
- firmware string: **V 04.10 — 09-09-2004**;
- EKA1 ROM base:
  `0x50000000`;
- ROM size:
  `0x1170000` = `18,284,544` bytes;
- root directory:
  `0x50311000`;
- structural traversal:
  - 97 directories;
  - 1060 files;
  - không thấy file/directory pointer vượt ROM trong kiểm tra đã làm.

Hashes:

MD5:

`ba9ae3d584c42b24d44bf0f25ba5896f`

SHA-256:

`4824ee55086cade3415c7478ec54b9654658bfdd737e779995dfe0ef716ab70b`

File generated ZIP:

`RH-29_QD_V04.10_SYM-ROM.zip`

Runtime path:

`/mnt/data/RH-29_QD_V04.10_SYM-ROM.zip`

SHA-256 ZIP:

`3dc38659252ad35d11500a7ec8078e4f6eaf42d2f5a29ec3e5e1e522ee612230`

ZIP chứa:
- `SYM.ROM` — 18,284,544 bytes;
- `README.txt`.

## 12.1 Quan trọng: chưa DEVICE PASS

File SYM.ROM này mới:
- được dựng;
- kiểm tra cấu trúc;
- hash;
- tree traversal.

**Chưa được user cài/test trong EKA2L1 iOS.**

Bước đầu tiên của chat mới phải là test file này trước khi dùng nó làm baseline MACHINE1.

---

# 13. GitHub N-Gage SYM.ROM user đưa link

User đưa:

https://github.com/Abdess/retrobios/blob/main/bios/Nokia/N-Gage/SYM.ROM

Kết luận trong chat:
- path repo là `Nokia/N-Gage`;
- đây được coi là reference cho **N-Gage original NEM-4**, không phải RH-29 QD baseline;
- vì user đã có RH-29 firmware và đã dựng được RH-29 SYM.ROM, không ưu tiên dùng file GitHub đó cho MACHINE1.

Không trộn NEM-4 ROM với RH-29 device profile.

---

# 14. EKA2L1 ROM facts đã xác nhận từ source

Current EKA2L1 loader định nghĩa EKA1 ROM base:

```cpp
static constexpr std::uint32_t EKA1_ROM_BASE = 0x50000000;
```

`rom_header` đầu ROM gồm:
- 124-byte jump;
- restart_vector;
- time/version;
- rom_base;
- rom_size;
- rom_root_dir_list;
- kern_data_address;
- kern_limit;
- primary/secondary file;
- hardware/screen/bpp info cho EKA1;
- v.v.

EKA2L1 `load_rom()`:
- đọc header;
- seek theo `rom_root_dir_list - rom_base`;
- traverse burn-tree/root dirs;
- reject nếu không có root directory.

Đây là cơ sở để kiểm tra SYM.ROM RH-29.

---

# 15. N-Gage MACHINE1 — kiến trúc dự kiến

Không được nhảy thẳng tới full phone.

## MACHINE1 / Phase 1

Chỉ dựng:

```text
ARM920T-class CPU
↓
ROM map
↓
RAM
↓
reset/restart vector
↓
unknown-MMIO tracer
```

Mục tiêu:
- ROM bắt đầu thực thi từ đường boot thật;
- trace mọi MMIO access chưa mô hình hóa;
- không cần Home/Menu/LCD ngay.

## Phase 2

```text
timer
+
interrupt controller
+
IRQ/FIQ
```

Mục tiêu:
- ROM đi qua early boot phụ thuộc tick/interrupt.

## Phase 3

LCD/framebuffer tối thiểu.

Mục tiêu:
- thấy framebuffer thay đổi;
- sau đó mới kỳ vọng Nokia logo/startup image.

## Phase 4

Keypad:
- d-pad;
- softkeys;
- numeric keys;
- power;
- game buttons.

Không phải xử lý touch.

## Phase 5

Storage tối thiểu:
- ROM/flash behavior cần thiết;
- MMC presence nếu boot cần;
- GSM/Bluetooth/audio ban đầu có thể stub hoặc defer.

Triết lý Cloudpilot áp dụng:

```text
emulate boot-critical hardware first
+
stub/paravirtualize peripheral không cần cho boot
```

---

# 16. Điều chưa biết / không được tự giả định

Chưa xác nhận:
- SYM.ROM RH-29 vừa dựng được EKA2L1 iOS nhận đúng device;
- app list của RH-29 từ ROM này có chạy;
- full-machine cần chính xác SoC/peripheral map nào;
- reset vector/boot section trong SYM.ROM này có đủ để machine boot hay cần BOOT/ROOT dumps riêng;
- memory map phần cứng RH-29 ngoài EKA1 ROM base;
- interrupt controller/timer/GPIO register addresses;
- LCD controller model;
- flash/MMC electrical/register behavior.

Đừng bịa các địa chỉ MMIO.

Phải lấy bằng chứng từ:
- firmware/binary;
- service manuals/schematics;
- public Nokia DCT4/WD2 research;
- source emulators/tools;
- trace thực nghiệm.

---

# 17. Bước tiếp theo khi mở chat mới

## Bước A — test SYM.ROM trước

Dùng current iOS EKA2L1 build và thử install:

`SYM.ROM`

PASS tối thiểu:

```text
ROM được import
→ device được nhận
→ firmware code/device hợp lý
→ AppList scan được
→ ít nhất một app guest mở được
```

Nếu fail:
- lấy log iOS;
- xác định loader parse / product code / ROM layout issue;
- không nhảy ngay sang machine-emulation.

## Bước B — nếu ROM pass

Tạo nhánh nghiên cứu riêng, ví dụ:

`ngage-machine1`

Không sửa phá `hybridhome-upstream1`.

Có thể đặt branch trong controller repo hoặc tạo repo riêng sau khi có PoC.

## Bước C — machine probe tối thiểu

Implement/prototype:
1. ROM map `0x50000000`;
2. CPU state/reset entry;
3. RAM region;
4. fetch/execute trace;
5. log unmapped memory/MMIO reads/writes;
6. stop after bounded instruction count;
7. dump PC/LR/CPSR/registers khi fault.

Mục tiêu của MACHINE1 đầu tiên không phải UI.

Mục tiêu chỉ là trả lời:

> ROM RH-29 chạm vào hardware/MMIO nào trong vài nghìn/vài triệu instruction đầu?

---

# 18. Không được làm mất các mốc đã PASS

Giữ nguyên:
- ESign Bundle ID fix;
- JIT OFF;
- Vietnamese localization;
- VPL importer;
- VPL multi-file DEVICE PASS;
- build cache;
- Hybrid Home RM-356 architecture;
- official upstream pin/controller workflow.

N-Gage full-machine là **research branch song song**, không thay thế app hiện tại.

---

# 19. Câu lệnh mở chat mới

Dùng nguyên văn:

```text
Tiếp tục dự án EKA2L1 từ docs/handoff/NEWCHAT-NGAGE-QD-MACHINE1-2026-09-30.md trong repo phai-nguyen/EKA2L1-S60-Hybrid-Home. Giữ nguyên nhánh sản phẩm hybridhome-upstream1 và các mốc DEVICE PASS hiện có. Hướng nghiên cứu mới là Nokia N-Gage QD RH-29 full-machine theo mô hình Cloudpilot. SYM.ROM RH-29 V04.10 đã được dựng nhưng CHƯA device-test. Việc đầu tiên là kiểm tra/test SYM.ROM đó trong current EKA2L1 iOS; nếu pass mới tạo nhánh ngage-machine1 và bắt đầu ROM/RAM/reset/MMIO probe tối thiểu. Không quay lại RM-356 full-hardware ở giai đoạn này.
```

---

# 20. Trạng thái cuối cùng

**Nhánh sản phẩm:**
- HYBRIDHOME3: build GREEN;
- VPL: DEVICE PASS;
- Vietnamese: working;
- ESign/JIT constraints solved.

**Nhánh nghiên cứu:**
- Cloudpilot architecture đã nghiên cứu;
- đã chọn N-Gage QD RH-29;
- user đã tìm được firmware package;
- đã dựng được RH-29 V04.10 `SYM.ROM`;
- ROM mới **chưa device-test**;
- MACHINE1 branch **chưa tạo**.

Đây là checkpoint cuối cùng còn đọc được của cuộc trò chuyện.
