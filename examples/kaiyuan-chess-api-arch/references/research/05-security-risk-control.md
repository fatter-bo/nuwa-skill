# 维度五：风控与安全体系

## 1. 防机器人/防共谋/防刷分系统

### 多层反作弊架构

```
┌──────────────────────────────────────────────┐
│ Layer 1: 实时检测层 (Real-time Detection)      │
│  行为模式分析 │ 操作频率监控 │ IP/设备指纹     │
├──────────────────────────────────────────────┤
│ Layer 2: 准实时分析层 (Near Real-time)        │
│  关联图谱分析 │ 资金流向追踪 │ 胜率异常检测    │
├──────────────────────────────────────────────┤
│ Layer 3: 离线深度分析层 (Offline Analysis)     │
│  大数据挖掘 │ ML模型训练 │ 历史回溯审计       │
└──────────────────────────────────────────────┘
```

### 防机器人策略

| 维度 | 检测方法 | 指标 |
|------|---------|------|
| 操作模式 | 出牌时间分布分析 | 人类操作有随机延迟，机器人操作时间过于稳定 |
| 决策模式 | 策略一致性检测 | 机器人策略过于"完美"或过于"固定" |
| 设备指纹 | 浏览器/设备特征采集 | 同一设备指纹多账号、模拟器特征 |
| 行为验证 | 不定期弹出验证码/互动验证 | 长时间在线不休息 |
| 元信息 | IP地理位置 + 注册信息交叉验证 | 批量注册特征 |

### 防共谋检测

共谋(Collusion)是棋牌游戏最严重的作弊形式——多个账号在同一牌桌配合。

检测方法：
- **同桌频率分析**：两个账号在同一牌桌出现的频率是否异常高
- **资金单向流动**：A持续输给B（洗分/转移资金）
- **操作时序关联**：多个账号操作时间间隔过于规律（同一人操控）
- **IP/设备关联**：共享IP、相同设备指纹、WiFi相同
- **策略互补分析**：A在B有强牌时总是弃牌（信息共享）

```typescript
interface CollusionDetector {
  // 计算两个玩家的共谋嫌疑分数
  calculateSuspicion(playerA: string, playerB: string): SuspicionScore;
  
  // 分析一桌中所有玩家组合的共谋可能
  analyzeTable(tableId: string): CollusionReport;
  
  // 基于历史数据的关联图谱
  buildRelationGraph(timeRange: TimeRange): PlayerGraph;
}

interface SuspicionScore {
  coOccurrenceScore: number;   // 同桌频率得分 (0-100)
  fundFlowScore: number;       // 资金单向流动得分
  timingCorrelation: number;   // 操作时序相关性
  deviceCorrelation: number;   // 设备/IP关联度
  strategyCorrelation: number; // 策略互补度
  overallRisk: "low" | "medium" | "high" | "critical";
}
```

来源: [Casino Software Security - Gamingsoft](https://www.gamingsoft.com/blog/2026/03/casino-software-security/)

## 2. 资金安全（入金出金风控）

### 入金风控

- **限额管控**：单笔限额、日限额、月限额（可按VIP等级差异化）
- **来源验证**：实名制要求充值账户与注册账户一致
- **异常检测**：短时间大额充值、凌晨异常时段充值
- **反洗钱(AML)**：大额交易上报、可疑交易标记、PEP(政治敏感人物)筛查

### 出金风控

```
提现请求 → 基础校验（余额/限额/实名）
         → 风控规则引擎
           ├── 低风险（<阈值、老用户）→ 自动放行
           ├── 中风险（新用户、首次提现）→ 人工审核
           └── 高风险（异常模式、关联账号）→ 冻结+调查
         → 支付渠道执行
         → 到账确认
```

### 关键风控规则

- 充值后未游戏直接提现 → 拒绝（防洗钱）
- 流水不足倍数 → 限制提现（投注流水要求）
- 短时间内多次小额提现 → 触发审核（拆分提现）
- 新注册24小时内大额提现 → 强制人工审核

来源: [iGaming Regulatory Compliance](https://whitelabelcoders.com/blog/how-do-you-ensure-regulatory-compliance-in-igaming-software/)

## 3. RNG认证与第三方审计

### RNG系统设计

随机数生成器是棋牌平台公平性的基石。

**CSPRNG（密码学安全的伪随机数生成器）实现要求：**
- 使用经过验证的算法：AES-CTR-DRBG 或 HMAC-SHA256-DRBG
- 熵源收集：硬件噪声(/dev/urandom) + 系统事件时间 + 网络包间隔
- 种子定期轮换：每N局或每T时间重新播种
- 输出不可预测性：即使知道前N个输出，也无法预测第N+1个

### 可验证公平性（Provably Fair）

```
发牌前：
  server_seed = random()                    # 服务端种子（隐藏）
  server_seed_hash = SHA256(server_seed)    # 种子哈希（公开给玩家）
  client_seed = player_input()              # 玩家种子（玩家提供）

发牌：
  result = HMAC_SHA256(server_seed, client_seed + ":" + nonce)
  cards = mapToCards(result)                # 将哈希映射为发牌结果

牌局结束后：
  reveal(server_seed)                       # 公开服务端种子
  玩家可验证: SHA256(server_seed) == server_seed_hash  ✓
  玩家可复现: HMAC_SHA256(server_seed, client_seed + ":" + nonce) == result  ✓
```

### 认证标准与机构

| 机构 | 标准 | 覆盖范围 |
|------|------|---------|
| GLI (Gaming Laboratories International) | GLI-19 | RNG标准、电子表格游戏 |
| eCOGRA | eCOGRA Standards | RNG测试、支付审计、玩家保护 |
| BMM Testlabs | BMM Standards | RNG、游戏数学、系统完整性 |
| Gaming Associates | ISO/IEC 17025 | RNG测试、RGS认证 |
| iTech Labs | iTech Labs RNG | RNG、游戏测试 |

### RNG测试内容

- **统计随机性测试**：NIST SP 800-22测试套件（单比特频率、块内频率、游程测试等16项）
- **分布均匀性**：卡方检验确认每种牌出现概率均匀
- **序列独立性**：连续结果之间无统计相关性
- **不可预测性**：前向不可预测 + 后向不可追溯
- **实现审计**：源码审查确认算法正确实现

来源: [RNG in iGaming - Gaming Associates](https://gamingassociates.com/blog/randomness-in-igaming/)
来源: [RNG Testing & Certification](https://gamingassociates.com/random-number-generator-rng/)
来源: [Provably Fair Randomness - Chainlink](https://chain.link/article/provably-fair-randomness)
来源: [RNG Certification in iGaming](https://key2law.com/en/news/what-is-rng-certification-in-igaming)

## 4. 通信安全与数据保护

### 传输层安全

- 全链路TLS 1.3加密
- WebSocket连接使用WSS协议
- 服务间通信使用mTLS（双向TLS）
- API签名：HMAC-SHA256签名防篡改

### 数据安全

- 敏感数据（密码/支付信息）加密存储（AES-256）
- PII数据脱敏（日志中手机号/身份证号脱敏）
- 数据库备份加密
- 定期渗透测试

### 客户端安全

- 游戏逻辑零泄露：客户端不包含任何游戏逻辑/输赢计算
- 前端thin client模式：所有关键计算在服务端完成
- WebSocket协议加密：防止抓包逆向
- 客户端完整性校验：防止篡改客户端代码

来源: [Remote Gaming Server Certification](https://gamingassociates.com/blog/remote-gaming-server-rgs-certification/)
来源: [Game Certification in Regulated iGaming](https://gamingassociates.com/press-release/game-certification-in-regulated-igaming/)
