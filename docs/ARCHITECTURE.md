# Architecture — EKA2L1 S60 Hybrid Home

## Mô hình

Hybrid Home chia hệ thống thành hai phần:

### Host Shell

Chạy ở frontend iOS và chịu trách nhiệm cho phần trải nghiệm người dùng:

- Home Screen
- app launcher
- status bar
- shortcuts
- theme/wallpaper
- điều hướng giữa Home và ứng dụng

### Symbian Backend

Giữ nguyên bên trong EKA2L1:

- guest kernel / process model
- Window Server
- AppArc / app registry
- filesystem
- Central Repository
- IPC services
- input
- audio
- ứng dụng Symbian

## Nguyên tắc bridge

Host Shell không tự mô phỏng ứng dụng Symbian.

Ví dụ launcher:

```text
Hybrid Home
  → bridge::get_apps()
  → app UID + app name
  → launchAppUid(uid)
  → bridge::launch_app(uid)
  → EKA2L1 launcher
  → real guest process
```

## HYBRIDHOME1

HYBRIDHOME1 dùng UIKit để dựng proof-of-concept Home surface.

Guest Menu3 UID `0x101F4CD2` vẫn chạy bên dưới.

Trong lúc Hybrid Home hiển thị, host chiếm input để sự kiện không lọt xuống Menu3.

Khi chọn một app:

1. Hybrid Home ẩn.
2. Frontend chuyển `currentGameUid` sang app đích.
3. Luồng launcher hiện có của EKA2L1 được sử dụng.
4. Guest app được tạo theo cơ chế bình thường.

## Tại sao không dùng Home UID 0x102750F0 ở đường Hybrid

Home gốc của RM-356 có startup chain phức tạp với nhiều phụ thuộc. Hybrid Home cho phép nghiên cứu usability của hệ thống mà không phải chờ mọi dependency đó hoàn thiện.

Điều này không ngăn dự án HOMEONLY tiếp tục sửa Home thật.

## Mục tiêu dài hạn

Hybrid Home nên ngày càng ít hardcode và ngày càng lấy dữ liệu từ firmware thật:

- app registry
- icon
- resource strings
- CenRep shortcuts
- wallpaper/theme
- locale
- system state

Mục tiêu cuối cùng là **Shell tái tạo, backend thật**.
