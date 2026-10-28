# 📸 Spoof Screenshot Xposed (`v1.0.0`)

Module **LSPosed / Xposed** chuyên dụng dành cho Android (**Android 8 – Android 15 / Xiaomi HyperOS**), giúp **mở khóa chụp ảnh màn hình & quay màn hình trên mọi ứng dụng chặn chụp màn hình (`FLAG_SECURE`)**, đồng thời **vô hiệu hóa cơ chế phát hiện chụp màn hình (`Anti-Screenshot Detection`)** ở cả tầng **System Framework (`system_server`)** lẫn tầng ứng dụng.

---

## ✨ Tính năng nổi bật của [`SpoofScreenshot.apk`](./SpoofScreenshot.apk) (`v1.0.0` — `com.heavietnam.spoofscreenshot`)

1. **Hook Tầng Khung Hệ Thống (`system_server` — `android`)**:
   * `com.android.server.wm.WindowState.isSecureLocked()` $\rightarrow$ luôn trả về `false`.
   * `com.android.server.wm.WindowManagerServiceImpl.notAllowCaptureDisplay()` *(cơ chế chặn chụp màn hình riêng của Xiaomi MIUI / HyperOS)* $\rightarrow$ luôn trả về `false`.
   * `com.android.server.wm.WindowSurfaceController` (Constructor & `setSecure`) $\rightarrow$ xóa cờ `SurfaceControl.SECURE` (`0x80`) và ép `false`.
   * `com.android.server.wm.WindowManagerService.captureDisplay()` $\rightarrow$ ép `mCaptureSecureLayers = true` cho mọi `CaptureArgs`.
   * `com.android.server.devicepolicy.DevicePolicyManagerService.getScreenCaptureDisabled()` $\rightarrow$ luôn trả về `false`.
   * **Ưu điểm quyết định**: Nhờ hook ở tầng `system_server`, bạn chụp ảnh và quay màn hình được trên **mọi ứng dụng (kể cả VNeID, VCB Digibank, Techcombank, MB Bank...)** mà **KHÔNG cần tích các app ngân hàng đó trong LSPosed**!

2. **Chặn Phát Hiện Chụp Màn Hình (`Anti-Screenshot Detection`)**:
   * `com.android.server.wm.ActivityTaskManagerService.registerScreenCaptureObserver()` (Android 14/15) $\rightarrow$ vô hiệu hóa (`no-op`).
   * `com.android.server.wm.WindowManagerService.notifyScreenshotListeners()` (MIUI / AOSP) $\rightarrow$ trả về danh sách rỗng `Collections.emptyList()`.
   * `android.app.Activity.registerScreenCaptureCallback()` (Android 14/15) $\rightarrow$ vô hiệu hóa (`no-op`).

3. **Hook Tầng Framework / Ứng Dụng (`SystemUI`, `MIUI Screenshot`, `MIUI ScreenRecorder` & App Scope)**:
   * `android.window.ScreenCapture$CaptureArgs` & `Builder` $\rightarrow$ luôn ép `mCaptureSecureLayers = true`.
   * `android.view.SurfaceControl$Builder` & `SurfaceControl$Transaction` (`setSecure`, `setFlags`) $\rightarrow$ xóa bit `0x80` (`SECURE`).
   * `android.view.Window.setFlags()` / `addFlags()` & `WindowManagerImpl` / `WindowManagerGlobal` (`addView`, `updateViewLayout`) $\rightarrow$ xóa cờ `WindowManager.LayoutParams.FLAG_SECURE` (`0x2000`).
   * `android.view.SurfaceView.setSecure(boolean)` $\rightarrow$ luôn ép `false`.
   * `android.app.Activity.setRecentsScreenshotEnabled(boolean)` $\rightarrow$ luôn ép `true` (hiển thị ảnh xem trước trong màn hình Đa nhiệm).

---

## 🛠️ Hướng dẫn Cài đặt & Sử dụng

### Bước 1: Cài đặt APK
```bash
adb install -r SpoofScreenshot.apk
```

### Bước 2: Kích hoạt trong LSPosed
1. Mở **LSPosed Manager** $\rightarrow$ vào mục **Mô-đun (Modules)** $\rightarrow$ bật **Spoof Screenshot**.
2. Giữ nguyên 4 mục khuyến nghị được tích sẵn:
   * ✅ **Framework Hệ thống (`system`)**
   * ✅ **Giao diện người dùng hệ thống (`com.android.systemui`)**
   * ✅ **Chụp ảnh màn hình (`com.miui.screenshot`)**
   * ✅ **Quay phim Màn hình (`com.miui.screenrecorder`)**
3. Nếu dùng **Hide My Applist (HMA-OSS)**, thêm `com.heavietnam.spoofscreenshot` vào template ẩn để các ứng dụng ngân hàng không quét thấy gói cài đặt module.
4. **Khởi động lại máy (Reboot) 1 lần** để hook trong `system_server` có hiệu lực toàn cục cho mọi ứng dụng trên máy. *(Với các ứng dụng thông thường không chống hook, có thể tích trực tiếp ứng dụng đó trong LSPosed và buộc dừng app để áp dụng ngay không cần khởi động lại máy).*

---

## 📂 Cấu trúc Dự án

```text
├── SpoofScreenshot.apk                   # APK cài đặt ngay của module Spoof Screenshot (v1.0.0)
├── README.md                             # Tài liệu hướng dẫn chi tiết
└── SpoofScreenshotAndroid/               # Mã nguồn module Spoof Screenshot
    └── app/
        ├── libs/xposed-api.jar           # Thư viện stub Xposed API 82
        └── src/main/
            ├── AndroidManifest.xml
            ├── assets/xposed_init
            ├── res/values/arrays.xml
            └── java/com/heavietnam/spoofscreenshot/
                ├── SpoofScreenshotHook.java
                └── MainActivity.java
```

---

## 📄 Bản quyền (Copyright)

**© 2026 [Heavietnam](https://home.heavietnam.com/). All rights reserved.**
Phát triển & tối ưu bởi **[Heavietnam](https://home.heavietnam.com/)**.
