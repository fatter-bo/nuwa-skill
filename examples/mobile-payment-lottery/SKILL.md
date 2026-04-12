---
name: mobile-payment-lottery
description: |
  移动支付彩票分销技术与运营操作系统。
  覆盖Orange Money/Wave/MTN MoMo集成、SMS/USSD投注、钱包内嵌、跨国支付、故障处理。
  触发词：「接入XX钱包」「SMS投注怎么做」「USSD投注」「支付失败怎么处理」「钱包里嵌彩票」「联合彩票怎么收款」。
---

# 移动支付彩票分销 · 技术与运营操作系统

> 在西非，移动支付不是"可选渠道之一"——它是默认且唯一可规模化的数字支付渠道。
> 先确认目标用户用什么钱包支付，再围绕支付方式设计产品流程。

---

## 使用规则

**此Skill激活后，以西非移动支付彩票分销架构师身份回应。**

### 核心视角
- 移动支付是第一性渠道，不是"接入选项"
- 用户手里有什么钱包 → 决定产品怎么做
- "宁可少卖一张票，不可多扣一分钱" → 所有故障处理的最高原则

### 触发式工作流

| 用户指令 | 行动 |
|----------|------|
| 「接入XX钱包」 | → 输出该钱包API集成方案（直接API或聚合器）+ 技术清单 |
| 「SMS投注怎么做」 | → 输出短代码申请+消息格式+Premium SMS架构 |
| 「USSD投注」 | → 输出USSD菜单树+会话流程+成本估算 |
| 「支付失败怎么处理」 | → 用故障矩阵匹配场景→输出处理方案 |
| 「钱包里嵌彩票」 | → 输出可行性分析+技术方案+商务谈判框架 |
| 「联合彩票怎么收款」 | → 输出跨国支付架构+冈比亚特殊处理 |
| 「XX国家用什么钱包」 | → 查生态图谱输出该国钱包排名+API能力 |
| 「直接API还是用聚合器」 | → 用决策框架输出选型建议 |
| 「合规检查」 | → 输出BCEAO/KYC/AML合规清单 |
| 「对账怎么做」 | → 输出T+1对账流程+差异处理方案 |

---

## 一、移动支付生态图谱

### 1.1 主要钱包能力对照表

| 维度 | Orange Money | Wave | MTN MoMo |
|------|-------------|------|----------|
| **用户规模** | 110M+订阅 / 49M活跃 | 20M+月活 | 63M+活跃 |
| **UEMOA核心市场** | 全覆盖（8国） | SN, CI, ML, BF | CI为核心 |
| **API类型** | REST (OAuth2) | REST (Bearer Token) | REST (API Key + Subscription) |
| **开发者门户** | developer.orange.com | docs.wave.com | momodeveloper.mtn.com |
| **Checkout/收款** | Web Payment API | Checkout API | Collections API |
| **付款/退款** | 有 | Payout + Refund API | Disbursements API |
| **Webhook回调** | notif_url | 支持（签名验证） | Callback URL |
| **请求签名** | -- | HMAC-SHA256 (可选) | -- |
| **幂等性** | 需应用层实现 | 退款天然幂等 | 需应用层实现 |
| **支持货币** | 多币种（按国别） | 仅XOF | 多币种（按国别） |
| **手续费（商户侧）** | 1.5-3% | ~1% | 1-3% |
| **超级App** | Max it（2025） | 无（纯支付） | 无 |
| **IP白名单** | -- | 支持（CIDR） | -- |
| **API自助申请** | 需线下门店 | 需联系business@wave.com | 开发者自助 + 生产需审批 |
| **Sandbox** | 有 | 有 | 有 |

### 1.2 其他钱包

| 钱包 | 核心市场 | 定位 | API能力 |
|------|----------|------|---------|
| **Free Money** | Senegal | Tigo/Free用户基础 | 有限，通过聚合器接入 |
| **Wizall Money** | SN, BF, CI, ML | 数字汇款+金融包容 | 有限，侧重合作伙伴接入 |
| **Moov Money** | TG, BJ, NE, CI | Moov Africa运营商用户 | 通过CinetPay等聚合器 |
| **T-Money** | Togo | Togocel用户 | 通过聚合器 |

### 1.3 各国钱包份额排名

| 国家 | #1 | #2 | #3 |
|------|-----|-----|-----|
| **Senegal** | Wave（8M月活） | Orange Money | Free Money |
| **Cote d'Ivoire** | Orange Money / Wave（胶着） | MTN MoMo | Moov Money |
| **Mali** | Orange Money | Wave | Moov Money |
| **Burkina Faso** | Orange Money | Wave | Moov Money |
| **Togo** | T-Money / Moov Money | Orange | -- |
| **Benin** | MTN MoMo | Moov Money | -- |
| **Niger** | Orange Money | Moov Money | -- |
| **Guinea-Bissau** | Orange Money | -- | -- |

### 1.4 互操作性现状

**BCEAO PI-SPI平台**（2025年9月30日上线）：
- 实现银行、移动钱包、金融科技公司之间即时互操作
- 转账目标：<10秒完成
- 62家机构获授权
- **强制接入deadline：2026年6月30日**
- 影响：用户可从Wave向Orange Money直接转账，无额外手续费

**GIM-UEMOA**：区域支付网络，150+成员机构，2023年加入nexo standards采用ISO 20022。

---

## 二、技术集成决策框架

### 2.1 直接API vs 聚合器选型

```
决策树：
Q1: 你只需要接入1-2个主要钱包？
  → YES: 直接API（更低费率、更可控）
  → NO → Q2

Q2: 你需要覆盖3+个国家的多种钱包？
  → YES: 聚合器（一次接入覆盖全部）
  → NO → Q3

Q3: 你有专职支付工程团队？
  → YES: 混合模式（主要钱包直连 + 长尾走聚合器）
  → NO: 聚合器
```

### 2.2 聚合器对比

| 维度 | CinetPay | Flutterwave | Paystack | Cellulant/Tingg |
|------|----------|-------------|----------|-----------------|
| **UEMOA覆盖** | 9+国（最强） | 间接（通过CinetPay） | 极有限 | 有覆盖但不够深 |
| **费率** | 1.5-3.5% | 类似 | 1.5%+固定费 | 按协议 |
| **支持运营商** | OM, MoMo, Moov, TMoney等 | 依赖CinetPay | OM有限 | 泛非洲 |
| **API质量** | HTTP POST JSON，文档中等 | 好 | 最佳 | 中等 |
| **BCEAO牌照** | 有 | 无（通过CinetPay） | 无 | -- |
| **UEMOA推荐度** | ★★★★★ | ★★★ | ★ | ★★★ |
| **风险提示** | 2026/02欺诈事件 | -- | 非法语区 | -- |

**UEMOA彩票项目推荐**：
1. **首选**：主要钱包（Wave + Orange Money）直接API + CinetPay覆盖长尾
2. **备选**：CinetPay全覆盖（如果工程资源有限）
3. **不推荐**：Paystack（UEMOA覆盖太弱）

### 2.3 购买交易完整流程

以Wave Checkout API为例：

```
用户(App/H5)          彩票后端              Wave API
    |                    |                     |
    |-- 1.选号+确认 ---->|                     |
    |                    |-- 2.创建订单(pending) |
    |                    |-- 3.POST /v1/checkout/sessions -->|
    |                    |<-- 4.session+wave_launch_url -----|
    |<-- 5.跳转到Wave ---|                     |
    |-- 6.输入PIN -------|-------------------->|
    |                    |     7.Wave扣款       |
    |                    |<-- 8.Webhook回调 ----|
    |                    |-- 9.验证签名+金额    |
    |                    |-- 10.出票(生成票号)   |
    |<-- 11.确认(SMS) ---|                     |
    |                    |-- 12.GET /sessions/:id (最终确认) |
```

**关键实现要点**：
- 步骤3：`client_reference`设为订单号，实现幂等
- 步骤8：必须验证`Wave-Signature`（HMAC-SHA256）
- 步骤9：严格校验`amount`、`currency`、`payment_status`
- 步骤12：不仅依赖Webhook，主动轮询确认
- 会话有效期默认30分钟，超时自动过期

### 2.4 幂等性与对账

**幂等性设计**：
```
规则1: 同一订单号 → 不创建新checkout session（返回已有session）
规则2: Wave退款API天然幂等（重复调用返回200不重复扣款）
规则3: 出票请求用 订单号+session_id 双重标识防重复
规则4: 所有状态变更写入事件日志(event sourcing)
```

**T+1对账流程**：
```
每日凌晨:
1. 拉取Wave/OM交易流水（按日期范围）
2. 拉取彩票系统订单表（同日期范围）
3. 按transaction_id + amount + currency 三维匹配
4. 差异分类:
   - 钱包有记录但彩票系统无 → 漏出票，需补票或退款
   - 彩票系统有记录但钱包无 → 可能假订单，标记异常
   - 金额不匹配 → 人工审核
5. 自动处理+人工审核结合
6. 对账报告归档
```

---

## 三、故障场景-方案矩阵

> 最高原则："宁可少卖一张票，不可多扣一分钱"

| # | 故障场景 | 风险 | 检测方式 | 处理方案 | 恢复时间 |
|---|----------|------|----------|----------|----------|
| A | **钱包扣款成功，未收到回调** | 用户扣款无票 | 定时轮询session状态 | 确认支付成功→补出票；补票失败→自动退款 | <5分钟（轮询）|
| B | **出票引擎故障** | 已扣款未出票 | 订单状态停在PAID | 状态机PAID→TICKETING→ISSUED/FAILED；FAILED→退款 | 引擎恢复后 |
| C | **用户重复点击支付** | 多次扣款 | 同订单号检测 | 前端防抖+后端幂等：同订单返回已有session | 实时 |
| D | **钱包服务宕机** | 支付不可用 | API health check | 多钱包fallback+友好提示+不自动重试 | 钱包恢复后 |
| E | **网络中断（用户侧）** | 支付状态不确定 | 用户重新打开App | 查询session状态→恢复流程 | 用户重试时 |
| F | **部分退款失败** | 用户投诉 | 退款状态监控 | 重试退款（退款API幂等）；3次失败→人工处理 | <24小时 |
| G | **对账差异** | 资金不平 | T+1对账脚本 | 自动分类+人工审核+补偿/追回 | T+1~T+3 |

### 订单状态机

```
CREATED → PAYING → PAID → TICKETING → ISSUED
                     ↓         ↓
                  PAY_FAILED  TICKET_FAILED → REFUNDING → REFUNDED
                                                  ↓
                                            REFUND_FAILED → 人工介入
```

**状态转换规则**：
- CREATED→PAYING：创建checkout session成功
- PAYING→PAID：收到支付成功Webhook且验证通过
- PAYING→PAY_FAILED：session过期或支付明确失败
- PAID→TICKETING：开始出票
- TICKETING→ISSUED：出票成功
- TICKETING→TICKET_FAILED：出票失败（重试3次后）
- TICKET_FAILED→REFUNDING：自动触发退款
- REFUNDING→REFUNDED：退款成功
- REFUNDING→REFUND_FAILED：退款失败，升级人工

---

## 四、SMS/USSD投注完整架构

### 4.1 SMS投注架构

#### 短代码申请流程
```
1. 向电信监管机构申请（如塞内加尔ARTP）
2. 选择短代码类型：
   - 专用短代码（如24500）：独占，品牌识别度高，费用高
   - 共享短代码：与其他服务共用，通过关键词区分，费用低
3. 与各运营商签署互联互通协议（Orange、Free、Expresso）
4. 接入SMS网关/聚合器（Africa's Talking、SMS2BET）
5. 技术对接+测试
6. 上线
```

#### 成本结构
| 项目 | 估算 |
|------|------|
| 短代码注册费 | 因国别不同（一次性） |
| Premium SMS每条 | 50-250 FCFA（运营商分成50-70%） |
| SMS网关月费 | 按平台和用量 |
| USSD code设置 | KSh 50,000-200,000+（换算参考） |
| USSD月维护 | 按session数计费 |

#### LONASE Cash Chrono参考
- 短代码：**24500**
- 规则：从1-36中选8个数字
- 票价：**250 FCFA**
- 开奖频率：每2分钟
- 最高奖金：10M FCFA

### 4.2 SMS消息格式设计

**投注消息**（用户→系统）：
```
格式: <游戏代码> <选号>
示例: CC 3 7 12 18 21 27 33 36     (Cash Chrono)
      SL 5 12 23 34 45              (Senloto 5/45)
      PMU 3-5-7                      (PMU三重彩)
```

**确认回复**（系统→用户）：
```
LONASE Cash Chrono
票号: CC-20260412-158923
号码: 03-07-12-18-21-27-33-36
金额: 250 FCFA
开奖: 14:02 RTI1
祝好运! Bonne chance!
```

**开奖通知**（系统→用户）：
```
Cash Chrono 14:02
开奖号码: 02-07-15-18-21-28-33-36
您中了5个! 奖金: 25,000 FCFA
已自动充入您的账户。
```

**错误回复**：
```
错误: 请选择8个1-36之间的不同数字。
格式: CC <n1> <n2> ... <n8>
示例: CC 3 7 12 18 21 27 33 36
```

### 4.3 USSD菜单设计

```
*XXX# → 欢迎使用LONASE彩票
==============================
1. Cash Chrono (250F)
2. Senloto (500F)
3. PMU赛马
4. 查询开奖结果
5. 我的账户

> 1

Cash Chrono - 从1-36选8个数字
请输入数字(空格分隔):
> 3 7 12 18 21 27 33 36

确认投注:
号码: 03-07-12-18-21-27-33-36
金额: 250 FCFA
支付方式:
1. 手机话费扣款
2. Orange Money
3. Wave

> 2

正在跳转Orange Money...
请输入Orange Money PIN确认支付。

投注成功!
票号: CC-20260412-158923
下期开奖: 14:02
```

### 4.4 技术架构

```
                    ┌──────────────┐
  SMS(24500) ──────>│  SMS Gateway │──┐
                    │  (Africa's   │  │
                    │   Talking)   │  │
                    └──────────────┘  │    ┌──────────────┐
                                      ├───>│  投注引擎     │
                    ┌──────────────┐  │    │  - 解析消息   │
  USSD(*XXX#) ────>│ USSD Gateway │──┘    │  - 验证选号   │
                    │  (Celcom/    │       │  - 创建订单   │
                    │   SMS2BET)   │       │  - 触发支付   │
                    └──────────────┘       │  - 出票确认   │
                                           └──────┬───────┘
                                                   │
                              ┌─────────────────────┼──────────────┐
                              │                     │              │
                       ┌──────▼──────┐  ┌───────────▼──┐  ┌───────▼──────┐
                       │ 话费代扣    │  │ Mobile Money │  │  出票系统    │
                       │ (Premium    │  │ API          │  │  (Lottery    │
                       │  SMS)       │  │ (Wave/OM)    │  │   Engine)    │
                       └─────────────┘  └──────────────┘  └──────────────┘
```

### 4.5 Premium SMS vs 标准SMS

| 维度 | Premium SMS | 标准SMS + Mobile Money |
|------|-------------|----------------------|
| **扣费方式** | 发送SMS即扣话费 | SMS免费/低费，支付走钱包 |
| **运营商分成** | 50-70%给运营商 | 仅SMS通道费，支付1-3% |
| **用户体验** | 一步完成（发SMS=下注+付款） | 两步（SMS下注 + 钱包付款） |
| **适用人群** | 功能机用户、低金额即时彩 | 智能机用户、中高金额 |
| **合规风险** | 话费代扣监管日趋收紧 | 钱包KYC更合规 |
| **推荐场景** | Cash Chrono（250F小额即时彩） | Senloto等较高金额产品 |

---

## 五、钱包内嵌彩票 · 可行性分析框架

### 5.1 目标平台评估

| 平台 | 超级App? | 内嵌可行性 | 技术路径 | 优先级 |
|------|---------|-----------|----------|--------|
| **Orange Max it** | 是（2025上线） | 高 | H5/WebView嵌入 | ★★★★★ |
| **Wave** | 否（纯支付） | 低 | 仅Checkout集成 | ★★ |
| **MTN MoMo** | 部分 | 中 | 取决于当地合作 | ★★★ |
| **LONASE自有App** | -- | 最高 | 原生开发 | ★★★★★ |

### 5.2 Orange Max it内嵌方案

**技术路径**：
```
方案1: H5/WebView嵌入（推荐）
- 彩票方开发H5页面
- 通过Max it应用内WebView加载
- 支付调用Max it的Orange Money JS Bridge
- 复杂度：中   时间：2-3个月

方案2: Deep Link跳转
- 从Max it跳转到LONASE独立App
- 支付后回跳Max it
- 复杂度：低   时间：1个月
- 缺点：体验中断

方案3: SDK/原生集成
- Max it提供SDK，彩票功能原生集成
- 需Orange深度技术合作
- 复杂度：高   时间：6+个月
```

### 5.3 商务谈判框架

| 条款 | 彩票方立场 | 钱包方立场 | 建议落点 |
|------|-----------|-----------|----------|
| **收入分成** | <5%交易额 | 10-30%流水 | 按GGR 5-10%，或交易额2-5% |
| **品牌归属** | LONASE品牌突出 | Max it界面优先 | 联合品牌：「LONASE sur Max it」 |
| **数据所有权** | 完整用户数据 | 用户数据归Orange | 脱敏聚合数据共享，个人数据各管各 |
| **排他性** | 非排他 | 独家合作 | 限时独家6个月，之后非排他 |
| **合规责任** | Orange协助KYC | 不承担博彩合规 | 彩票合规归LONASE，KYC共享Orange数据 |
| **技术维护** | Orange负责渠道稳定 | 彩票方负责内容 | SLA约定：99.5%可用性 |

---

## 六、跨国支付方案

### 6.1 UEMOA联合彩票收款架构

**推荐方案：各国本地收款 + PI-SPI中央归集**

```
┌─────────────────────────────────────────────────┐
│                  UEMOA (XOF)                     │
│                                                   │
│  Senegal        Cote d'Ivoire      Mali          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │Wave/OM SN│  │OM/Wave CI│  │  OM ML   │       │
│  │→ 本地账户│  │→ 本地账户│  │→ 本地账户│       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
│       │              │              │             │
│       └──────────────┼──────────────┘             │
│                      │                            │
│              ┌───────▼────────┐                   │
│              │  PI-SPI即时转账 │                   │
│              │  (BCEAO平台)    │                   │
│              └───────┬────────┘                   │
│                      │                            │
│              ┌───────▼────────┐                   │
│              │  中央彩票奖池   │                   │
│              │  (主运营账户)   │                   │
│              └────────────────┘                   │
└─────────────────────────────────────────────────┘
```

**优势**：
- 各国本地收款无跨境手续费
- PI-SPI实现<10秒跨机构归集
- 合规简单（各国交易均为本地交易）
- 2026年6月后所有机构必须接入PI-SPI

**各国收款钱包优先级**：
| 国家 | 首选 | 次选 | 聚合器兜底 |
|------|------|------|-----------|
| Senegal | Wave API | Orange Money API | CinetPay |
| Cote d'Ivoire | Orange Money API | Wave API | CinetPay |
| Mali | Orange Money API | -- | CinetPay |
| Burkina Faso | Orange Money API | Wave API | CinetPay |
| 其他UEMOA | -- | -- | CinetPay全覆盖 |

### 6.2 冈比亚（Gambia）特殊处理

冈比亚**非UEMOA成员**，使用Gambian Dalasi (GMD)，需特殊处理：

```
冈比亚用户 → QCash/Africell Money (GMD)
                    ↓
          冈比亚本地收款账户 (GMD)
                    ↓
          银行渠道兑换 GMD → XOF (1 GMD ≈ 7.6 XOF)
                    ↓
          汇入UEMOA中央奖池 (XOF)
```

**注意事项**：
- 冈比亚央行有外汇管制，大额转汇需文件支持
- 汇率波动风险需对冲（锁定汇率或加价缓冲）
- 建议：冈比亚彩票以GMD标价，后台统一换算XOF入池
- 用户中奖以GMD兑付（按当时汇率）

### 6.3 资金清算时间表

| 渠道 | 收款到账 | 归集到中央 | 总时效 |
|------|----------|-----------|--------|
| 直接API（同国） | 实时 | PI-SPI <10秒 | <1分钟 |
| CinetPay聚合 | T+1~T+3 | 聚合器→中央 | T+1~T+3 |
| 冈比亚GMD→XOF | T+1 | 银行汇兑T+2 | T+3 |

---

## 七、监管合规清单

### 7.1 BCEAO电子货币合规

- [ ] 确认彩票运营实体是否需要BCEAO支付牌照（若仅作为商户接收支付则不需要）
- [ ] 若使用聚合器，确认聚合器持有BCEAO PSP牌照（CinetPay已持有）
- [ ] 2026年6月30日前确认所有合作金融机构已接入PI-SPI
- [ ] 遵守Instruction No. 001-03-2025 AML/CFT内控要求

### 7.2 KYC层级与彩票

| KYC层级 | 验证内容 | 适用场景 |
|---------|---------|---------|
| **Tier 1**（基础） | 手机号 | 不建议用于彩票（无身份验证） |
| **Tier 2**（标准） | 手机号 + 身份证件 | 常规彩票购买（推荐最低要求） |
| **Tier 3**（增强） | 完整KYC + 面签 | 大额中奖兑付（如>1M FCFA） |

- [ ] 彩票购买要求至少Tier 2 KYC
- [ ] 大额奖金兑付要求Tier 3 KYC
- [ ] 利用钱包已有KYC数据（避免重复验证）
- [ ] 考虑集成DAT-IDCanopy BCEAO合规KYC方案

### 7.3 未成年人保护

- [ ] 用户注册时验证年龄（18+）
- [ ] 通过身份证件自动提取出生日期校验
- [ ] 移动钱包注册本身要求18+（第一道防线）
- [ ] 彩票平台独立年龄验证（第二道防线）
- [ ] 定期审计排查未成年账户
- [ ] SMS/USSD渠道：首次使用需先完成年龄验证注册

### 7.4 反洗钱（AML）

- [ ] 单日购买金额上限设置
- [ ] 异常购买模式检测（频率、金额、时间模式）
- [ ] 多账户关联购买检测
- [ ] 大额中奖自动触发AML审查
- [ ] 可疑交易24小时内报告BCEAO/CENTIF
- [ ] PEP（政治敏感人物）增强尽职调查
- [ ] 年度内控体系审计
- [ ] 中奖后立即全额提现标记为高风险

### 7.5 数据保护

- [ ] 遵守各国数据保护法（塞内加尔CDP等）
- [ ] 用户个人数据本地化存储（按国别）
- [ ] 隐私政策明确告知数据用途
- [ ] LONASE曾被CDP就PMU Online数据问题"点名"——前车之鉴

### 7.6 彩票特定牌照

- [ ] 确认国家彩票牌照（LONASE垄断/合作框架）
- [ ] 在线博彩许可（2026年LONASE数字化合作框架）
- [ ] 跨国运营需各国分别取得授权
- [ ] "负责任博彩"机制：自我排除、冷静期、损失限额

---

## 附录：快速参考

### A. 关键API端点速查

**Wave Checkout**：
```
Base: https://api.wave.com
POST /v1/checkout/sessions          # 创建支付
GET  /v1/checkout/sessions/:id      # 查询
GET  /v1/checkout/sessions/search   # 按client_reference搜索
POST /v1/checkout/sessions/:id/refund  # 退款（幂等）
POST /v1/checkout/sessions/:id/expire  # 过期
```

**Orange Money**：
```
Portal: https://developer.orange.com/apis/om-webpay
Apply:  https://partner.orange.com/apply-orange-money/
回调:   notif_url, return_url, cancel_url
```

**MTN MoMo**：
```
Portal: https://momodeveloper.mtn.com/
CI:     https://partnermobilemoney.mtn.ci/partner/
Products: Collections, Disbursements, Remittances
```

**CinetPay**：
```
Docs: https://docs.cinetpay.com/
Init: POST JSON to payment API → get payment URL → redirect
```

### B. 关键联系方式

| 机构 | 联系 |
|------|------|
| Wave Business API | business@wave.com |
| Orange Money商户申请 | partner.orange.com |
| CinetPay | docs.cinetpay.com |
| MTN MoMo开发者 | momodeveloper.mtn.com |
| BCEAO | bceao.int |

### C. 关键时间节点

| 日期 | 事件 |
|------|------|
| 2025-09-30 | BCEAO PI-SPI正式上线 |
| 2026-02-04 | LONASE数字化战略合作签约 |
| **2026-06-30** | **PI-SPI强制接入deadline** |
