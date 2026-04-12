# Telegram Bot API 基础

> 来源: https://core.telegram.org/bots/api, https://grammy.dev/guide/

## 1. Bot 创建与配置

- 通过 BotFather 创建 Bot，获取唯一 token（格式: `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`）
- 所有 API 调用走 HTTPS: `https://api.telegram.org/bot<token>/METHOD_NAME`
- 参数传递支持: URL query string / application/x-www-form-urlencoded / application/json / multipart/form-data（上传文件）
- BotFather 可配置: Bot 名称/描述/头像/命令列表/Menu Button/Inline Mode/Payments 等

## 2. Updates 机制

两种互斥的获取方式:

### Long Polling (getUpdates)
- 主动轮询，返回 Update 对象数组
- 可配置 timeout（推荐 30s）实现长轮询
- 开发环境友好，无需公网 HTTPS
- 适合低并发场景

### Webhook (setWebhook)
- Telegram 主动 POST 到指定 HTTPS URL
- 支持自定义证书和 IP 地址
- 最大 100 个并发连接（可配置）
- 支持端口: 443, 80, 88, 8443
- 生产环境首选，实时性好

## 3. 核心概念

### Commands
- 以 `/` 开头，可在 BotFather 注册
- 支持 scope（针对特定聊天/用户设置不同命令列表）
- 必须实现 `/start` 和 `/help`

### Inline Mode
- 用户在任意聊天输入 `@botname query` 触发
- Bot 返回 InlineQueryResult 数组供用户选择
- 支持缓存结果（cache_time）

### Payments
- sendInvoice / createInvoiceLink 创建支付
- pre_checkout_query 预检查（10秒内必须响应）
- successful_payment 确认支付完成
- 支持 Telegram Stars（XTR）和第三方支付

### Games
- sendGame 发送游戏消息
- setGameScore 设置分数
- getGameHighScores 获取排行榜
- 游戏本质是内嵌 HTML5 页面

## 4. Bot SDK 选型

| SDK | 语言 | 特点 | 推荐场景 |
|-----|------|------|---------|
| grammY | TS/JS | 现代化、类型安全、插件丰富、支持 Deno | 新项目首选 |
| Telegraf | TS/JS | 成熟稳定、生态完善、中间件模式 | 存量 Node.js 项目 |
| python-telegram-bot | Python | 官方维护、文档完善 | Python 后端 |
| aiogram | Python | 异步优先、性能好 | 高并发 Python 场景 |

### grammY 核心特性
- 中间件架构: `bot.on("message", ctx => {...})`
- Filter queries: `bot.on("message:text")` 精确过滤
- 内置 session 插件支持多存储后端
- 官方插件: conversations / menus / hydration / i18n
- 部署灵活: VPS / Deno Deploy / Cloudflare Workers / Vercel

## 5. API 关键更新（2025-2026）

- Bot API 9.6 (2026.04): Managed Bot 创建、多答案测验
- Mini App 全屏模式、设备传感器访问（加速度计/陀螺仪/方向）
- BiometricManager 生物识别
- CloudStorage / DeviceStorage / SecureStorage 三级存储
- Stars 支付全面落地
