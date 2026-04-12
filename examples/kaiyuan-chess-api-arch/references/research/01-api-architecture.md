# 维度一：API化架构设计

## 1. 分层API设计体系

### 开元模式的核心架构分层

开元棋牌的API化架构将整个平台拆分为五大API域，每个域独立部署、独立版本管理：

```
┌─────────────────────────────────────────────────────┐
│                 API Gateway (统一入口)                │
│         认证/限流/路由/日志/版本管理                    │
├──────────┬──────────┬──────────┬──────────┬─────────┤
│ 用户系统  │ 游戏大厅  │ 玩法游戏  │ 支付系统  │ 后台管理 │
│ User API │ Lobby API│ Game API │ Pay API  │Admin API│
├──────────┴──────────┴──────────┴──────────┴─────────┤
│              共享基础设施层                            │
│    消息队列 / 缓存集群 / 数据库集群 / 日志系统          │
└─────────────────────────────────────────────────────┘
```

### 各API域职责

**用户系统API (User Service)**
- 注册/登录（支持手机号、第三方OAuth、游客模式）
- 用户Profile管理（昵称、头像、VIP等级）
- Token管理（JWT签发、刷新、多设备登录控制）
- 运营商侧用户同步（第三方运营商将自有用户映射到平台用户）

**游戏大厅API (Lobby Service)**
- 房间列表查询（按玩法/级别/人数筛选）
- 快速匹配（根据段位/资金自动分配房间）
- 锦标赛/活动入口
- 公告/跑马灯/推送通知

**玩法游戏API (Game Service)**
- 每种玩法一个独立Game Service实例
- 标准化接口：加入房间、准备、操作（出牌/加注/过）、离开
- 游戏状态推送（通过WebSocket长连接）
- 牌局记录与回放

**支付系统API (Payment Service)**
- 充值/提现
- 多渠道支付抽象（银行卡、电子钱包、加密货币）
- 账户余额管理（Seamless Wallet模式）
- 交易流水与对账

**后台管理API (Admin Service)**
- 运营商后台（数据统计、用户管理、活动配置）
- 风控面板（异常检测、封禁操作）
- 财务报表（GGR/NGR计算、分成结算）

来源: [Microservices Architecture for iGaming Platforms](https://whitelabelcoders.com/blog/how-do-you-build-microservices-architecture-for-igaming-platforms/)
来源: [Essential APIs for iGaming Integration](https://whitelabelcoders.com/blog/what-apis-are-essential-for-igaming-software-integration/)

## 2. 接口协议规范

### 协议选型矩阵

| 场景 | 协议 | 理由 |
|------|------|------|
| 用户注册/登录/Profile | HTTP REST | 低频请求，标准CRUD，易于对接 |
| 大厅房间列表/活动查询 | HTTP REST | 可缓存、幂等、CDN友好 |
| 游戏内实时操作 | WebSocket | 双向通信、低延迟、服务端主动推送 |
| 服务间内部通信 | gRPC + Protobuf | 高性能、强类型、流式支持 |
| 异步事件（结算/通知） | 消息队列(Kafka/NATS) | 解耦、削峰、保证最终一致性 |

### REST API设计规范

```
基础路径: https://api.platform.com/v1/
认证方式: Bearer Token (JWT)
版本管理: URL路径版本 (/v1/, /v2/)

示例端点:
POST   /v1/auth/login           # 登录
POST   /v1/auth/register        # 注册
GET    /v1/lobby/rooms           # 房间列表
POST   /v1/lobby/match           # 快速匹配
GET    /v1/game/{gameId}/state   # 获取游戏状态
POST   /v1/payment/deposit       # 充值
GET    /v1/admin/reports/ggr     # GGR报表
```

### WebSocket游戏协议

```json
// 客户端→服务端
{
  "type": "action",
  "gameId": "room_12345",
  "action": "play_card",
  "data": { "cards": [{"suit":"hearts","rank":"A"}] },
  "seq": 42,
  "timestamp": 1680000000000
}

// 服务端→客户端
{
  "type": "state_update",
  "gameId": "room_12345",
  "state": {
    "phase": "playing",
    "currentPlayer": "player_3",
    "lastAction": { "player": "player_2", "action": "play_card" },
    "publicCards": [...],
    "pot": 5000
  },
  "seq": 43
}
```

来源: [Building Scalable Real-Time Multiplayer Card Games](https://dev.to/krishanvijay/building-scalable-real-time-multiplayer-card-games-3kn6)
来源: [Casino Server - Node.js + Socket.io](https://github.com/floatinghotpot/casino-server)

## 3. 第三方运营商接入流程

### 典型接入流程（从签约到上线）

```
Step 1: 签约 → 获取 merchant_id + API密钥对 (public_key / secret_key)
Step 2: 对接用户系统 → 运营商通过API同步用户或使用平台用户体系
Step 3: 选择玩法模块 → 从玩法商城选择需要的游戏模块
Step 4: 对接钱包 → Seamless Wallet(实时扣款) 或 Transfer Wallet(预存转入)
Step 5: 前端集成 → iframe嵌入或SDK直接集成
Step 6: 测试环境验证 → sandbox环境全流程测试
Step 7: 上线 → 切换到生产环境
```

### Seamless Wallet vs Transfer Wallet

**Seamless Wallet（无缝钱包）**
- 玩家余额始终在运营商侧
- 每次下注/赢取都实时调用运营商的Debit/Credit API
- 优点：用户体验无缝、余额实时一致
- 缺点：对运营商API性能要求极高

**Transfer Wallet（转账钱包）**
- 玩家进入游戏前将资金转入游戏平台钱包
- 游戏过程中不频繁调用运营商API
- 游戏结束后将余额转回
- 优点：降低API调用频率、游戏内延迟低
- 缺点：用户需手动转账、余额不实时同步

来源: [Seamless API Wallet System: Single vs Transfer Wallets](https://dbgaming-api.com/api-transfer-wallet/)
来源: [Game Aggregation Platform - VeliTech](https://velitech.com/products/game-aggregator/)

## 4. API Gateway设计

### 网关核心功能

- **认证鉴权**：JWT验证 + API Key验证（运营商级别）
- **限流熔断**：基于令牌桶算法，按运营商/用户/IP多维度限流
- **路由分发**：根据URL路径将请求路由到对应微服务
- **协议转换**：外部REST → 内部gRPC
- **日志审计**：所有API调用记录审计日志
- **灰度发布**：基于Header/Cookie的流量分发

### 技术选型参考

| 方案 | 适用场景 | 特点 |
|------|---------|------|
| Kong | 中大型平台 | 插件生态丰富、Lua扩展 |
| APISIX | 高性能需求 | 基于OpenResty、热更新路由 |
| Envoy + Istio | K8s原生 | 服务网格、流量管理 |
| 自研Gateway | 深度定制 | 完全控制、灵活但成本高 |

来源: [Microservices Architecture for iGaming](https://whitelabelcoders.com/blog/how-do-you-build-microservices-architecture-for-igaming-platforms/)
