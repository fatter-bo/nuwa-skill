# 01 Apple App Store全球审核规则调研

调研时间：2026-04-12
调研范围：App Store Review Guidelines gambling/simulated gambling条款、各国区差异、被拒原因、申诉策略

---

## 1. 核心条款解读

### Guideline 5.3 — Gaming, Gambling, and Lotteries

Apple将博彩/赌博类应用分为以下子类：

| 子条款 | 适用范围 | 核心要求 |
|--------|----------|----------|
| 5.3.1 | 非即时抽奖（如彩票） | 需列明奖品和获奖概率 |
| 5.3.2 | 即时抽奖（如刮刮卡） | 同上 + 即时兑奖流程透明 |
| 5.3.3 | 慈善/公益类抽奖 | 需证明公益资质 |
| 5.3.4 | 真金博彩（Real Money Gambling） | 需合法牌照 + 限制上架地区 + 必须免费下载 + 年龄验证 |

### Guideline 4.3 — Design: Spam

棋牌/小游戏高频被拒条款：
- 4.3(a): 与其他开发者提交的App二进制/元数据/概念相似，仅有细微差异
- 4.3(b): 同一开发者提交多个功能集相同的App，仅内容/语言不同
- 核心判定标准：是否具有独特的用户体验和实质性差异

### Guideline 3.1.1 — In-App Purchase（抽卡/开箱）

- 所有包含随机虚拟物品购买机制（loot box）的App必须在购买前向用户公示获得每种物品的概率
- 此规则全球适用，不分国家区域
- 2017年底Apple正式加入此条款，主要受中国法规推动

### Guideline 4.7 — HTML5 Mini Apps/Mini Games

- 2024年起Apple对寄宿型小游戏平台（如超级App内的小游戏）有新规
- 要求每个小游戏都需符合App Store审核指南
- 直接影响微信/支付宝等超级App内嵌小游戏的合规

## 2. 各国区审核差异

### 年龄分级

| 分级 | Simulated Gambling含量 | 适用场景 |
|------|----------------------|----------|
| 12+ | 偶尔/轻度模拟赌博 | 含基础棋牌玩法、无付费开箱 |
| 17+ | 频繁/强烈模拟赌博 | 含付费开箱、类赌场视觉体验 |

### 地区限制

- **中国大陆**：真金博彩App完全禁止；所有游戏需有版号（ISBN）
- **日本**：gacha（扭蛋）系统受《景品表示法》约束，complete gacha已被禁止
- **韩国**：概率公示为法律强制要求（2025年修订的《游戏产业促进法》）
- **德国**：2021年《州际赌博条约》实施后，对模拟赌博分类更严格
- **澳大利亚**：含付费loot box的游戏必须分级为M（15+），模拟赌场类必须R18+
- **沙特阿拉伯/UAE**：所有赌博（含模拟）严格禁止，需geo-block
- **巴西**：2024年底通过新博彩法，正在逐步开放，但规则仍在完善中

## 3. 棋牌游戏常见被拒原因

### 高频被拒Guideline排名

1. **4.3 Design: Spam** — 棋牌游戏同质化严重，最常见被拒原因
2. **5.3.4 Gambling** — 被误判为真金赌博或虚拟货币体系设计触发审核红线
3. **3.1.1 IAP** — 虚拟筹码/货币未使用Apple IAP系统
4. **2.1 Performance** — App崩溃或功能不完整
5. **1.1.6 False Info** — 截图/描述与实际内容不符

### 棋牌游戏被判为Gambling的红线

- 虚拟货币可直接或间接兑换为真实货币
- 存在真金充值+提现闭环
- 牌桌有"赌注"概念且与金额挂钩
- 胜负结果影响用户可提现余额
- 邀请码/代理系统暗示线下资金流转

## 4. 申诉策略

### 4.3 Spam申诉框架

1. **差异化论证**：列举本App与被比较App的具体差异（玩法规则、AI策略、社交功能、美术风格）
2. **独特价值陈述**：说明目标用户群的特殊需求
3. **技术差异证明**：提供代码架构差异说明（非同一模板/引擎直出）
4. **文化差异论证**：不同地区棋牌规则天然不同（如麻将就有数百种地方规则）

### 5.3 Gambling误判申诉框架

1. **明确声明**：App不涉及真金（No Real Money）
2. **货币体系说明**：虚拟货币只能购买不能提现，无任何回流通道
3. **合规设计证据**：截图展示"仅供娱乐"提示、无提现入口
4. **年龄分级合理性**：说明已选择适当年龄分级

### 申诉渠道

- Resolution Center（App Store Connect内）
- Appeal页面：https://developer.apple.com/contact/request/app-review/appeal/
- 电话申诉（紧急情况）：部分地区支持直接致电App Review团队

## 5. 来源

- Apple Developer: App Review Guidelines — https://developer.apple.com/app-store/review/guidelines/
- Fenwick: Apple Now Requires Disclosure of Loot Box Odds — https://www.fenwick.com/insights/publications/apple-now-requires-disclosure-of-loot-box-odds
- ShopApper: Fix Apple Gambling App Rejection (Guideline 5.3) — https://shopapper.com/fix-apple-gambling-app-rejection-guideline-5-3/
- Gummicube: How Casino Apps Stay Compliant — https://www.gummicube.com/blog/how-casino-apps-stay-compliant
- MACHash: A 2025 Guide to App Store Laws and Casino Apps — https://machash.com/special/383606/2025-guide-to-app-store-laws-casino-apps/
- The App Launchpad: iOS App Store Review Guidelines 2026 — https://theapplaunchpad.com/blog/app-store-review-guidelines
