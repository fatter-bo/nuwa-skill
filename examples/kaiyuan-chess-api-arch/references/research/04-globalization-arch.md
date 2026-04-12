# 维度四：全球化技术架构

## 1. 多地区部署架构

### 分区域独立部署模型

棋牌游戏对延迟极度敏感（出牌操作需<200ms响应），因此必须就近部署。

```
┌─────────────────────────────────────────────────────────┐
│                    全局管控层 (Global)                    │
│  全局用户注册中心 │ 全局配置中心 │ 跨区数据同步 │ DNS路由  │
├──────────┬────────────────┬────────────────┬────────────┤
│ 亚太区域  │  欧洲区域       │ 北美区域       │ 拉美区域   │
│ (新加坡)  │  (法兰克福)     │ (弗吉尼亚)    │ (圣保罗)   │
│          │                │               │            │
│ ┌──────┐ │ ┌──────┐      │ ┌──────┐      │ ┌──────┐   │
│ │完整   │ │ │完整   │      │ │完整   │      │ │完整   │   │
│ │服务栈 │ │ │服务栈 │      │ │服务栈 │      │ │服务栈 │   │
│ │+数据库│ │ │+数据库│      │ │+数据库│      │ │+数据库│   │
│ └──────┘ │ └──────┘      │ └──────┘      │ └──────┘   │
└──────────┴────────────────┴────────────────┴────────────┘
```

### 部署策略选择

| 策略 | 描述 | 适用场景 |
|------|------|---------|
| 单区域集中部署 | 所有服务部署在一个数据中心 | 起步阶段、单一市场 |
| 多区域独立部署 | 每个区域独立完整服务栈 | 跨国运营、合规要求 |
| 边缘节点+中心架构 | 游戏服务边缘部署、管理服务集中 | 追求极低延迟 |
| 混合云部署 | 核心数据私有云、弹性计算公有云 | 合规+弹性需求 |

来源: [Building Scalable iGaming Platforms](https://whitelabelcoders.com/blog/how-do-you-build-scalable-igaming-platforms/)
来源: [iGaming Software Development - Intellias](https://intellias.com/igaming-software-development/)

## 2. 多语言支持架构

### i18n架构设计

```
├── 后端多语言
│   ├── 错误信息国际化（错误码+i18n映射）
│   ├── 推送通知多语言模板
│   └── 邮件/短信模板多语言
│
├── 前端多语言
│   ├── UI文本（JSON语言包 → CDN按需加载）
│   ├── 游戏内文本（内嵌在游戏资源包中）
│   └── 动态内容（活动/公告 → CMS多语言版本）
│
└── 数据多语言
    ├── 玩法名称/描述（数据库多语言字段）
    └── 用户生成内容（昵称/聊天 → 实时翻译或不翻译）
```

### 语言包管理

```json
// 语言包结构 (en-US.json)
{
  "lobby": {
    "title": "Game Lobby",
    "quick_match": "Quick Match",
    "room_list": "Room List"
  },
  "game": {
    "your_turn": "Your Turn",
    "fold": "Fold",
    "call": "Call",
    "raise": "Raise",
    "all_in": "All In"
  },
  "payment": {
    "deposit": "Deposit",
    "withdraw": "Withdraw",
    "balance": "Balance"
  }
}
```

## 3. 多币种与多支付渠道抽象层

### 支付抽象架构

```
┌──────────────────────────────────────┐
│           Payment Gateway API         │
│  统一接口：deposit / withdraw / query │
├──────────────────────────────────────┤
│         Payment Router (路由层)       │
│  根据地区/币种/金额选择最优渠道       │
├─────┬──────┬───────┬───────┬────────┤
│Alipay│WeChat│Stripe │UPI    │Crypto  │
│支付宝│微信   │信用卡 │印度UPI │USDT/BTC│
└─────┴──────┴───────┴───────┴────────┘
```

### 多币种钱包设计

```typescript
interface MultiCurrencyWallet {
  userId: string;
  balances: CurrencyBalance[];
  
  deposit(amount: Money): Promise<Transaction>;
  withdraw(amount: Money): Promise<Transaction>;
  convert(from: Currency, to: Currency, amount: number): Promise<ConvertResult>;
  getBalance(currency: Currency): number;
}

interface Money {
  amount: number;       // 统一用最小单位（分/聪）存储
  currency: Currency;   // ISO 4217货币代码
}

// 汇率服务
interface ExchangeRateService {
  getRate(from: Currency, to: Currency): number;
  // 游戏内统一使用"游戏币"作为中间货币
  toGameCoins(money: Money): number;
  fromGameCoins(coins: number, targetCurrency: Currency): Money;
}
```

### 支付编排（Payment Orchestration）

支付编排是iGaming平台管理多支付渠道的关键模式：
- 智能路由：根据地理位置、币种、历史成功率自动选择最优支付渠道
- 失败重试：首选渠道失败后自动切换备用渠道
- 统一对账：所有渠道交易归一化后统一对账

来源: [iGaming Payment Systems Explained](https://fluidpayments.io/articles/igaming-payment-systems-explained-integration-optimization)
来源: [Payment Orchestration for iGaming](https://primer.io/blog/payment-orchestration-for-igaming)
来源: [CryptoProcessing - iGaming Payment Solutions](https://cryptoprocessing.com/learn/igaming-payment-solutions)

## 4. 延迟优化策略

### 网络层优化

| 策略 | 实现方式 | 效果 |
|------|---------|------|
| 就近接入 | CDN + 边缘节点WebSocket接入 | RTT降低50-80% |
| 协议优化 | WebSocket替代HTTP轮询 | 减少握手开销 |
| 数据压缩 | Protobuf替代JSON | 包体减少60-80% |
| 连接复用 | 多游戏共享WebSocket连接 | 减少连接数 |
| 预测与回滚 | 客户端预测+服务端校正 | 感知延迟降低 |

### 边缘计算节点

```
用户 → 边缘WebSocket节点(就近) → 消息总线 → 游戏服务(区域中心)
                                              ↓
用户 ← 边缘WebSocket节点(就近) ← 消息总线 ← 游戏结果
```

边缘节点只负责WebSocket连接管理和消息转发，不含业务逻辑，可以部署在全球40+PoP点。

### 分区匹配策略

- 同区域玩家优先匹配，保证对局内所有玩家延迟接近
- 跨区域匹配仅在同区域玩家不足时启用
- 显示延迟指示器，让玩家感知网络状况

来源: [Building Secure Backend for Sports Betting](https://symphony-solutions.com/insights/building-secure-backend-for-sports-betting)
来源: [SDLC Corp - iGaming Software Development](https://sdlccorp.com/services/games/igaming-software-development/)
