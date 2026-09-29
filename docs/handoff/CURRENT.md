# CURRENT — HYBRIDHOME1

**Ngày:** 2026-09-29

## Trạng thái

HYBRIDHOME1 đã build GREEN và đang chờ device test.

### Build

- Run: `36600699450`
- Build host HEAD: `7325f43f9ef6f9079f20636ec2991138ac84ad06`
- Source prototype HEAD: `83e1326dd25bedec0b8d6d20435484b0ee965b2a`
- Artifact: `EKA2L1-HYBRIDHOME1-OLDOS-SHELL-IPA`
- Artifact ID: `11048733884`
- IPA SHA-256: `101b3bdbd1d9d56e71126e053d03d720beed11f58afcb3481ae957ba564ad23f`

## Architecture under test

```text
Menu3 real
→ host-rendered Hybrid Home
→ bridge::get_apps()
→ existing EKA2L1 launcher
→ real Symbian app
```

Real Nokia Home UID `0x102750F0` is not required for this test.

## Device test

1. Vào Menu3.
2. Mở Menu trò chơi.
3. Chọn **Hybrid Home (thử nghiệm)**.
4. Xác nhận Hybrid Home hiển thị.
5. Chọn **Trở về Menu3 thật**.
6. Mở Hybrid Home lần nữa.
7. Chọn **Ứng dụng Symbian thật**.
8. Chọn một ứng dụng.
9. Xác nhận ứng dụng guest thật chạy.

## PASS

```text
Menu3 real
→ Hybrid Home
→ Menu3 real
→ Hybrid Home
→ real app list
→ real Symbian app
```

## Log markers

```text
[HYBRIDHOME1][TRIGGER]
[HYBRIDHOME1][SHOW]
[HYBRIDHOME1][RETURN_MENU3]
[HYBRIDHOME1][APP_CHOOSER]
[HYBRIDHOME1][LAUNCH_REAL_APP]
```

## Next

Nếu device test PASS, bắt đầu HYBRIDHOME2:

- app icons thật;
- wallpaper Nokia 5800;
- S60-style status bar;
- Telephone / Contacts / Messaging shortcuts;
- layout 360×640;
- bắt đầu chuyển dữ liệu UI từ hardcode sang resource/firmware-driven.
