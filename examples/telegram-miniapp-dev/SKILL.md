---
name: telegram-miniapp-dev
description: |
  Telegram Mini App开发专家。系统掌握Telegram Bot API和Mini App（WebApp）完整开发技术栈，
  覆盖从Bot创建到WebApp开发到TON Connect钱包接入到TG Stars支付到平台分发的全流程。
  触发词：「TG小游戏」「Telegram Bot」「Mini App」「WebApp」「TON Connect」「TG Stars」「电报小游戏」「telegram开发」
---

# Telegram Mini App 开发专家

> 核心目标：系统输出 Telegram Mini App 全栈开发方案——从 Bot 搭建到 WebApp 开发到 TON 钱包接入到 Stars 支付到增长分发，一站式覆盖。

## Skill 规则

- Bot SDK 首选 grammY（TypeScript）或 aiogram（Python），新项目避免选择已停止维护的框架
- Mini App 前端必须引入 `telegram-web-app.js`，所有用户身份以服务端验证 initData 为准，禁止信任 initDataUnsafe
- 数字商品/服务支付必须使用 Telegram Stars（XTR），链上资产交易走 TON Connect
- 游戏引擎推荐 Phaser.js（2D 轻量）或 Cocos Creator（高性能），根据玩法复杂度选型
- 所有方案必须考虑多平台兼容性（iOS/Android/Desktop/Web），尤其注意 WebView 差异

---

## 回答工作流

| 触发指令 | 行动 |
|---------|------|
| 「设计TG小游戏方案」 | 接收游戏类型和需求，输出完整 Mini App 技术方案（架构/引擎/通信/支付/分发） |
| 「创建Telegram Bot」 | 输出 Bot 创建配置流程 + SDK 初始化代码 + Webhook/Polling 配置 |
| 「开发Mini App」 | 输出 WebApp 项目结构 + initData 验证 + 主题适配 + 核心 API 使用 |
| 「接入TON Connect」 | 输出钱包连接方案 + React 集成代码 + 交易发送 + Jetton/NFT 交互 |
| 「接入Stars支付」 | 输出支付流程 + Invoice 创建 + Webhook 回调 + 退款处理 + 定价策略 |
| 「设计增长方案」 | 输出分发渠道 + 邀请系统 + 病毒传播机制 + 社群运营策略 |
| 「评审XXX方案」 | 按七大维度（安全/性能/合规/体验/增长/支付/架构）审计，输出问题清单 |

---

## Agentic Protocol

**Step 1: 需求分析与技术方案**
接收游戏类型（点击/合成/养成/对战/棋牌等）和核心需求，输出：
- 技术栈选型（Bot SDK + 前端框架 + 游戏引擎 + 后端 + 数据库）
- 系统架构图（Bot ↔ 后端 ↔ Mini App ↔ TON）
- 关键模块划分与接口设计

**Step 2: 关键代码模块输出**
依次输出可运行的代码模块：
- Bot 初始化 + Webhook 配置
- Mini App initData 服务端验证
- TON Connect 钱包连接 + 交易发送
- Stars 支付 Invoice 创建 + 回调处理
- 游戏前端与后端通信方案

**Step 3: 上线检查清单**
输出部署前必须逐项确认的检查清单（安全/支付/性能/合规/分发）。

---

## 一、Telegram Bot API 基础

### 1.1 Bot 创建与配置

通过 BotFather 创建 Bot，获取 token：

```
/newbot → 输入名称 → 输入用户名 → 获取 token
/setdescription → 设置描述
/setcommands → 注册命令列表
/newapp → 创建 Mini App
/setmenubutton → 配置菜单按钮
```

### 1.2 grammY 初始化（推荐）

```typescript
import { Bot, webhookCallback } from "grammy";
import express from "express";

const bot = new Bot(process.env.BOT_TOKEN!);

// 命令处理
bot.command("start", async (ctx) => {
  await ctx.reply("Welcome!", {
    reply_markup: {
      inline_keyboard: [[
        { text: "Play Game", web_app: { url: "https://your-app.com" } }
      ]]
    }
  });
});

// Webhook 模式（生产环境）
const app = express();
app.use(express.json());
app.use("/webhook", webhookCallback(bot, "express"));

// 设置 Webhook
await bot.api.setWebhook("https://your-server.com/webhook");

app.listen(3000);
```

### 1.3 Webhook vs Long Polling 选型

| 维度 | Webhook | Long Polling |
|------|---------|-------------|
| 实时性 | 即时推送 | 取决于轮询间隔 |
| 基础设施 | 需公网 HTTPS | 无需公网 |
| 适用场景 | 生产环境 | 开发/调试 |
| 并发处理 | 最大100并发连接 | 顺序处理 |
| 容错 | 自动重试 | 需自行处理 |

**生产建议**：Webhook + 反向代理（Nginx）+ SSL（Let's Encrypt）。

---

## 二、Mini App (WebApp) 开发

### 2.1 项目结构

```
mini-app/
  ├── public/
  │   └── index.html          # 入口，引入 telegram-web-app.js
  ├── src/
  │   ├── App.tsx              # React 主组件
  │   ├── hooks/
  │   │   ├── useTelegramApp.ts    # WebApp API 封装
  │   │   └── useThemeParams.ts    # 主题适配
  │   ├── utils/
  │   │   └── initDataValidator.ts # 服务端验证
  │   └── pages/
  ├── server/
  │   ├── bot.ts               # Bot 逻辑
  │   └── api.ts               # 后端 API
  └── package.json
```

### 2.2 initData 服务端验证（核心安全）

```typescript
import crypto from "crypto";

function validateInitData(initData: string, botToken: string): boolean {
  const params = new URLSearchParams(initData);
  const hash = params.get("hash");
  params.delete("hash");

  // 按 key 字母排序，拼成 key=value\n 格式
  const dataCheckString = Array.from(params.entries())
    .sort(([a], [b]) => a.localeCompare(b))
    .map(([k, v]) => `${k}=${v}`)
    .join("\n");

  // 计算 secret_key 和 hash
  const secretKey = crypto
    .createHmac("sha256", "WebAppData")
    .update(botToken)
    .digest();

  const calculatedHash = crypto
    .createHmac("sha256", secretKey)
    .update(dataCheckString)
    .digest("hex");

  return calculatedHash === hash;
}
```

### 2.3 WebApp API 封装

```typescript
// hooks/useTelegramApp.ts
export function useTelegramApp() {
  const tg = window.Telegram.WebApp;

  return {
    // 用户信息
    user: tg.initDataUnsafe?.user,
    startParam: tg.initDataUnsafe?.start_param,

    // UI 控制
    ready: () => tg.ready(),
    expand: () => tg.expand(),
    close: () => tg.close(),

    // MainButton
    showMainButton: (text: string, onClick: () => void) => {
      tg.MainButton.setText(text);
      tg.MainButton.onClick(onClick);
      tg.MainButton.show();
    },
    hideMainButton: () => tg.MainButton.hide(),

    // BackButton
    showBackButton: (onClick: () => void) => {
      tg.BackButton.onClick(onClick);
      tg.BackButton.show();
    },

    // 触觉反馈
    hapticImpact: (style: "light" | "medium" | "heavy") =>
      tg.HapticFeedback.impactOccurred(style),
    hapticNotification: (type: "error" | "success" | "warning") =>
      tg.HapticFeedback.notificationOccurred(type),

    // 支付
    openInvoice: (url: string) =>
      new Promise<string>((resolve) => tg.openInvoice(url, resolve)),

    // 主题
    colorScheme: tg.colorScheme,
    themeParams: tg.themeParams,

    // 全屏
    requestFullscreen: () => tg.requestFullscreen(),
    exitFullscreen: () => tg.exitFullscreen(),

    // 存储
    cloudStorage: tg.CloudStorage,
  };
}
```

### 2.4 主题适配

```css
/* 使用 Telegram 主题 CSS 变量 */
body {
  background-color: var(--tg-theme-bg-color);
  color: var(--tg-theme-text-color);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
}

.button-primary {
  background-color: var(--tg-theme-button-color);
  color: var(--tg-theme-button-text-color);
  border: none;
  border-radius: 12px;
  padding: 12px 24px;
}

.hint-text {
  color: var(--tg-theme-hint-color);
}

.section {
  background-color: var(--tg-theme-section-bg-color);
  border-bottom: 1px solid var(--tg-theme-section-separator-color);
}

/* 安全区域适配（全屏模式） */
.container {
  padding-top: env(safe-area-inset-top);
  padding-bottom: env(safe-area-inset-bottom);
}
```

### 2.5 存储 API 三级体系

| 存储类型 | 容量 | 同步方式 | 适用场景 |
|---------|------|---------|---------|
| CloudStorage | 1024项 x 4KB | 服务端同步 | 用户设置/进度/偏好 |
| DeviceStorage | 5MB | 本地持久化 | 缓存/离线数据 |
| SecureStorage | 10项 | 加密本地 | 密钥/Token/敏感信息 |

---

## 三、TG 小游戏开发实践

### 3.1 游戏引擎选型

```
┌─────────────────────────────────────────────────────┐
│  玩法复杂度低        中等             高              │
│  (点击/合成/Idle)  (平台跳/消除)   (SLG/棋牌/RPG)   │
│                                                     │
│  纯 Canvas/PixiJS   Phaser.js      Cocos Creator    │
│  包体 < 100KB       包体 300KB     包体 > 1MB       │
│  零依赖              2D 全能        2D/3D 全能       │
│  1-2周开发           2-4周开发      4-8周开发         │
└─────────────────────────────────────────────────────┘
```

### 3.2 Phaser.js + Mini App 集成

```typescript
import Phaser from "phaser";

class GameScene extends Phaser.Scene {
  private tg = window.Telegram.WebApp;

  create() {
    // 通知 Telegram 加载完成
    this.tg.ready();
    this.tg.expand();

    // 使用 Telegram 主题色
    const bgColor = this.tg.themeParams.bg_color || "#ffffff";
    this.cameras.main.setBackgroundColor(bgColor);

    // 游戏逻辑...
    this.add.text(100, 100, "Tap to Play!", { fontSize: "24px" });
  }

  submitScore(score: number) {
    // 提交分数到后端
    fetch("/api/score", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "X-Init-Data": this.tg.initData, // 服务端验证身份
      },
      body: JSON.stringify({ score }),
    });
  }
}

const config: Phaser.Types.Core.GameConfig = {
  type: Phaser.AUTO,
  width: window.innerWidth,
  height: window.innerHeight,
  parent: "game-container",
  scene: GameScene,
  scale: {
    mode: Phaser.Scale.RESIZE,
    autoCenter: Phaser.Scale.CENTER_BOTH,
  },
};

new Phaser.Game(config);
```

### 3.3 用户身份与排行榜

```typescript
// 服务端：验证身份 + 更新分数 + 返回排行榜
app.post("/api/score", async (req, res) => {
  const initData = req.headers["x-init-data"] as string;
  if (!validateInitData(initData, BOT_TOKEN)) {
    return res.status(401).json({ error: "Invalid initData" });
  }

  const params = new URLSearchParams(initData);
  const user = JSON.parse(params.get("user")!);
  const { score } = req.body;

  // Redis Sorted Set 排行榜
  await redis.zadd("leaderboard", score, user.id.toString());

  // 同步到 Bot API 排行榜（可选）
  // await bot.api.setGameScore(user.id, score, chatId, messageId);

  const rank = await redis.zrevrank("leaderboard", user.id.toString());
  const top10 = await redis.zrevrange("leaderboard", 0, 9, "WITHSCORES");

  res.json({ rank: rank! + 1, top10 });
});
```

### 3.4 分享邀请机制

```typescript
// Bot 端：生成邀请链接
bot.command("invite", async (ctx) => {
  const inviteCode = generateInviteCode(ctx.from.id);
  const link = `https://t.me/YourBot/game?startapp=invite_${inviteCode}`;
  await ctx.reply(`Share this link to invite friends:\n${link}`);
});

// Mini App 端：检测邀请来源
const startParam = window.Telegram.WebApp.initDataUnsafe?.start_param;
if (startParam?.startsWith("invite_")) {
  const inviteCode = startParam.replace("invite_", "");
  await fetch("/api/accept-invite", {
    method: "POST",
    headers: { "X-Init-Data": window.Telegram.WebApp.initData },
    body: JSON.stringify({ inviteCode }),
  });
}

// switchInlineQuery：让用户在聊天中分享
window.Telegram.WebApp.switchInlineQuery("Play with me!", ["users", "groups"]);
```

---

## 四、TON Connect 与 Web3 集成

### 4.1 React 集成方案

```tsx
import { TonConnectUIProvider, TonConnectButton, useTonConnectUI, useTonWallet } from "@tonconnect/ui-react";

// 1. Provider 包裹应用
function App() {
  return (
    <TonConnectUIProvider manifestUrl="https://your-app.com/tonconnect-manifest.json">
      <Game />
    </TonConnectUIProvider>
  );
}

// 2. 连接按钮 + 交易发送
function Game() {
  const [tonConnectUI] = useTonConnectUI();
  const wallet = useTonWallet();

  const sendPayment = async () => {
    if (!wallet) {
      tonConnectUI.openModal();
      return;
    }

    await tonConnectUI.sendTransaction({
      validUntil: Math.floor(Date.now() / 1000) + 300,
      messages: [{
        address: "EQxxxx...",         // 目标地址
        amount: "500000000",          // 0.5 TON (nanoToncoin)
      }],
    });
  };

  return (
    <div>
      <TonConnectButton />
      <button onClick={sendPayment}>Buy Item (0.5 TON)</button>
    </div>
  );
}
```

### 4.2 Manifest 文件

```json
{
  "url": "https://your-app.com",
  "name": "Your Game",
  "iconUrl": "https://your-app.com/icon-192.png"
}
```

部署要求：应用根目录可访问，无 CORS 限制，无授权拦截。

### 4.3 链上积分方案（混合模式推荐）

```
高频操作（游戏内）        低频操作（资产化）
    │                          │
    ▼                          ▼
链下积分系统              链上 Jetton/NFT
(Redis/PostgreSQL)        (TON 智能合约)
    │                          │
    └───── 定期批量上链 ────────┘
          (降低 Gas 成本)
```

---

## 五、TG Stars 支付系统

### 5.1 完整支付流程

```typescript
// === Bot 端 ===

// 1. 创建 Invoice 链接
bot.command("buy", async (ctx) => {
  const link = await ctx.api.createInvoiceLink(
    "Premium Pack",                    // title
    "Unlock all premium features",     // description
    "premium_pack_v1",                 // payload（自定义，用于识别商品）
    "",                                // provider_token（Stars 留空）
    "XTR",                             // currency（必须 XTR）
    [{ label: "Premium Pack", amount: 150 }]  // 150 Stars
  );
  await ctx.reply("Get Premium!", {
    reply_markup: {
      inline_keyboard: [[
        { text: "Buy (150 Stars)", url: link }
      ]]
    }
  });
});

// 2. 预检查回调（10秒内必须响应）
bot.on("pre_checkout_query", async (ctx) => {
  // 验证库存/资格
  const payload = ctx.preCheckoutQuery.invoice_payload;
  const isValid = await checkInventory(payload);
  await ctx.answerPreCheckoutQuery(isValid);
});

// 3. 支付成功回调
bot.on("message:successful_payment", async (ctx) => {
  const payment = ctx.message.successful_payment;
  const chargeId = payment.telegram_payment_charge_id;
  const payload = payment.invoice_payload;

  // 必须保存 chargeId（用于退款）
  await db.savePayment({
    userId: ctx.from.id,
    chargeId,
    payload,
    amount: payment.total_amount,
    timestamp: Date.now(),
  });

  // 发放商品
  await deliverProduct(ctx.from.id, payload);
  await ctx.reply("Payment successful! Enjoy your Premium Pack!");
});
```

### 5.2 Mini App 内支付

```typescript
// Mini App 前端
async function buyItem(itemId: string) {
  // 1. 向后端请求创建 Invoice
  const res = await fetch("/api/create-invoice", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Init-Data": window.Telegram.WebApp.initData,
    },
    body: JSON.stringify({ itemId }),
  });
  const { invoiceLink } = await res.json();

  // 2. 打开支付弹窗
  const status = await new Promise<string>((resolve) => {
    window.Telegram.WebApp.openInvoice(invoiceLink, resolve);
  });

  // 3. 处理结果
  if (status === "paid") {
    showSuccess("Purchase complete!");
    refreshInventory();
  } else if (status === "cancelled") {
    // 用户取消，不做处理
  } else {
    showError("Payment failed, please try again");
  }
}
```

### 5.3 Stars vs TON 支付选型

```
数字商品/服务（皮肤/VIP/道具）  →  Stars 支付（合规 + 低门槛）
链上资产（NFT/Jetton/DeFi）    →  TON Connect（去中心化 + 即时到账）
混合模式                       →  Stars 入门 + TON 高级玩家
```

---

## 六、平台分发与增长

### 6.1 增长飞轮

```
新用户（深度链接/频道/群组/目录）
         │
         ▼
    体验核心玩法 ──────→ 流失
         │
         ▼
  产生社交行为（排行/成就/挑战）
         │
         ▼
  分享给好友（邀请链接/Inline/Story）
         │
         ▼
  获得奖励 ←── 好友注册 ──→ 新用户
```

### 6.2 曝光渠道矩阵

| 渠道 | 成本 | 规模 | 质量 |
|------|------|------|------|
| 邀请奖励 | 虚拟货币成本 | 指数增长 | 高（社交推荐） |
| Inline Mode | 零 | 取决于用户活跃度 | 高 |
| 频道/群组 | 运营成本 | 中等 | 中等 |
| Telegram Ads | CMP 付费 | 大（可控） | 中等 |
| Mini App 目录 | 零 | 大（被动） | 中等 |
| Story 分享 | 零 | 取决于用户关注者 | 高 |
| Bot 消息召回 | 零 | 存量用户 | 高（精准） |

### 6.3 召回策略

```typescript
// 分层召回（基于不活跃天数）
async function recallInactiveUsers() {
  const segments = [
    { days: 3, message: "Your crops are ready to harvest!" },
    { days: 7, message: "You have unclaimed rewards waiting!" },
    { days: 14, message: "We miss you! Come back for a special bonus!" },
    { days: 30, message: "Welcome back gift: 500 coins!" },
  ];

  for (const seg of segments) {
    const users = await db.getInactiveUsers(seg.days);
    for (const user of users) {
      await bot.api.sendMessage(user.telegramId, seg.message, {
        reply_markup: {
          inline_keyboard: [[
            { text: "Play Now", web_app: { url: APP_URL } }
          ]]
        }
      });
    }
  }
}
```

---

## 框架（Frameworks）

### F-1: Mini App 全栈技术架构

```
┌─ Telegram Client ──────────────────────────────────┐
│  WebView (Mini App)                                 │
│  ┌─ Frontend ───────────────────────────────────┐  │
│  │  React/Vue + Phaser.js/Cocos + TG WebApp SDK │  │
│  │  + TON Connect UI                            │  │
│  └──────────────────────────────────────────────┘  │
│                    ↕ HTTPS / WSS                    │
└────────────────────────────────────────────────────┘
                     ↕
┌─ Backend ──────────────────────────────────────────┐
│  Node.js (Express/Fastify) + grammY Bot            │
│  ├─ initData 验证层                                │
│  ├─ 业务逻辑层（游戏/支付/社交）                     │
│  ├─ Stars 支付回调处理                              │
│  └─ TON RPC 交互层                                 │
├────────────────────────────────────────────────────┤
│  Redis (排行榜/会话/缓存)                           │
│  PostgreSQL/MongoDB (持久化数据)                     │
└────────────────────────────────────────────────────┘
                     ↕
┌─ TON Blockchain ───────────────────────────────────┐
│  Jetton 合约 / NFT 合约 / 自定义合约                │
└────────────────────────────────────────────────────┘
```

### F-2: initData 验证 + 身份中间件

所有 Mini App 后端 API 必须经过 initData 验证中间件：
1. 从请求头提取 initData
2. HMAC-SHA256 验证签名
3. 检查 auth_date 时效性（建议 5 分钟内）
4. 提取 user.id 作为业务身份标识
5. 注入 ctx.telegramUser 供下游使用

### F-3: Stars 支付三阶段框架

```
阶段一: Invoice 创建
  → createInvoiceLink(title, desc, payload, "", "XTR", prices)
  → 返回支付链接给前端

阶段二: 预检查
  → pre_checkout_query 回调
  → 10秒内 answerPreCheckoutQuery(ok/error)
  → 验证库存、价格、用户资格

阶段三: 交付
  → successful_payment 回调
  → 保存 telegram_payment_charge_id
  → 发放商品 + 更新用户状态
  → 发送确认消息
```

### F-4: TON Connect 钱包交互框架

```
连接阶段: TonConnectButton → openModal → 用户选择钱包 → 建立会话
交易阶段: sendTransaction({validUntil, messages}) → 钱包签名 → 广播
验证阶段: 后端监听链上事件 / 轮询交易状态 → 确认到账 → 更新状态
```

### F-5: 游戏-Bot-Mini App 三角通信框架

```
Bot (grammY)
  ├── /start → 欢迎消息 + Mini App 按钮
  ├── /invite → 生成邀请链接
  ├── /rank → 查询排名
  ├── pre_checkout_query → 支付预检查
  └── successful_payment → 支付确认

Mini App (React + Game Engine)
  ├── 启动 → ready() + expand() + initData 验证
  ├── 游戏循环 → WebSocket 实时通信
  ├── 支付 → openInvoice() → invoiceClosed 事件
  ├── 分享 → switchInlineQuery() / shareToStory()
  └── 设置 → CloudStorage 读写

Backend (Node.js)
  ├── initData 验证中间件
  ├── 游戏逻辑 + 反作弊
  ├── 排行榜 (Redis Sorted Set)
  ├── 支付流水 (PostgreSQL)
  └── TON RPC 交互
```

### F-6: 病毒增长引擎框架

```
触发层: 游戏内成就/排名变化/限时活动 → 激发分享欲望
机制层: 邀请码 + 双向奖励 + Squad 排名 + Story 分享
渠道层: 深度链接 / Inline Mode / 频道推送 / Telegram Ads
转化层: start_param 追踪 → 注册奖励 → 引导核心玩法
度量层: K-factor / 邀请转化率 / 次日留存 / ARPU
```

### F-7: 上线部署检查框架

```
安全检查:
  □ initData 服务端验证已实现
  □ HTTPS 已配置（SSL证书有效）
  □ Bot Token 未暴露在前端代码中
  □ 敏感操作有频率限制
  □ 反作弊机制已部署

支付检查:
  □ pre_checkout_query 10秒响应保证
  □ telegram_payment_charge_id 持久化存储
  □ /terms 和 /support 命令已实现
  □ 退款流程已测试
  □ 支付失败回滚逻辑完备

性能检查:
  □ 首屏加载 < 2秒（已调用 ready()）
  □ 资源已压缩（图片 WebP/AVIF、JS 压缩）
  □ WebSocket 心跳和断线重连已实现
  □ 低端设备降级策略已配置

兼容性检查:
  □ iOS / Android / Desktop / Web 四端测试通过
  □ Light / Dark 主题适配正确
  □ 安全区域（SafeArea）适配到位
  □ isVersionAtLeast() 版本检查已添加

分发检查:
  □ 深度链接可正常打开 Mini App
  □ 邀请系统端到端测试通过
  □ Bot 命令列表已在 BotFather 注册
  □ 频道/群组已创建并配置
```

---

## 启发式规则（Heuristics）

1. **"initData 不可信"原则**：永远不要在客户端信任 initDataUnsafe，所有涉及身份、支付、排名的操作必须由服务端验证 initData 的 HMAC-SHA256 签名，并检查 auth_date 时效性。这是 Mini App 安全的第一道防线。

2. **"Stars 优先、TON 补充"原则**：数字商品和服务必须使用 Telegram Stars 支付（iOS/Android 合规要求），链上资产（NFT/Jetton）走 TON Connect。混合模式下，Stars 覆盖大众用户，TON 覆盖 Web3 高级用户。

3. **"2秒首屏"原则**：Mini App 首屏加载必须控制在 2 秒以内。使用骨架屏占位，资源按需加载，图片用 WebP/AVIF，JS 做 Tree Shaking 和代码分割。加载完成后立即调用 `ready()` 隐藏 Telegram 的 loading 界面。

4. **"10秒生死线"原则**：pre_checkout_query 必须在 10 秒内响应 answerPreCheckoutQuery，否则支付自动失败。支付预检查逻辑要极简——只验证库存和资格，复杂逻辑放到 successful_payment 之后。

5. **"链下高频、链上低频"原则**：游戏内高频操作（积分增减、道具使用）全部走链下系统（Redis/PostgreSQL），低频资产操作（提现、NFT铸造）才上链。定期批量上链降低 Gas 成本，避免每次操作都发链上交易。

6. **"双向奖励驱动 K>1"原则**：邀请系统必须同时奖励邀请人和被邀请人。目标是病毒系数 K-factor > 1（每个用户平均带来 > 1 个新用户）。结合 Squad 排名、邀请排行榜、阶梯奖励放大传播效果。

7. **"主题色跟随"原则**：Mini App UI 必须使用 Telegram 的 CSS 变量（`var(--tg-theme-*)`）适配 Light/Dark 主题，禁止硬编码颜色值。监听 themeChanged 事件动态切换，确保与 Telegram 原生界面视觉一致。

8. **"chargeId 必须持久化"原则**：每笔 Stars 支付成功后的 `telegram_payment_charge_id` 必须写入持久化存储（数据库），不能只存内存或缓存。这是退款的唯一凭证，丢失即无法退款，将面临用户投诉和平台处罚。

9. **"分层召回、克制推送"原则**：Bot 消息推送按用户不活跃天数分层（3/7/14/30天），内容个性化且带 Mini App 入口。频率严格控制——过度推送导致用户屏蔽 Bot，等于永久失去该用户。关键通知和营销消息必须区分。

10. **"Webhook 生产、Polling 开发"原则**：生产环境必须使用 Webhook（实时+可靠），配合 Nginx 反向代理和 SSL 证书。开发环境用 Long Polling 快速调试。两种模式互斥，切换时必须先 deleteWebhook。

---

## 参考资源

- [Telegram Bot API 官方文档](https://core.telegram.org/bots/api)
- [Telegram Mini Apps 官方文档](https://core.telegram.org/bots/webapps)
- [Telegram Stars 支付文档](https://core.telegram.org/bots/payments-stars)
- [TON Connect 协议概述](https://docs.ton.org/develop/dapps/ton-connect/overview)
- [TON 开发者文档](https://docs.ton.org/)
- [grammY 框架文档](https://grammy.dev/)
- [Telegraf 框架文档](https://telegraf.js.org/)
- [@tonconnect/ui-react 文档](https://docs.ton.org/develop/dapps/ton-connect/react)
- [tma.js - Telegram Mini Apps SDK](https://docs.telegram-mini-apps.com/)
- [Phaser.js 游戏引擎](https://phaser.io/)
- [Cocos Creator 文档](https://docs.cocos.com/creator/)

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
