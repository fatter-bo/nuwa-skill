# 06 实操上架指南调研

调研时间：2026-04-12
调研范围：开发者账号注册、税务处理、支付渠道接入、截图描述本地化

---

## 1. 开发者账号注册

### Apple Developer Program

| 项目 | 个人账号 | 组织账号 |
|------|----------|----------|
| 年费 | $99/年 | $99/年 |
| D-U-N-S号 | 不需要 | 需要 |
| 法人实体 | 不需要 | 需要（营业执照等） |
| App发布名 | 个人名字 | 公司名称 |
| 团队管理 | 仅自己 | 支持多角色团队 |
| App Transfer | 支持 | 支持 |
| 适用场景 | 个人开发者/独立游戏 | 工作室/公司 |

#### 各国注册注意事项

- **中国大陆**：组织账号需有大陆注册的公司 + 企业D-U-N-S号；个人账号需中国身份证
- **美国**：最简单的注册流程；LLC/Corporation均可
- **日本**：需要日本法人或个人在日本有地址
- **韩国**：需韩国事业者登录证
- **欧盟**：任一成员国注册的法人均可
- **东南亚**：各国流程差异大，建议通过当地合作伙伴

### Google Play Console

| 项目 | 个人账号 | 组织账号 |
|------|----------|----------|
| 注册费 | $25（一次性） | $25（一次性） |
| D-U-N-S号 | 不需要 | **需要** |
| 身份验证 | 政府ID | 商业注册文件 + D-U-N-S |
| 2026新规 | 需完成开发者验证 | 需完成开发者验证 |

#### D-U-N-S号获取

- 免费申请：通过Dun & Bradstreet官网 (https://www.dnb.com)
- 处理时间：免费通道约30天；付费加急约1-5个工作日
- 注意：D-U-N-S上的公司名称必须与营业执照完全一致

### 微信小游戏

| 项目 | 要求 |
|------|------|
| 微信开放平台账号 | 需要 |
| 小程序备案 | 2023年9月起强制 |
| ICP备案 | 需要（网站域名） |
| 版号（ISBN） | 涉及付费/变现的游戏需要 |
| 主体要求 | 企业主体（个人主体功能受限） |
| 游戏类目资质 | 需上传《网络文化经营许可证》或版号证明 |

#### 中国游戏版号（ISBN）申请

- 申请主体：需持有《互联网出版许可证》(IPP)的出版单位代为申请
- 审批周期：6-12个月
- 审批机关：国家新闻出版署（NPPA）
- 成本：版号代理费 + 审核费，通常RMB 5-20万
- 2025年上海试点：外商投资企业开发的游戏可享受"国产游戏"待遇

## 2. 税务处理

### Apple App Store税务

#### 非美国开发者的W-8BEN/W-8BEN-E

| 项目 | W-8BEN（个人） | W-8BEN-E（实体） |
|------|---------------|-----------------|
| 适用对象 | 非美国个人 | 非美国公司/组织 |
| 有效期 | 签署日起3年 | 签署日起3年 |
| 默认预扣税率 | 30%（无税收协定时） | 30%（无税收协定时） |
| 填写位置 | App Store Connect → 协议、税务和银行 | 同左 |

#### 重要澄清

- **App销售收入**：非美国开发者在App Store/Book Store的应用销售收入**不受**美国预扣税影响（不需要在Part III填写税收协定）
- **广告收入**：来自美国来源的广告收入可能受预扣税影响
- **订阅收入**：与App销售相同，一般不受预扣
- 仍需提交W-8BEN/W-8BEN-E以证明外国身份

#### 各国税收协定对预扣税率的影响（适用于广告/版税收入）

| 国家 | 标准预扣税率 | 协定优惠税率 |
|------|------------|-------------|
| 无协定 | 30% | 不适用 |
| 中国 | 30% | 10% |
| 日本 | 30% | 0% |
| 韩国 | 30% | 10% |
| 英国 | 30% | 0% |
| 德国 | 30% | 0% |
| 新加坡 | 30% | 0% |（需在W-8BEN中声明协定条款）
| 印度 | 30% | 15% |

#### 2026年4月爱尔兰VAT变更

- Apple Distribution International Ltd不再作为出口商
- 在爱尔兰设立且注册VAT的实体需完成爱尔兰税务表

### Google Play税务

- 与Apple类似，需在Google Payments Center完成税务信息填写
- 非美国开发者同样需提交W-8BEN/W-8BEN-E
- Google会根据美国来源的收入比例代扣税款

### 各国增值税/消费税

| 国家/地区 | 数字服务税/增值税 | 责任方 |
|-----------|-----------------|--------|
| 欧盟（MOSS） | 各国VAT税率（17%-27%） | 平台代收代缴 |
| 日本 | 消费税10% | 平台代收代缴 |
| 韩国 | VAT 10% | 平台代收代缴 |
| 澳大利亚 | GST 10% | 平台代收代缴 |
| 印度 | GST 18% | 平台代收代缴 |
| 巴西 | 各州税率不同 | 复杂（需本地顾问） |
| 东南亚 | 各国不同（6%-12%） | 多数平台代收 |

**好消息**：Apple和Google在大多数国家承担了增值税代收代缴义务，开发者通常不需要在每个国家单独注册税号。

## 3. 支付渠道接入

### Apple IAP（In-App Purchase）

- **强制性**：所有数字内容/虚拟物品必须通过Apple IAP
- **分成**：标准30%；小型企业计划（年收入<$100万）15%；订阅第二年起15%
- **2024年起DMA变更**：欧盟地区允许替代支付方式（但Apple仍收取佣金）

### Google Play Billing

- **强制性**：数字内容/虚拟物品必须通过Google Play Billing
- **分成**：标准30%；前$100万收入15%（Play Media Experience Program）
- **第三方支付试点**：部分市场允许第三方支付（佣金降至26%或11%）

### 微信支付（小游戏内）

- **分成**：微信平台抽成约40%（包含渠道费+平台费）
- **2026年激励**：新发布首款小游戏可享受零分成窗口期（最高5000万激励池）
- **安卓/iOS差异**：iOS端虚拟物品购买仍需走Apple IAP

### 第三方支付方案（网页/H5游戏）

| 渠道 | 适用地区 | 费率 |
|------|----------|------|
| Stripe | 全球（40+国家） | 2.9% + $0.30/笔 |
| PayPal | 全球 | 3.49% + 固定费用 |
| Xsolla | 游戏专用，200+支付方式 | 5% |
| 本地钱包（GCash/OVO/GoPay等） | 东南亚 | 1-3% |
| M-Pesa | 非洲 | 0.5-2% |

## 4. 截图描述本地化

### 截图最佳实践

#### 技术要求

| 平台 | 必须尺寸 | 最大数量 | 格式 |
|------|----------|----------|------|
| App Store | 6.9" iPhone + 13" iPad（2024年起新要求） | 10张/本地化 | JPEG/PNG, RGB, 无透明, ≤10MB |
| Google Play | 至少2张，推荐8张 | 8张/类型 | JPEG/PNG, 16:9或9:16 |
| 微信小游戏 | 按微信要求 | 5张 | PNG/JPG |

#### 本地化策略（不仅仅是翻译）

| 维度 | 做法 | 示例 |
|------|------|------|
| 文字翻译 | 截图上的文案翻译为目标语言 | 英→日→韩→繁中 |
| 文化适配 | 角色外观/场景适配目标文化 | 日本市场用和风元素 |
| 货币显示 | 使用目标市场的货币符号 | $→¥→₩→€ |
| 色彩偏好 | 适配目标市场的色彩心理 | 红色在中国=吉祥，在西方可能=警告 |
| 功能优先级 | 不同市场突出不同功能 | 日本突出社交功能，美国突出竞技功能 |

### ASO元数据本地化

| 字段 | App Store | Google Play | 注意事项 |
|------|-----------|-------------|----------|
| 标题 | 30字符 | 30字符 | 含核心关键词 |
| 副标题 | 30字符 | 不适用 | 补充关键词 |
| 短描述 | 不适用 | 80字符 | 高权重ASO字段 |
| 关键词 | 100字符（隐藏） | 不适用 | 不要重复标题词 |
| 长描述 | 4000字符 | 4000字符 | Google Play前3行最关键 |

### 关键市场本地化优先级

| 优先级 | 市场 | 语言 | 预期回报 |
|--------|------|------|----------|
| P0 | 美国 | English (US) | 最大单一市场 |
| P0 | 中国 | 简体中文 | 需版号；微信渠道 |
| P0 | 日本 | 日语 | 高ARPU市场 |
| P1 | 韩国 | 韩语 | 高ARPU + 棋牌文化 |
| P1 | 英国/澳洲 | English (UK) | 与US共享但需文化调整 |
| P1 | 德国 | 德语 | 欧洲最大市场 |
| P2 | 巴西 | 巴西葡萄牙语 | 增长快，ARPU低 |
| P2 | 东南亚 | 多语言 | 增长市场，需精细运营 |
| P2 | 中东 | 阿拉伯语 | RTL排版 + 文化敏感 |

## 5. 来源

- Apple Developer: Provide tax information — https://developer.apple.com/help/app-store-connect/manage-tax-information/provide-tax-information/
- Accountably: Form W-8BEN Foreign Status Certificate Guide 2025 — https://accountably.com/irs-forms/fw8ben/
- Kerem Erkan: W-8BEN-E instructions for Apple and Google — https://keremerkan.dev/posts/w-8ben-e-instructions-apple-google/
- Google Play Help: Required information for Play Console developer account — https://support.google.com/googleplay/android-developer/answer/13628312
- Global Link Consulting: How to register DUNS for Google Developer Account in 2026 — https://globallinkconsulting.sg/en/article/duns-registration/duns-for-google-developer
- AppInChina: How to Get a WeChat Mini Program Filing — https://appinchina.co/blog/how-to-get-a-wechat-mini-program-filing-the-complete-guide/
- PocketGamer: How to get your game published in China in 2025 — https://www.pocketgamer.biz/how-to-get-your-game-published-in-china-in-2025/
- Niko Partners: Game regulations in China — https://nikopartners.com/game-regulations-in-china-everything-you-need-to-know/
- SplitMetrics: App Store Screenshot Sizes 2025 — https://splitmetrics.com/blog/app-store-screenshots-aso-guide/
- AppTweak: Complete guide to app store localization — https://www.apptweak.com/en/aso-blog/guide-to-app-store-localization
