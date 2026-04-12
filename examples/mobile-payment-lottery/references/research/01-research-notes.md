# 移动支付彩票分销技术与运营 -- 调研笔记

> 调研时间：2026-04-12
> 调研范围：西非移动支付生态、技术集成、故障处理、SMS/USSD投注、钱包内嵌、跨国支付、监管合规

---

## 1. 西非移动支付生态

### 1.1 Orange Money

**覆盖国家**：Mali、Cameroon、Cote d'Ivoire、Senegal、Madagascar、Botswana、Guinea Conakry、Guinea Bissau、Sierra Leone、RD Congo、Central African Republic（共11国+）

**用户规模**：110M+订阅用户（17国），约49M活跃用户，2024年交易额EUR 164B

**API能力**：
- Web Payment / M Payment API（REST）
- 开发者门户：https://developer.orange.com/apis/om-webpay
- 认证方式：OAuth2 Bearer Token
- 回调URL支持：return_url（成功）、cancel_url（取消）、notif_url（Webhook通知）
- 支持国家各自独立的API实例（每个国家单独申请）
- 申请入口：https://partner.orange.com/apply-orange-money/
- 商户须在当地Orange门店注册、完成KYA合规

**支付流程**（4步）：
1. 客户在商户网站选Orange Money支付
2. 客户通过USSD获取临时密码（OTP）
3. 客户在支付页面输入OTP
4. 交易完成，触发notif_url回调

**超级App -- Max it**：
- Orange于2025年推出"Max it"超级应用
- 整合Orange Money转账支付、电商、数字内容、票务
- 覆盖5国首发，计划扩展至Orange运营的全部12个非洲和中东国家
- 非Orange客户也可使用

**手续费**：商户侧通常1.5-3%，因国别和交易量不同而异

**日限额/KYC层级**：
- 按BCEAO规定分层KYC
- Level 1（基础）：日交易限额较低，仅需手机号
- Level 2（标准）：需身份证件，限额提高
- Level 3（增强）：完整KYC，最高限额

> 来源：
> - [Orange Developer - OM WebPay](https://developer.orange.com/apis/om-webpay)
> - [Orange Max it 超级App公告](https://newsroom.orange.com/orange-introduces-its-super-app-max-it-to-simplify-everyday-life-for-people-in-africa-and-the-middle-east/)
> - [Nuvei - Orange Money](https://www.nuvei.com/apm/orange-money)

---

### 1.2 Wave

**核心市场**：Senegal（主导地位）、Cote d'Ivoire（快速增长）、Mali、Burkina Faso、Cameroon等17国

**用户规模**：20M+ 月活用户，150,000+代理网点

**零手续费商业逻辑**：
- 用户P2P转账收取flat 1%手续费（远低于竞争对手）
- 存取款在代理点免费
- 商业模式依赖巨大的交易量和极低的运营成本
- 在塞内加尔已超过Orange Money成为市场份额第一
- 2021年起在科特迪瓦市场份额快速增长

**API能力**：
- 开发者文档：https://docs.wave.com/business
- Checkout API：https://docs.wave.com/checkout
  - POST `/v1/checkout/sessions` -- 创建支付会话
  - GET `/v1/checkout/sessions/:id` -- 查询会话
  - GET `/v1/checkout/sessions/search` -- 按client_reference搜索
  - POST `/v1/checkout/sessions/:id/refund` -- 退款（幂等：重复退款返回200不重复扣款）
  - POST `/v1/checkout/sessions/:id/expire` -- 过期会话
- Payout API：https://docs.wave.com/payout（商户向用户付款）
- B2B Payout API：https://docs.wave.com/b2b/payout/index.html
- Balance API：https://docs.wave.com/balance-api
- Base URL：`https://api.wave.com`

**认证机制**：
- Bearer Token（API Key绑定至Business Wallet）
- 可选请求签名：Wave-Signature header（HMAC-SHA256）
  - 格式：`t={timestamp},v1={signature}`
  - 时间戳校验：过去5分钟内，未来30秒内
- 可选IP白名单（CIDR范围支持）

**Checkout会话对象**：
- 字段：id, amount, currency, checkout_status, payment_status, transaction_id, wave_launch_url, when_created, when_expires（默认30分钟）, when_completed, last_payment_error
- 错误码：insufficient-funds, blocked-account, cross-border-payment-not-allowed, payer-mobile-mismatch

**技术限制**：
- 仅支持XOF货币（文档明确标注）
- 跨境支付受限（cross-border-payment-not-allowed错误码存在）
- API访问非自助——需联系business@wave.com或合作团队
- 签名密钥仅创建时显示一次，丢失需吊销重新创建

> 来源：
> - [Wave Checkout API文档](https://docs.wave.com/checkout)
> - [Wave Business APIs](https://docs.wave.com/business)
> - [Wave Payout API](https://docs.wave.com/payout)
> - [Wave零手续费模式分析](https://triplepundit.com/2025/wave-mobile-money-cote-divoire/)

---

### 1.3 MTN MoMo

**核心市场**：Cote d'Ivoire、Ghana、Cameroon、Uganda等16个非洲国家

**用户规模**：63M+活跃用户

**Open API Program**：
- 开发者门户：https://momodeveloper.mtn.com/
- 科特迪瓦合作者门户：https://partnermobilemoney.mtn.ci/partner/
- API产品：Collections（收款）、Disbursements（付款）、Remittances（汇款）
- Sandbox环境可用
- 第3届Open API Hackathon覆盖14个国家（含科特迪瓦）

> 来源：
> - [MTN MoMo API](https://momo.mtn.com/api/)
> - [MTN MoMo开发者门户](https://momodeveloper.mtn.com/api-documentation)
> - [Ericsson案例 - MTN MoMo Open APIs](https://www.ericsson.com/en/cases/2023/mtn-mobile-money-open-apis)

---

### 1.4 其他钱包

**Free Money（Tigo/Free）**：
- 塞内加尔市场参与者之一
- 用户在多钱包间切换使用（如Wizall→Free Money转账）
- LONASE已将Free Money列为充值方式之一

**Wizall Money**：
- 总部塞内加尔，覆盖Burkina Faso、Cote d'Ivoire、Mali
- 专注数字汇款和金融包容
- 支持WorldRemit等国际汇款合作
- LONASE已将Wizall列为充值方式之一

**Moov Money（Moov Africa）**：
- 西非区域性玩家
- 通过Nuvei等支持商户收款
- 在Togo、Benin、Niger有较强存在

> 来源：
> - [Wizall Money - UNCDF](https://migrantmoney.uncdf.org/resources/insights/switching-from-cash-to-digital-remittances-research-insights-from-wizall-money-in-senegal/)
> - [Moov支付 - Nuvei](https://www.nuvei.com/apm/moov)

---

### 1.5 各钱包按国家份额

| 国家 | 主导钱包 | 第二名 | 其他 |
|------|----------|--------|------|
| Senegal | Wave（#1, 8M月活） | Orange Money | Free Money, Wizall |
| Cote d'Ivoire | Orange Money / Wave（竞争中） | MTN MoMo | Moov Money |
| Mali | Orange Money | Wave | Moov Money |
| Burkina Faso | Orange Money | Wave | Moov Money |
| Togo | Moov Money / T-Money | Orange | -- |
| Benin | MTN MoMo | Moov Money | -- |
| Niger | Orange Money | Moov Money | -- |
| Guinea Conakry | Orange Money | MTN MoMo | -- |

---

### 1.6 互操作性现状

**BCEAO PI-SPI平台**（2025年9月30日正式启动）：
- 实现银行、移动钱包、金融科技公司之间的即时互操作支付
- 目标：10秒内完成跨机构转账
- 62家机构获授权提供公共服务
- **强制deadline**：2026年6月30日前所有金融机构必须完成技术接入
- 覆盖全部8个UEMOA国家
- 用户可从Wave向Orange Money转账、从银行向MoMo转账，无额外手续费

**GIM-UEMOA**：
- 2003年成立的区域支付系统
- 连接150+成员机构（银行、微金融、邮政、金融科技）
- 2023年加入nexo standards，采用ISO 20022
- 支持ATM、POS、电子交易的区域互通

> 来源：
> - [BCEAO PI-SPI启动](https://www.dabafinance.com/en/news/interoperability-bceao-wave-uemoa-2025)
> - [PI-SPI授权机构名单](https://www.financialafrik.com/en/2025/10/06/pi-spi-list-of-62-institutions-authorized-by-bceao-to-offer-services-to-the-public-in-uemoa/)
> - [BCEAO强制接入deadline](https://www.ecofinagency.com/news-finances/0404-54418-bceao-imposes-june-30-deadline-to-complete-instant-payments-integration)
> - [GIM-UEMOA加入nexo standards](https://www.nexo-standards.org/news/gim-uemoa-joins-nexo-standards-simplify-cross-border-payments-across-west-africa)

---

## 2. 技术集成

### 2.1 直接API vs 聚合器

#### CinetPay
- **总部**：Cote d'Ivoire（2016创立）
- **UEMOA覆盖**：Senegal、Cote d'Ivoire、Mali、Burkina Faso、Togo、Benin、Niger + Cameroon、Guinea Conakry、DRC（共9+国）
- **支持货币**：XOF、XAF、GNF、CDF
- **集成方式**：HTTP POST JSON API
- **费率**：1.5%-3.5%（按国别和交易量浮动）
- **支持运营商**：Orange Money、MTN MoMo、Moov Money、TMoney、Airtel Money、Mobicash
- **牌照**：持有BCEAO支付服务提供商牌照，2025年获批加入区域跨境支付系统
- **25,000+商户**，30M+交易处理
- **文档**：https://docs.cinetpay.com/
- **注意**：2026年2月曝出网络欺诈事件，合作方损失USD 1.1M

#### Flutterwave
- CinetPay的投资方之一
- 在西非法语区通过CinetPay间接覆盖
- 直接覆盖：Nigeria、Ghana等英语区为主
- 对UEMOA法语区支持不如CinetPay原生

#### Paystack
- Stripe旗下（被收购）
- 主要市场：Nigeria、Ghana、South Africa
- 费率：1.5% + NGN100（本地），3.9% + NGN100（国际）
- **UEMOA覆盖极有限**——不适合作为西非法语区主选
- 60,000+商户

#### Cellulant（Tingg）
- 泛非洲支付网关，35+国家
- 150M+消费者
- 聚合所有支付方式
- 对UEMOA有覆盖但不如CinetPay深度本地化

#### PawaPay
- 专注非洲移动支付
- 20个非洲市场
- 提供对账、FX、结算
- 侧重技术可靠性和交易追溯

> 来源：
> - [CinetPay文档](https://docs.cinetpay.com/)
> - [CinetPay概览](https://docs.cinetpay.com/api/1.0-en/introduction/overview)
> - [CinetPay欺诈事件](https://weetracker.com/2026/02/02/cinetpay-1m-debt-cyber-fraud/)
> - [Flutterwave投资CinetPay](https://techcrunch.com/2021/12/08/4dx-ventures-and-flutterwave-back-francophone-africas-cinetpay-in-2-4m-round/)
> - [Paystack定价](https://paystack.com/pricing)
> - [Cellulant/Tingg](https://www.cellulant.io/)
> - [PawaPay](https://www.pawapay.io/)

---

### 2.2 购买交易完整流程

以Wave Checkout API为例的完整彩票购买流程：

```
用户（App/H5）          彩票后端              Wave API
    |                      |                     |
    |-- 1.选号+确认 ------>|                     |
    |                      |-- 2.创建订单(pending) |
    |                      |-- 3.POST /v1/checkout/sessions -->|
    |                      |<-- 4.返回session(wave_launch_url) |
    |<-- 5.跳转wave_launch_url --|                |
    |-- 6.用户在Wave App输入PIN -->|              |
    |                      |     |-- 7.Wave扣款 -->|
    |                      |<-- 8.Webhook回调(notif_url) -----|
    |                      |-- 9.验证签名+金额     |
    |                      |-- 10.出票(生成票号)   |
    |<-- 11.出票确认(SMS/Push) ---|               |
    |                      |-- 12.GET /sessions/:id(最终确认) |
```

**关键节点**：
- 步骤3：创建checkout session时传入`client_reference`（订单号），用于幂等性和对账
- 步骤8：Webhook回调需验证Wave-Signature（HMAC-SHA256）
- 步骤9：严格校验金额、货币、session状态
- 步骤12：主动查询确认（不仅依赖Webhook）

---

### 2.3 幂等性与对账

**幂等性**：
- Wave退款API天然幂等（重复退款返回200不重复扣款）
- 创建session建议使用唯一`client_reference`标识
- 后端应用层：订单号+支付session_id双重关联，防重复

**对账机制**：
- T+1对账：每日拉取Wave/Orange Money交易流水 vs 彩票系统订单
- 对账字段：transaction_id、amount、currency、when_completed
- 差异处理：多扣→退款，少扣→补收，未匹配→人工审核
- 建议使用聚合器（如CinetPay）时也保留直接对账能力

> 来源：
> - [Wave Checkout API文档](https://docs.wave.com/checkout)
> - [支付幂等性与对账研究](https://www.researchgate.net/publication/380285760_Idempotency_and_Reconciliation_in_Payment_Software)

---

## 3. 故障处理

### 核心原则："宁可少卖一张票，不可多扣一分钱"

### 3.1 典型故障场景

**场景A：钱包扣款成功但彩票系统未收到回调**
- 原因：网络中断、服务器宕机、Webhook投递失败
- 风险：用户被扣款但没收到票
- 处理：
  1. 定时轮询Wave GET `/v1/checkout/sessions/:id`确认支付状态
  2. 补偿出票（确认支付成功后补发）
  3. 若补偿出票失败→自动退款
  4. T+1对账兜底

**场景B：出票请求未确认（彩票系统内部故障）**
- 原因：出票引擎宕机、数据库写入失败
- 风险：扣了款但票号未生成
- 处理：
  1. 订单状态机：PAID→TICKETING→ISSUED / TICKET_FAILED
  2. TICKET_FAILED状态触发自动退款
  3. 退款前再次确认是否实际已出票（防止重复出票+退款）

**场景C：重复点击支付**
- 原因：用户等待超时后重复点击
- 风险：创建多个checkout session，多次扣款
- 处理：
  1. 前端防抖（disable按钮）
  2. 后端幂等：同一订单号不创建新session
  3. 若已存在pending session→返回已有session的wave_launch_url
  4. 若已存在completed session→直接返回出票结果

**场景D：钱包服务宕机**
- 原因：Wave/Orange Money平台维护或故障
- 风险：支付请求超时、用户体验差
- 处理：
  1. 实时监控各钱包API可用性（health check）
  2. 多钱包fallback（Wave不可用→提示用户切换Orange Money）
  3. 展示友好错误信息+预计恢复时间
  4. 支付请求排队，恢复后不自动重试（避免用户不知情被扣款）

> 来源：
> - [移动支付交易失败分析](https://medium.com/@arabaaddaquay/why-mobile-money-transactions-fail-and-what-the-data-reveals-0e8aef002cc0)
> - [支付失败重试逻辑](https://medium.com/@mallikarjunpasupuleti/how-to-handle-payment-failures-and-retry-logic-in-your-application-ab878e2d8849)

---

## 4. SMS/USSD投注

### 4.1 LONASE Cash Chrono模式

**产品详情**：
- 游戏类型：即时彩票（Loto Instantane）
- 规则：从1-36中选8个数字
- 票价：250 FCFA
- 短代码：24500
- 开奖频率：每2分钟一次（电视直播）
- 最高奖金：10M FCFA
- 渠道：手机SMS发送至24500

**SMS投注格式**（推断）：
```
发送到: 24500
格式: CC <n1> <n2> <n3> <n4> <n5> <n6> <n7> <n8>
示例: CC 3 7 12 18 21 27 33 36
回复: "票号:XXX 选号:3-7-12-18-21-27-33-36 金额:250F 下期开奖:14:02"
```

### 4.2 短代码申请

**流程**：
1. 向各国电信监管机构（如塞内加尔ARTP）申请短代码
2. 与运营商（Orange、Free、Expresso）签署互联互通协议
3. 接入SMS网关（如Africa's Talking Premium SMS API）
4. 测试+上线

**成本参考**：
- 短代码注册费：因国别不同
- Premium SMS每条收费：通常50-250 FCFA（运营商分成50-70%）
- 月维护费：按平台和运营商不同

**关键供应商**：
- Africa's Talking：Premium SMS API，支持内容变现
- SMS2BET：专注非洲彩票/博彩的SMS/USSD方案
- Dusane Gaming：SMS和USSD彩票平台
- Celcom Africa：USSD和短代码服务

### 4.3 USSD菜单设计

```
*XXX# → LONASE彩票
  1. Cash Chrono (250F)
  2. Senloto (500F)
  3. 查询开奖结果
  4. 查询余额/中奖
  5. 我的投注记录

选择1 → Cash Chrono
  请输入8个数字(1-36)，用空格分隔:
  > 3 7 12 18 21 27 33 36

  确认投注:
  号码: 03-07-12-18-21-27-33-36
  金额: 250 FCFA
  1. 确认支付(从手机余额扣款)
  2. 取消

  > 1
  投注成功! 票号: CC-20260412-0001
  开奖时间: 14:02
  祝好运!
```

**USSD vs SMS对比**：
| 维度 | SMS | USSD |
|------|-----|------|
| 交互模式 | 单条消息 | 会话式菜单 |
| 需要智能手机 | 否 | 否 |
| 实时性 | 异步（可能延迟） | 实时会话 |
| 设置成本 | 较低 | 较高（USSD code + 聚合器） |
| 费用承担 | Premium SMS用户付费 | 通常免费或极低费用 |
| 用户体验 | 需记忆格式 | 引导式操作 |
| 支付 | 运营商代扣（话费） | 运营商代扣或钱包扣款 |

> 来源：
> - [LONASE Cash Chrono](https://www.lejecos.com/Loterie-La-LONASE-met-sur-le-marche-Cash-Chrono-un-nouveau-jeu-de-loto-instantane_a7250.html)
> - [SMS2BET非洲博彩方案](https://sms2bet.com/)
> - [Dusane Gaming SMS/USSD平台](https://www.dusanegaming.com/sms-and-ussd/)
> - [Africa's Talking Premium SMS](https://africastalking.com/sms/premiumsms)
> - [USSD投注技术](https://helloduty.com/blogs/ussd-technology-in-betting)
> - [Celcom Africa USSD/短代码](https://celcomafrica.com/ussd-short-codes)

---

## 5. 钱包内嵌彩票服务

### 5.1 "超级App"模式

**Orange Max it**：
- 2025年推出的超级App
- 已整合Orange Money、电商、数字内容、票务
- 理论上可以内嵌彩票服务作为"数字内容"或"票务"品类
- 覆盖5国首发（含Senegal、Cote d'Ivoire）

**Wave**：
- 目前是纯支付工具，非超级App定位
- 但有Business API允许商户集成checkout
- 不提供小程序或SDK内嵌能力（截至2026年4月）

### 5.2 技术方案

| 方案 | 适用场景 | 复杂度 | 用户体验 |
|------|----------|--------|----------|
| H5嵌入（WebView） | Orange Max it内嵌 | 中 | 较好，但受WebView性能限制 |
| Mini-Program | 若Orange开放小程序平台 | 高 | 最佳原生体验 |
| SDK集成 | 双方深度技术合作 | 最高 | 最佳 |
| 深度链接(Deep Link) | 从钱包跳转到独立App | 低 | 体验中断 |

### 5.3 商务条款框架

| 条款 | 彩票方诉求 | 钱包方诉求 | 谈判要点 |
|------|-----------|-----------|----------|
| 收入分成 | 最低分成（<5%） | 10-30%流水分成 | 按GGR(毛博彩收入)还是按交易额 |
| 品牌归属 | 彩票品牌露出 | 钱包主界面品牌优先 | 联合品牌 vs 白标 |
| 数据所有权 | 获取完整用户数据 | 用户数据归钱包 | 脱敏聚合数据可共享 |
| 排他性 | 非排他（多钱包接入） | 独家合作 | 限时独家（6-12个月） |
| 合规责任 | 由彩票运营方承担 | 不承担博彩合规责任 | 明确责任边界 |

> 来源：
> - [Orange Max it公告](https://www.orange.com/en/press-release/orange-introduces-its-super-app-max-it-to-simplify-everyday-life-for-people-in-africa-and-the-middle-east-233801)
> - [Orange Money在线赌场使用](https://www.gamblerspick.com/payments/orange-money-online-casinos-r502/)

---

## 6. 跨国支付

### 6.1 UEMOA同币种XOF框架

**优势**：8个UEMOA国家（Senegal、Cote d'Ivoire、Mali、Burkina Faso、Togo、Benin、Niger、Guinea-Bissau）使用统一XOF货币，无汇率风险。

**BCEAO支付基础设施**：
- **PI-SPI平台**（2025年9月启动）：即时跨机构支付，涵盖银行、移动钱包、金融科技
- **GIM-UEMOA**：区域支付网络，150+成员机构
- **RTGS**：BCEAO大额实时清算系统

### 6.2 联合彩票收款架构

**方案A：各国分别收款→中央清算**
```
Senegal用户 → Orange Money SN → LONASE SN账户 ──┐
CdI用户 → Wave CI → LONASE CI账户 ──────────────┤→ 中央清算账户 → 奖池
Mali用户 → Orange Money ML → LONASE ML账户 ──────┘   (T+1汇总)
```
优势：各国本地收款无跨境手续费，合规简单（各国本地交易）
劣势：需在每个国家开设商户账户，资金归集延迟

**方案B：统一收款→各国分配**
```
所有UEMOA用户 → CinetPay统一网关 → 中央运营账户 → 各国分配
```
优势：单一接入点，技术简单
劣势：跨境交易可能触发额外合规要求，费率可能更高

**推荐**：方案A更适合，因为PI-SPI已实现低成本跨机构转账，各国本地收款避免跨境监管复杂性。

### 6.3 冈比亚特殊处理

- **非UEMOA成员**，使用Gambian Dalasi (GMD)，非XOF
- 1 GMD ≈ 7.6 XOF
- 移动钱包：QCash (QCell)、Africell Money
- **方案**：
  1. 在冈比亚本地以GMD收款
  2. 通过银行渠道按市场汇率兑换为XOF
  3. 或通过国际汇款通道（WorldRemit等）以XOF结算
  4. 需额外应对冈比亚中央银行的外汇管制要求

> 来源：
> - [Senegal支付架构](https://www.transfi.com/blog/senegals-payment-rails-how-they-work---gim-uemoa-wave-mobile-money-instant-transfers)
> - [BCEAO PI-SPI](https://www.financialafrik.com/en/2025/09/30/uemoa-new-era-with-bceaos-interoperable-instant-payment-system-pi-spi/)
> - [Gambia支付概览](https://payatlas.com/countries/gambia-gm)
> - [GIM-UEMOA](https://www.nexo-standards.org/news/gim-uemoa-joins-nexo-standards-simplify-cross-border-payments-across-west-africa)

---

## 7. 监管合规

### 7.1 BCEAO电子货币监管

- **Instruction No. 001-03-2025**：强化AML/CFT内控要求
- 电子货币发行机构需BCEAO牌照
- 支付服务提供商需单独牌照（CinetPay已持有）
- PI-SPI接入强制deadline：2026年6月30日
- 分层KYC框架：
  - Tier 1：基础账户（手机号验证），低限额
  - Tier 2：标准账户（身份证件），中等限额
  - Tier 3：增强账户（完整KYC+面签），高限额

### 7.2 彩票支付KYC要求

- 彩票购买属于敏感交易类别
- 建议至少Tier 2 KYC（身份证件验证）
- 大额中奖兑付需Tier 3 KYC
- DAT-IDCanopy方案：BCEAO合规的KYC/KYB解决方案（2025年5月推出）
  - 支持远程开户
  - 自动化身份验证
  - 符合BCEAO严格认证要求

### 7.3 未成年人验证

- BCEAO KYC要求包含年龄验证
- 移动钱包注册通常需18岁+
- 额外措施：
  - 彩票平台独立年龄验证（生日+身份证号校验）
  - 高风险交易触发增强验证
  - 定期审计未成年账户
- 技术方案：通过身份证件自动提取出生日期，2秒内完成年龄校验

### 7.4 反洗钱

- 可疑交易24小时内报告
- 年度内控体系审计
- PEP（政治敏感人物）增强尽职调查
- 彩票特定风险：
  - 大额频繁购买（可能洗钱）
  - 多账户关联购买
  - 中奖后立即全额提现
- 监控措施：
  - 单日购买金额上限
  - 异常购买模式检测
  - 大额中奖自动触发AML审查

### 7.5 LONASE数字化合规

- 2026年2月与LonaseWay、Lonase 22bet、Lonase Partner签署战略合作
- 目标：合规的在线博彩平台（符合国家垄断要求）
- 强调透明度、个人数据保护、负责任博彩
- CDP（数据保护委员会）曾就个人数据问题"点名"LONASE PMU Online

> 来源：
> - [BCEAO KYC合规方案](https://techafricanews.com/2025/05/06/dat-and-idcanopy-launch-bceao-compliant-kyc-kyb-solution/)
> - [塞内加尔KYC合规指南](https://blog.voveid.com/kyc-compliance-in-senegal-2025-guide-for-fintechs-and-regulated-businesses/)
> - [科特迪瓦KYC合规指南](https://blog.voveid.com/kyc-compliance-in-cote-divoire-2025-guide-for-regulated-businesses/)
> - [BCEAO外汇新规](https://www.gide.com/en/news-insights/nouvelle-reglementation-des-changes-en-zone-uemoa-une-reforme-discrete-qui-renforce-le-controle-de-la-bceao-et-accentue-les-complexites-administratives-dans-la-region/)
> - [LONASE数字化战略合作](https://www.seneweb.com/en/news/Video/digital-transformation-lonase-seals-a-strategic-alliance-to-boost-online-gaming_n_482485.html)
> - [BCEAO银行与移动钱包互联](https://www.financialafrik.com/en/2025/09/28/banks-and-mobile-money-finally-connected-bceao-simplifies-payments/)

---

## 8. 关键数据汇总

| 指标 | 数值 | 来源年份 |
|------|------|----------|
| Orange Money全球用户 | 110M+订阅 / 49M活跃 | 2024 |
| Orange Money交易额 | EUR 164B | 2024 |
| Wave月活用户 | 20M+ | 2025 |
| Wave代理网点 | 150,000+ | 2025 |
| MTN MoMo活跃用户 | 63M+ | 2025 |
| CinetPay商户数 | 25,000+ | 2025 |
| CinetPay交易处理 | 30M+ | 2025 |
| PI-SPI授权机构 | 62家 | 2025 |
| GIM-UEMOA成员 | 150+ | 2025 |
| LONASE Cash Chrono票价 | 250 FCFA | -- |
| Cash Chrono最高奖金 | 10M FCFA | -- |
| XOF/GMD汇率 | 1 GMD ≈ 7.6 XOF | 2026 |
