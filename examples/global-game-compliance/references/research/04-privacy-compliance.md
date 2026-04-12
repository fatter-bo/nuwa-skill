# 04 隐私合规调研

调研时间：2026-04-12
调研范围：GDPR/CCPA/PIPA/APPI/PDPA核心差异、隐私政策模板要素、数据合规技术方案

---

## 1. 全球主要隐私法规对比

### 核心差异矩阵

| 维度 | GDPR (EU) | CCPA/CPRA (California) | PIPA (韩国) | APPI (日本) | PDPA (新加坡/泰国) |
|------|-----------|----------------------|-------------|-------------|-------------------|
| 生效时间 | 2018.05 | 2020.01/2023.01 | 2011 (多次修订) | 2003 (2022大改) | 新加坡2012/泰国2022 |
| 同意模式 | **Opt-in**（非必要处理需主动同意） | **Opt-out**（用户有权退出数据销售/共享） | **明确同意**（几乎所有数据收集需详细告知+同意） | 敏感数据opt-in；一般数据可implied | **Deemed consent**可推定同意 + 正当利益例外 |
| 儿童年龄门槛 | 16岁（成员国可降至13岁） | 16岁（需opt-in） | 14岁 | 18岁 | 各国不同（新加坡无明确/泰国10岁） |
| 数据跨境 | 需充分性认定/SCC/BCR | 无特别限制（但需告知） | 需同等保护水平或同意 | 需同等保护或同意 | 需同等保护或同意 |
| DPO要求 | 特定情况强制 | 无 | 特定情况强制 | 无（但建议） | 强制（新加坡）/特定情况（泰国） |
| 处罚上限 | 全球年营收4%或EUR 2000万 | $7,500/故意违规 | KRW 5亿+3%营收 | JPY 1亿 | SGD 100万或10%年营收 |
| 域外效力 | 有 | 有（面向CA居民） | 有 | 有 | 有 |

### 中国《个人信息保护法》(PIPL) 补充

- 2021年11月生效，被称为"中国版GDPR"
- 同意模式：明确同意（敏感信息需单独同意）
- 跨境：需安全评估/标准合同/认证（数据量大的需向CAC申报）
- 处罚：最高全球年营收5%或RMB 5000万
- 对游戏App影响：必须本地化存储中国用户数据；需设置专门的个人信息保护负责人

## 2. 游戏App隐私政策必备要素

### 通用必备条目（覆盖主要法规）

1. **数据控制者身份**：公司名称、注册地址、联系方式
2. **收集的数据类型**：
   - 账户信息（邮箱、用户名、密码哈希）
   - 设备信息（设备ID、OS版本、IP地址）
   - 游戏行为数据（游戏时长、胜负记录、消费记录）
   - 支付信息（通过第三方处理，不直接存储卡号）
   - 社交数据（好友列表、聊天记录）
   - 位置数据（如需geo-fencing）
3. **收集目的**：每类数据对应的具体使用目的
4. **法律基础**：GDPR要求明确（同意/合同/正当利益等）
5. **数据共享**：与哪些第三方共享、目的是什么
6. **数据保留期限**：各类数据保留多长时间
7. **用户权利**：访问/更正/删除/可携带/撤回同意/投诉
8. **儿童保护**：不向未达年龄门槛的儿童收集数据
9. **Cookie/SDK政策**：使用了哪些分析/广告SDK
10. **跨境传输**：数据存储和传输到哪些国家/地区
11. **安全措施**：加密、访问控制等技术保护措施概述
12. **变更通知**：隐私政策变更时如何通知用户

### 各平台特殊要求

| 平台 | 特殊要求 |
|------|----------|
| Apple App Store | 必须填写App Privacy Labels（"营养标签"）；需在App内可访问隐私政策 |
| Google Play | 必须填写Data Safety Section；2026年起强化年龄验证要求 |
| 微信小游戏 | 必须有隐私政策弹窗；需符合PIPL要求；需完成小程序备案 |

## 3. 数据合规技术方案

### 架构层面

#### 数据分区存储（Data Residency）

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  EU Region   │  │  CN Region   │  │  SEA Region  │
│  (Frankfurt) │  │  (Shanghai)  │  │ (Singapore)  │
│  EU用户数据  │  │ 中国用户数据 │  │ 东南亚用户  │
└─────────────┘  └─────────────┘  └─────────────┘
         ↕                ↕                ↕
    GDPR合规         PIPL合规         PDPA合规
```

- 按用户注册地区将个人数据存储在对应地区的服务器
- 游戏逻辑数据（非个人数据）可全球同步
- 个人数据跨区传输需满足对应法规的跨境条件

#### 同意管理平台（CMP）

- 首次启动时根据用户地区展示对应的同意弹窗
- EU用户：需明确opt-in才能启用非必要SDK（分析/广告）
- US/CA用户：可默认启用但提供opt-out选项
- 中国用户：需展示隐私政策并获得明确同意
- 同意记录需持久化存储，作为合规证据

#### 数据最小化

- 仅收集游戏功能必需的数据
- 广告ID收集需明确告知
- 设备指纹收集在iOS 14+/Android 13+受限
- 位置数据：仅在geo-fencing必需时收集精确位置

### SDK合规

#### 常用游戏SDK的隐私影响

| SDK类别 | 示例 | 收集的数据 | 合规注意 |
|---------|------|-----------|----------|
| 分析 | Firebase Analytics, Adjust | 设备ID, 事件, 用户属性 | 需在隐私政策中披露 |
| 广告 | AdMob, Unity Ads, ironSource | IDFA/GAID, 行为数据 | iOS需ATT弹窗; GDPR需同意 |
| 社交 | Facebook SDK, WeChat SDK | 社交图谱, 分享数据 | 需明确告知第三方共享 |
| 支付 | Apple IAP, Google Billing | 交易记录 | 通常由平台处理 |
| 崩溃报告 | Crashlytics, Sentry | 设备状态, 堆栈信息 | 一般属正当利益 |

#### iOS App Tracking Transparency (ATT)

- iOS 14.5+必须在追踪前弹窗获取用户授权
- 未授权时不得使用IDFA
- 影响：广告变现效率下降30-50%（行业平均opt-in率约25-35%）

## 4. 合规检查清单

### 上架前隐私合规检查

- [ ] 隐私政策已发布在可访问的URL
- [ ] 隐私政策覆盖所有目标市场的法规要求
- [ ] App内可访问隐私政策（Settings或首屏链接）
- [ ] Apple Privacy Labels已准确填写
- [ ] Google Data Safety Section已准确填写
- [ ] CMP（同意管理平台）已集成并按地区适配
- [ ] iOS ATT弹窗已实现（如使用IDFA）
- [ ] 儿童保护措施已实现（年龄门槛验证）
- [ ] 数据删除功能已实现（用户可请求删除账户和数据）
- [ ] 数据分区存储已配置（如面向EU/CN用户）
- [ ] 第三方SDK列表已审计，数据流已文档化
- [ ] DPA（数据处理协议）已与所有数据处理者签署

## 5. 来源

- ComplianceHub.Wiki: Privacy Laws Compared CCPA, GDPR, LGPD — https://compliancehub.wiki/privacy-laws-compared-ccpa-gdpr-and-lgpd-compliance-requirements-2025-update/
- SecurePrivacy: Mobile App Privacy Compliance Guide for 2026 — https://secureprivacy.ai/blog/app-privacy-compliance-guide
- Usercentrics: Biggest Data Privacy Issues in 2026 — https://usercentrics.com/knowledge-hub/2025-privacy-challenges-for-app-and-game-publishers/
- Reform.app: GDPR, CCPA, and APAC Consent Rules Compared — https://www.reform.app/blog/gdpr-ccpa-apac-consent-rules-compared
- TrustArc: Navigating APAC Data Privacy Laws — https://trustarc.com/resource/navigating-apac-data-privacy-laws-compliance-survival-guide/
- Appdome: 2026 GDPR & Privacy Laws for Mobile Apps — https://www.appdome.com/dev-sec-blog/gdpr-mobile-app-security-requirements-gdpr-security-requirements/
- SecurePrivacy: GDPR Compliance for Mobile Apps 2026 — https://secureprivacy.ai/blog/gdpr-compliance-mobile-apps
