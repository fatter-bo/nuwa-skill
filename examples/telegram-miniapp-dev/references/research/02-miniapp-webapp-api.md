# Mini App (WebApp) 开发

> 来源: https://core.telegram.org/bots/webapps

## 1. 技术本质

Mini App 本质是运行在 Telegram 客户端内嵌 WebView 中的 H5 页面。通过引入 `telegram-web-app.js` 脚本获取 `window.Telegram.WebApp` 对象与 Telegram 客户端通信。

```html
<script src="https://telegram.org/js/telegram-web-app.js"></script>
```

## 2. 初始化与身份验证

### initData
- 原始查询字符串，用于服务端验证
- 包含: query_id, user, chat, chat_type, chat_instance, start_param, auth_date, hash, signature
- **服务端验证**: HMAC-SHA256(data-check-string, secret_key) 比对 hash
- **第三方验证**: base64url Ed25519 签名验证

### initDataUnsafe
- 解析后的对象（仅客户端使用，不可信）
- user 包含: id, first_name, last_name, username, language_code, is_premium, photo_url

### 关键属性
- `version`: 当前客户端支持的 Bot API 版本
- `platform`: 平台标识 (ios/android/tdesktop/web 等)
- `colorScheme`: "light" | "dark"
- `isExpanded` / `isFullscreen` / `isActive`: 窗口状态

## 3. UI 控制组件

### MainButton (BottomButton)
- 底部主按钮，支持文字/颜色/加载状态/图标
- 方法: show() / hide() / enable() / disable() / setText() / setParams() / showProgress()
- 事件: mainButtonClicked

### SecondaryButton
- 次要按钮，支持上下左右位置
- 与 MainButton 共享 BottomButton 接口

### BackButton
- 顶部返回按钮
- 方法: show() / hide() / onClick() / offClick()
- 事件: backButtonClicked

### SettingsButton
- 上下文菜单中的设置项
- 方法: show() / hide() / onClick() / offClick()

## 4. 主题适配

### ThemeParams (16+ 颜色变量)
- bg_color / text_color / hint_color / link_color
- button_color / button_text_color
- secondary_bg_color / header_bg_color / bottom_bar_bg_color
- accent_text_color / section_bg_color / section_header_text_color
- section_separator_color / subtitle_text_color / destructive_text_color

### CSS 变量
所有颜色可通过 `var(--tg-theme-bg-color)` 等 CSS 变量使用。

### 自定义
- setHeaderColor(color) / setBackgroundColor(color) / setBottomBarColor(color)

## 5. 窗口模式与启动方式

7 种部署模式:
1. **Keyboard Button**: sendData() 直接回传数据，无需服务器
2. **Inline Button**: 完整交互，可发送预制消息
3. **Menu Button**: 快速启动变体
4. **Main Mini App**: 从 Bot 主页启动，支持 startapp 参数
5. **Inline Mode**: 创建内容嵌入内联查询
6. **Direct Link**: `t.me/botname/appname?startapp=xxx`，支持 mode=compact
7. **Attachment Menu**: 从附件菜单启动

### 全屏模式
- requestFullscreen() / exitFullscreen()
- lockOrientation() / unlockOrientation()
- safeAreaInset / contentSafeAreaInset 安全区域

## 6. 存储 API

### CloudStorage（服务端同步）
- 每 Bot 每用户最多 1024 项，每项最大 4096 字符
- setItem / getItem / getItems / removeItem / removeItems / getKeys

### DeviceStorage（本地持久化）
- 每用户 5MB
- setItem / getItem / removeItem / clear

### SecureStorage（加密本地存储）
- 最多 10 项，使用 Keychain(iOS) / Keystore(Android)
- setItem / getItem / restoreItem / removeItem / clear

## 7. 硬件访问

- **Accelerometer**: 加速度计 (x, y, z m/s^2)
- **DeviceOrientation**: 设备方向 (alpha, beta, gamma rad)
- **Gyroscope**: 陀螺仪 (x, y, z rad/s)
- **LocationManager**: 地理位置（需初始化和授权）

## 8. 生物识别 (BiometricManager)

- init() 初始化
- requestAccess(params) 请求权限
- authenticate(params) 执行认证
- updateBiometricToken() 更新令牌
- 属性: isInited / isBiometricAvailable / biometricType / isAccessRequested / isAccessGranted

## 9. 触觉反馈 (HapticFeedback)

- impactOccurred(style): light / medium / heavy / rigid / soft
- notificationOccurred(type): error / success / warning
- selectionChanged(): 选择变化反馈

## 10. 关键方法清单

### 导航
- openLink(url, options) / openTelegramLink(url) / switchInlineQuery(query, types)
- openInvoice(url, callback) / shareMessage(msg_id, callback) / shareToStory(media_url, params)

### 用户交互
- requestWriteAccess / requestContact / requestChat / requestEmojiStatusAccess / setEmojiStatus

### 文件与剪贴板
- downloadFile(params, callback) / readTextFromClipboard(callback)

### UI 弹窗
- showPopup(params, callback) / showAlert(message, callback) / showConfirm(message, callback)
- showScanQrPopup(params, callback) / closeScanQrPopup()

### 生命周期
- ready() / expand() / close() / enableClosingConfirmation() / disableClosingConfirmation()
- enableVerticalSwipes() / disableVerticalSwipes() / addToHomeScreen() / checkHomeScreenStatus()

## 11. 事件系统 (25+ 事件)

- 生命周期: activated / deactivated / themeChanged / viewportChanged / safeAreaChanged
- UI: mainButtonClicked / secondaryButtonClicked / backButtonClicked / settingsButtonClicked / popupClosed
- 硬件: accelerometer* / deviceOrientation* / gyroscope* / locationManager* / locationRequested
- 权限: writeAccessRequested / contactRequested / biometricManager* / biometricAuth*
- 功能: invoiceClosed / qrTextReceived / clipboardTextReceived / shareMessage* / fullscreen* / homeScreen*
