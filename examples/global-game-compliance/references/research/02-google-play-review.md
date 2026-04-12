# 02 Google Play审核规则调研

调研时间：2026-04-12
调研范围：Real Money Gambling政策、Simulated Gambling政策、内容分级、开发者账号验证

---

## 1. Real Money Gambling政策

### 核心定义

Google Play将Real Money Gambling定义为：用户投入真实货币（包括加密货币）以获取不确定结果（奖金/奖品）的活动。

### 适用类别

- 在线赌场
- 体育博彩
- 彩票
- 每日幻想体育（DFS）
- 扑克（真金）

### 上架要求

| 要求 | 详细说明 |
|------|----------|
| 合法牌照 | 必须持有目标市场的合法赌博牌照 |
| 地区限制 | 仅可在牌照覆盖地区上架 |
| 年龄分级 | 必须标记为AO（Adult Only）或IARC等效分级 |
| 年龄验证 | 2026年1月28日起，必须集成Google的age-screening工具 |
| 免费下载 | 必须以免费App形式发布 |
| 申请表 | 需填写Google Play的Gambling Application Form |
| 合规标识 | App内必须包含负责任博彩信息和自我排除选项 |

### 允许上架Real Money Gambling的市场（部分）

澳大利亚、比利时、加拿大、哥伦比亚、丹麦、芬兰、法国、德国、爱尔兰、日本、墨西哥、新西兰、挪威、罗马尼亚、西班牙、瑞典、英国、美国（部分州）

## 2. Simulated Gambling政策

### 核心规则

Google Play政策明确声明：Apps must not provide simulated gambling content。但这里的"simulated gambling"特指那些模仿真金赌博体验但不涉及真金的App（即social casino）。

### Social Casino的灰色地带

| 类型 | Google Play态度 | 注意事项 |
|------|----------------|----------|
| Social Casino（纯虚拟筹码） | 允许，但分级限制 | 不得暗示可赢取真金 |
| Sweepstakes Casino（双币制） | 2025年10月起禁止广告投放 | 被重新分类为赌博 |
| 棋牌游戏（无赌博元素） | 允许 | 确保无真金回流通道 |
| 棋牌游戏（含虚拟赌注） | 需审慎设计 | 可能触发更高分级 |

### 2025年重大政策变更

- Sweepstakes Casino（双币制赌场）被从Social Casino类别中移除
- 不再允许通过"社交赌场认证"（lighter certification）投放广告
- 必须获得完整的赌博牌照才能运营

## 3. 内容分级（IARC）

### IARC分级体系

Google Play采用International Age Rating Coalition (IARC)分级体系，一次填写问卷即可获得多地区分级：

| IARC问卷关键问题 | 影响 |
|-----------------|------|
| "Does this app simulate gambling?" | 触发更高年龄分级 |
| "Does this app contain loot boxes?" | 部分地区（如德国USK）会提高分级 |
| "Does this app involve real money?" | 触发AO/赌博分类 |

### 各地区分级对照

| 地区/组织 | Simulated Gambling分级 | Real Gambling分级 |
|-----------|----------------------|-------------------|
| ESRB（北美） | T (13+) 或 M (17+) | AO (18+) |
| PEGI（欧洲） | 12+ 或 18+ | 18+ |
| USK（德国） | 12+ 或 16+ | 18+ |
| ClassInd（巴西） | 14+ 或 18+ | 18+ |
| ACB（澳大利亚） | M (15+) | R18+ |
| GRAC（韩国） | 15+ 或 18+ | 18+ |

## 4. 开发者账号验证（2025-2026更新）

### 个人账号

- 注册费：$25（一次性）
- 需提供：身份证件、电话号码、电子邮件
- 身份验证：需上传政府颁发的ID

### 组织账号

- 注册费：$25（一次性）
- 必须提供D-U-N-S号码
- 需提供：商业注册文件、营业执照等
- D-U-N-S上的名称必须与商业注册文件一致

### 2026年重大变更

- 2026年9月起，部分地区要求所有App（含非Google Play分发）的开发者必须完成验证
- 开发者需提供个人详细信息和D-U-N-S号码
- 目标是打击恶意软件和欺诈性App

## 5. 常见违规与处罚

### Gambling App常见违规

| 违规类型 | 说明 | 后果 |
|----------|------|------|
| 无牌照运营 | 未持有目标市场赌博牌照 | 立即下架 |
| 地区越界 | 在未授权地区可访问 | 下架 + 警告 |
| 年龄验证缺失 | 未集成年龄筛选工具 | 下架 |
| 虚假分级 | IARC问卷填写不实 | 下架 + 账号处罚 |
| 推广不当 | 向未成年人推广赌博内容 | 严厉处罚 |

## 6. 来源

- Google Play Help: Real-Money Gambling, Games, and Contests — https://support.google.com/googleplay/android-developer/answer/9877032
- Google Play Help: Common violations for gambling apps — https://support.google.com/googleplay/android-developer/answer/13381106
- Google Play Help: Developer Program Policy — https://support.google.com/googleplay/android-developer/answer/16549787
- Gummicube: Google Play Developer Policy Changes & Real Money Gambling — https://www.gummicube.com/blog/google-play-developer-policy-changes-real-money-gambling
- Blockchain-Ads: Running Gambling Ads on Google in 2026 — https://www.blockchain-ads.com/post/gambling-ads-google
- Biometric Update: Google unveils identity verification rules — https://www.biometricupdate.com/202508/google-unveils-identity-verification-rules-for-android-app-developers
