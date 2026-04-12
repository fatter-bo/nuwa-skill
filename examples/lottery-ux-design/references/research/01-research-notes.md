# 彩票与博彩产品 UI/UX 设计——调研笔记

> 调研日期: 2026-04-12
> 调研目标: 为西非（塞内加尔/LONASE）彩票数字化运营提供 UX 设计决策依据
> 覆盖范围: 彩票购买流程、刮刮乐交互、SMS/USSD、移动支付、透明度、负责任博彩

---

## 1. 彩票购买流程 UX

### 1.1 选号界面设计

**核心发现**：
- 现代彩票 App 的核心是直觉化的号码选择界面，需同时服务随意玩家和频繁用户
- 老用户应能在 10 秒内完成查结果或买票操作；新用户需要引导提示和教程
- 手机端数字网格必须拇指友好——按钮间距和可读性优先于信息密度
- 杂乱的数字网格是最大 UX 问题，需优先保证间距和可读性

**5/90 大号码池特殊挑战**：
- 加纳/尼日利亚的 5/90 格式要求从 90 个号码中选 5 个
- Ghana NLA 通过运营商短码（MTN Mobile Money、Vodafone Cash）实现手机投注
- 移动端号码生成器提供"点击球选号"的交互模式
- 关键优化：分区显示（如 1-30/31-60/61-90 三页）比一次显示 90 个格子更可用

**购买确认信息**：
- 确认步骤必须包含：日期、开奖期次、所选号码、金额
- 应避免模糊的行动按钮和隐藏条款
- 透明度是信任建设的核心

**来源**：
- [Lotto UX & UI Design Tips | ux.bet](https://ux.bet/lotto-ux-ui-design-tips/)
- [Building the ultimate Android lottery app | AndroidHeadlines](https://www.androidheadlines.com/2025/03/building-the-ultimate-android-lottery-app-ux-security-and-monetization-trends.html)
- [The Role of UX Design in Lottery Website Development | Medium](https://a-mxicoders.medium.com/the-role-of-ux-design-in-lottery-website-development-3deb17d93b76)
- [NLA 5/90 Mobile Code | GWS Online GH](https://www.ghanawebsolutions.com/serve.php?id=3014&d=NLA-590-Mobile-Code)
- [Online Ghana Lotto NLA 590 Mobile Game | 590Mobile](https://www.590mobile.com.gh/)

---

## 2. 在线刮刮乐/即开票交互

### 2.1 刮开动画设计

**触摸交互**：
- 用户通过点击拖动（桌面）或手指滑动（触屏）"揭示"虚拟面板下的符号
- 移动刮刮乐利用触屏能力创造真实的刮擦感，这种参与感是桌面点击无法复制的
- 最佳实践：刮开效果需在触屏上无需额外配置即可响应

**动画与反馈**：
- 每个细节都经过精心设计——声音、动画、反馈都旨在保持用户参与
- 揭示时刻可以是滑块、点击、剥离或动态动画
- 移动端优化动画需要即时反馈：闪光、金币爆炸、发光效果
- 关键：强轮廓在小尺寸下清晰可读，高对比度确保在强光屏幕下可识别

### 2.2 心理设计

**情绪节奏编排**：
- 优秀的设计编排从第一次刮到最终揭示的情绪节奏
- 这些游戏基于行为科学、视觉心理学、奖励序列和品牌识别
- 每张中奖卡都是布局、节奏、期待和优化 UX 的刻意组合
- 揭示区域大小、数量、顺序都影响用户情绪体验

**低带宽适配**：
- 资源预加载是关键
- CSS + Canvas 实现比视频更节省带宽
- 需要提供跳过动画的"快速模式"

**来源**：
- [The Secret Psychology Behind Winning Scratch Cards | Lucky Lady Games](https://luckyladygames.com/the-secret-psychology-behind-winning-scratch-cards-how-great-design-drives-play-excitement-and-revenue/)
- [Online Scratch Cards Explained | Game Wisdom](https://game-wisdom.com/general/online-scratch-cards-explained-mechanics-strategy-play)
- [Scratch Card Game Using HTML/CSS/JS | SourceCodester](https://www.sourcecodester.com/javascript/17721/scratch-card-game-using-html-css-and-javascript-source-code.html)

---

## 3. SMS/USSD 投注系统

### 3.1 USSD 菜单设计最佳实践

**字符限制**：
- USSD 屏幕最大支持 182 字符，但建议控制在 160 字符内（各运营商差异）
- 肯尼亚 Safaricom: 160 字符；Airtel: 184 字符
- 尼日利亚: 160 字符限制

**会话超时**：
- 总会话时长约 180 秒
- 尼日利亚: 总时长 120 秒，空闲超时 30 秒
- 一般范围: 90-180 秒

**菜单设计规则**：
- 每屏不超过 5-7 个选项；超过 5 个造成认知过载
- 用户是扫描而非阅读，错误选择或放弃概率随选项增加
- 功能机小屏幕会截断长菜单；即使智能手机上 USSD 也是基础文本对话框
- USSD 菜单最好仅接受数字输入（多数用户使用功能机）
- 语言选择应在首屏；选择后跨会话记忆，无需每次重选

**为什么选择 USSD/SMS**：
- 非洲 60% 用户使用功能机，智能手机主要集中在大城市
- USSD 可用于下注、查看赔率、存取款
- SMS2BET 等平台整合 SMS、USSD、Web、WAP、App 多渠道

### 3.2 LONASE 塞内加尔系统

**已确认信息**：
- LONASE 是塞内加尔唯一负责组织彩票和体育博彩的机构
- LONASE 已通过 lonase.bet 平台将 Senloto Jackpot 搬上线
- 产品线：赛马投注、国家彩票、足球、轮盘、即开票
- 虚拟产品：虚拟赛马、虚拟足球、灰狗赛跑（通过投注终端和移动应用）
- 移动 App：Lonase App 提供安全的 PMU 投注、国家彩票和赌场游戏

**未确认信息**：
- Cash Chrono 具体的 SMS 投注格式和 24500 短码的详细操作流程在公开资料中较少
- 需要实际测试或联系 LONASE 确认具体消息格式

**来源**：
- [USSD Menu Design: 10 Best Practices | Arkesel](https://arkesel.com/ussd-menu-design-best-practices/)
- [USSD Technology in Betting | HelloDuty](https://helloduty.com/blogs/ussd-technology-in-betting)
- [USSD character limit | Africa's Talking Help](https://help.africastalking.com/en/articles/1284096-what-is-the-character-limit-for-ussd-menus-and-user-input-in-kenya)
- [USSD session duration Nigeria | Africa's Talking Help](https://help.africastalking.com/en/articles/2298298-how-long-is-the-duration-of-a-ussd-session-for-nigerian-telcos)
- [SMS and USSD Platform | Dusane Gaming](https://www.dusanegaming.com/sms-and-ussd/)
- [Lottery, sports betting solutions | SMS2BET](https://sms2bet.com/)
- [LONASE makes Senloto available online | FocusGN](https://focusgn.com/africa/senegals-national-lottery-operator-makes-senloto-jackpot-available-online)
- [Lonase App | PMU Online](https://www.pmuonline.sn/lonaseapp)
- [UX Best Practices for Feature Phone Apps | Medium/WorldCover](https://medium.com/worldcover-insurance/ux-best-practices-for-feature-phone-apps-c14d1cda18c3)

---

## 4. 信任与透明度 UI

### 4.1 概率信息展示

**关键发现**：
- 清晰的中奖概率说明让用户在花钱前能评估可能结果
- 用简单语言分享概率数据时，用户对其财务决策感到更安全
- 概率信息的一致准确性和易验证性能逐步增强用户信任
- 透明的赔率报告是建立长期信任的关键

### 4.2 认证标准

**WLA（世界彩票协会）**：
- 两个国际标准：WLA 安全控制标准（WLA-SCS）和 WLA 负责任博彩框架（WLA-RGF）
- 认证帮助获得监管认可、赢得玩家信任、提高品牌忠诚度
- RNG 必须满足：随机性、不可预测性、可审计性、可重复性、透明性、来源不可否认性

**GLI（Gaming Laboratories International）**：
- 提供信息安全和彩票运营流程的第三方审计和认证
- 针对领先的行业最佳实践标准进行评估

**来源**：
- [Analyzing Odds Transparency in Regulated Online Lottery Platforms | Onjira](https://www.onjira.com/analyzing-odds-transparency-in-regulated-online-lottery-platforms/)
- [WLA Industry Standards](https://world-lotteries.org/services/industry-standards/responsible-gaming-framework/framework)
- [GLI Lottery Compliance & Certification](https://gaminglabs.com/services/professional-services/lottery-gaming-practice/)
- [WLA-SCS:2020 Lottery Certification | SGS](https://www.sgs.com/en-us/services/wla-scs-2020-lottery-certification)

---

## 5. 移动支付 UX

### 5.1 Orange Money 支付流程

**流程细节**：
- 用户在商户网站选择 Orange Money 支付
- 通过 Orange Money USSD 服务用密钥请求临时密码（OTP）
- 在支付页面输入临时密码完成支付
- 用户将手机钱包绑定到手机号，无需银行卡即可快速处理

**覆盖市场**：
- 马里、喀麦隆、科特迪瓦、塞内加尔、马达加斯加等
- 支持 XAF（中非法郎）和 XOF（西非法郎）交易

### 5.2 Wave——颠覆者

**核心数据**：
- 塞内加尔月活 800 万用户
- 转账费率 1%，存取款免费——比竞品便宜约 70%
- 迫使 Orange Money 和 MTN Money 跟进降费到 1%

**UX 创新**：
- 绕过 USSD 垄断，采用 App-to-App + QR 码支付
- 比传统 USSD 拨号/SMS 系统更视觉化、更直观
- 技术路线：基于 App 的 QR 码，由 Agent 扫码完成交易

**启示**：
- Wave 的成功证明 App 支付可以在非洲市场超越 USSD
- 但仍需要为功能机用户保留 USSD/话费扣款渠道
- 彩票平台应同时支持 Wave（App 用户）和 Orange Money/话费扣款（功能机用户）

### 5.3 支付失败 UX 处理

**核心问题**：
- 通用错误消息如"交易失败"毫无帮助——用户不知道钱是否被扣、是否应该重试、还是等待
- "支付处理中"这样的模糊状态最容易造成误判（用户可能取消或重复支付）

**最佳实践**：
- 精确描述发生了什么："银行服务器繁忙导致支付中止"优于"交易失败，重试"
- 明确告知钱是否被扣
- 显示自动退款状态（如适用）
- 提供预估解决时间
- 软拒绝时允许单击"重试"，保留所有已填信息
- "支付处理中"措辞应改为更具体的状态描述

**情绪设计**：
- 支付失败不仅是技术问题——它是情绪问题
- 处理不当会打破信任、中断购买意图
- 正确的设计让失败时刻变成恢复和安抚的机会

**来源**：
- [Orange Money Web Payment API | Orange Developer](https://developer.orange.com/apis/om-webpay)
- [Orange Money via Nuvei](https://www.nuvei.com/apm/orange-money)
- [How Wave is disrupting mobile money in Senegal | Rest of World](https://restofworld.org/2022/how-wave-is-disrupting-francophone-africas-mobile-money-market/)
- [How Wave rose to become Francophone Africa's first unicorn | Quartz](https://qz.com/africa/2189528/how-wave-rose-to-become-francophone-africas-first-unicorn)
- [Senegal's Payment Rails | Transfi](https://www.transfi.com/blog/senegals-payment-rails-how-they-work---gim-uemoa-wave-mobile-money-instant-transfers)
- [Oops, payment failed? Designing a calm path | Headout Studio](https://www.headout.studio/oops-payment-failed-how-we-designed-a-calm-path-to-completion/)
- [Making Failed Payments Less Stressful | Medium](https://medium.com/@ajal.connect/making-failed-payments-less-stressful-a-ux-perspective-37374f75b7bf)
- [UX Design: Troubleshoot Payments | HelloHuishi](https://www.hellohuishi.com/ux-design-troubleshoot-payments)

---

## 6. 非洲博彩平台 UX 参考

### 6.1 SportyBet——低带宽优化标杆

**核心策略**：
- APK 安装包 <30MB，安装快速
- 3 次点击完成赛前投注，体感最快
- 页面加载极快——低数据消耗设计
- 即使低带宽也能直播赛事
- 被评为尼日利亚最受欢迎的投注 App

**设计启示**：
- 极简主义不是设计选择，是生存必需
- 图片极少，文字为主的布局
- 赛事列表可缓存供离线浏览

### 6.2 Betway Africa——信任+可及性

**核心策略**：
- 与运营商合作提供"Data-Free"（免流量）访问
- 低内存、老旧设备友好
- 特定市场无需消耗数据即可访问
- 针对空间受限的设备优化移动版

**"Old Mobile"网站（轻量版）**：
- 尼日利亚多家博彩公司提供精简版网站
- 更少图形 = 更快加载 = 更少流量消耗
- 专为旧手机和慢网络设计

### 6.3 PlayerBet 案例研究（设计研究）

- 通过负责任的以用户为中心的设计重新定义数字体育博彩体验
- 值得作为 UX 设计方法论参考

**来源**：
- [Best Low Data Betting App in Nigeria | Pulse Sports](https://www.pulsesports.ng/story/best-low-data-betting-app-in-nigeria-top-lightweight-apps-for-slow-networks-2026011310481268532)
- [Bet Without Lag: Nigeria's Old Mobile Betting Sites | Pulse Sports](https://www.pulsesports.ng/betting/story/bet-without-lag-a-guide-to-nigerias-best-old-mobile-betting-sites-2025092621390534187)
- [Top Betting Apps in Nigeria | Pulse Sports](https://www.pulsesports.ng/betting/story/the-top-betting-apps-in-nigeria-for-the-ultimate-mobile-experience-2025091117145970885)
- [PlayerBet Case Study | Medium/Bootcamp](https://medium.com/design-bootcamp/playerbet-case-study-redefining-the-sport-betting-experience-through-responsible-user-centric-29ab7e3e50f9)
- [Best Betting Sites in Africa | foot-africa](https://foot-africa.com/en/betting-sites/)

---

## 7. 负责任博彩

### 7.1 非侵入性整合策略

**时间/金额提醒**：
- "现实检查"弹窗应告知玩家时长和总输赢，但不完全打断游戏沉浸感
- 微干预：轻量 UI 通知告知会话时长或消费进度，不中断核心游戏循环
- 冷却计时器在连续亏损后触发

**自我排除**：
- 排除系统在数据库层面限制访问，可临时（冷却期）或永久
- 激活时必须清除活跃会话、终止进行中的游戏、阻止营销通讯
- 失败安全设计：激活后立即生效

**消费限制**：
- 控制面板提供集中界面，用户无需复杂导航即可调整限制
- 支持日/周/月存款上限预设
- 达到上限时平台应自动阻止，而非仅提示

### 7.2 设计趋势："有意摩擦"

- 历史上赌博 UX 追求"无摩擦"
- 行为科学家现在主张"设计摩擦"：
  - 亏损后冷却计时器
  - 损失和时间限制的预承诺功能
  - 长时间会话期间的视觉暗示（如灰度过渡）

**来源**：
- [Implement responsible gambling tools for iGaming | Wizards](https://wizards.us/blog/responsible-gambling-tools/)
- [Responsible product design: scoping review | PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC8057587/)
- [Balancing Fun and Responsibility | SDLC Corp](https://sdlccorp.com/post/balancing-fun-and-responsibility-the-role-of-responsible-gaming-in-gambling-apps/)
- [Strong Design to Encourage Responsible Gambling | Soft2Bet](https://www.soft2bet.com/news/strong-design-to-encourage-responsible-gambling)
- [The Future Of Gambling: Regulation, AI & UX | SCCG Management](https://sccgmanagement.com/sccg-news/2025/4/15/the-future-of-gambling-how-regulation-ai-ux-are-redefining-online-casinos/)

---

## 8. GSMA 移动支付 UX 指南与低识字率设计

### 8.1 GSMA 关键发现

**UX 障碍**：
- 语言限制、对基础和功能机不适用、USSD 的局限性导致 CX/UX 差
- 产品设计不适合目标用户是移动支付采用的主要障碍之一

**解决方案**：
- 改进用户界面可部分解决低识字率用户的使用问题
- 设计良好的 App 比 USSD 或 SIM Toolkit 更容易完成交易
- GSMA 的"全民生物识别"方案（语音、面部、指纹识别）有助于缩小使用差距
- "技术+触达"的组合——数字方案降低运营费用，但与客户的定期接触仍然重要

**数字素养**：
- 许多用户是首次使用移动支付的用户
- 解决教育和数字素养障碍对推动采用至关重要
- GSMA 提供数字素养培训指南和移动互联网技能培训工具包（MISTT）

### 8.2 低识字率 UI 设计研究

**核心挑战**：
- 全球约 7.5 亿成人是文盲，主要在南亚、西亚和撒哈拉以南非洲
- 低识字率用户面对模糊图标、稀疏导航线索和复杂布局时遇到困难

**有效的设计策略**：
- 用户可以成功与非文字元素交互（图标和图像）
- 视觉线索（图标、空间分组、颜色编码）减少认知负荷，支持基于识别的交互
- 但仅有图标和空间布局不足以支持导航或多步骤任务——会导致犹豫、误解和放弃

**设计框架建议**：
- 视觉元素（颜色、形状、叠加、动画、反馈）不仅支持识别，还要支持解释、导航和任务完成
- 提供图形线索、语音注释和本地语言支持
- 避免复杂导航和输入
- 纯文本界面对低识字率首次用户不可用
- 替代方案：语音对话系统、图形界面、人工客服

**最佳实践**：
- 图标始终配合描述性文字标签
- 分层设计：图标+文字+颜色三重编码

**来源**：
- [GSMA Mobile Money adoption strategies](https://www.gsma.com/solutions-and-impact/connectivity-for-good/mobile-for-development/blog/mobile-money-adoption-strategies-for-start-ups-in-emerging-markets/)
- [GSMA Digital Financial Literacy](https://www.gsma.com/solutions-and-impact/connectivity-for-good/mobile-for-development/mobile-money/digital-financial-literacy/)
- [GSMA Digital Financial Literacy Toolkit (PDF)](https://www.gsma.com/solutions-and-impact/connectivity-for-good/mobile-for-development/wp-content/uploads/2023/03/The-digital-financial-literacy-toolkit-Addressing-the-gap-in-low-and-middle-income-countries.pdf)
- [GSMA barriers to Mobile Money adoption](https://mobilemoneyafrica.com/blog/gsma-lists-barriers-to-mobile-money-adoption-in-emerging-markets)
- [Designing UI for Illiterate/Semi-Literate Users | SAGE Journals](https://journals.sagepub.com/doi/full/10.1177/21582440231172741)
- [Mobile interface design for low-literacy populations | ResearchGate](https://www.researchgate.net/publication/254004140_Mobile_interface_design_for_low-literacy_populations)
- [Designing Mobile Interfaces for Novice/Low-Literacy Users | ACM](https://dl.acm.org/doi/10.1145/1959022.1959024)
- [Actionable UI Design Guidelines for Low-Literate Users | ACM](https://dl.acm.org/doi/10.1145/3449210)

---

## 总结：设计决策关键洞察

### 最重要的 5 个发现

1. **SMS 优先不是口号**：60% 的非洲用户使用功能机，USSD 会话只有 120 秒，160 字符限制——这些是硬约束，必须先设计这个层级
2. **Wave 改变了支付格局**：800 万月活、1% 费率、App-to-App+QR 码——但功能机用户仍需 Orange Money USSD 和话费扣款
3. **支付失败是情绪危机**：用户最怕的不是"支付失败"而是"不知道钱去哪了"——交易编号和最终状态 SMS 通知是底线
4. **低识字率设计不只是加图标**：图标能帮助识别，但不能帮助导航多步骤任务——需要图形+语音+颜色的三重编码
5. **负责任博彩走向"有意摩擦"**：不再追求"无摩擦购买"，而是在关键节点设计冷却计时器、消费进度提示、灰度视觉暗示
