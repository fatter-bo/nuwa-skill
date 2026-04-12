# Web3 项目风险红旗清单 (Red Flags Checklist)

> 调研日期：2026-04-12
> 调研目标：建立全面的 Web3 项目风险红旗检测框架，用于投资尽调中的快速筛查与深度评估。

---

## 一、快速否决清单 (Instant Kill List)

**出现以下任何一条，直接放弃该项目，无需继续评估：**

| # | 红旗条目 | 说明 |
|---|---------|------|
| 1 | 合约未开源且未审计 | 无法验证代码逻辑，可能隐藏后门（mint、blacklist、转账限制等） |
| 2 | 流动性未锁定或锁定期 < 30 天 | 开发团队可随时撤走流动性，经典 rug pull 手法 |
| 3 | 合约含无限铸币（infinite mint）函数且无治理约束 | owner 可无限增发代币并抛售，Halborn 报告显示此类漏洞损失超数十亿美元 |
| 4 | 蜜罐合约（Honeypot）—— 用户只能买不能卖 | 隐藏的 sell 限制 / blacklist 机制，CertiK 确认这是大规模犯罪模式 |
| 5 | 创始团队完全匿名且无任何可验证产品 / 历史 | ZachXBT 调查显示匿名+无产品是 rug pull 最强预测指标 |
| 6 | 单一地址持有 > 50% 代币供应量 | 极端集中持仓，持有者可单方面崩盘价格 |
| 7 | 项目已上 Rekt.news 排行榜或被 ZachXBT 公开点名 | 已有确认的安全事件或欺诈指控 |
| 8 | 合约 owner 可直接修改余额 / 暂停交易 / 提走用户资金 | 权限过大，无 timelock 或多签保护，等同于中心化托管 |

---

## 二、合约安全红旗

### 2.1 审计相关

| 红旗 | 严重等级 | 详细说明 |
|------|---------|---------|
| 完全未经审计 | :red_circle: 立即放弃 | 未审计的合约无法排除后门、reentrancy、oracle 操纵等风险。CertiK 和 Trail of Bits 均强调审计是最低安全门槛 |
| 仅有单一审计方且非知名机构 | :yellow_circle: 需要解释 | 单一审计存在盲区。最佳实践是 2 家以上独立审计（如 CertiK + Trail of Bits + OpenZeppelin） |
| 审计后发生重大代码改动 | :red_circle: 立即放弃 | 审计报告覆盖范围已失效，改动可能引入新漏洞或后门 |
| 审计报告中的 Critical/High 问题未修复 | :red_circle: 立即放弃 | 说明团队不重视安全，或故意保留漏洞 |
| 审计报告未公开或仅提供摘要 | :yellow_circle: 需要解释 | 透明度不足，可能隐藏关键发现 |
| 审计报告日期过旧（> 12 个月） | :yellow_circle: 需要解释 | 合约可能已迭代多次，老审计不再适用 |

### 2.2 合约设计

| 红旗 | 严重等级 | 详细说明 |
|------|---------|---------|
| 代理合约（Proxy）可升级且 admin 为单一 EOA | :red_circle: 立即放弃 | Wormhole 曾因未初始化 UUPS 代理出现严重漏洞。admin 应为多签 + timelock |
| 未使用多签钱包管理关键权限 | :red_circle: 立即放弃 | Halborn 报告：仅 19% 被黑项目使用多签，仅 2.4% 使用冷存储 |
| owner 可暂停所有交易 / 修改代币参数 | :yellow_circle: 需要解释 | 紧急暂停可接受，但需 timelock + 治理流程，否则构成单点风险 |
| 无 timelock（时间锁）机制 | :yellow_circle: 需要解释 | 最佳实践要求 24-72 小时 timelock，给社区反应时间 |
| 合约使用 selfdestruct 或 delegatecall 到不可信地址 | :red_circle: 立即放弃 | 可导致合约被销毁或被任意控制 |
| 未遵循 CEI（Checks-Effects-Interactions）模式 | :yellow_circle: 需要解释 | reentrancy 攻击的主要入口，2024 年仍造成大量损失 |

### 2.3 已知攻击向量（基于 Rekt.news / Halborn 数据）

| 攻击类型 | 占比/规模 | 说明 |
|---------|----------|------|
| 访问控制漏洞 | 2025 年损失 $9.53 亿 | 排名第一的攻击向量，私钥泄露 + 权限滥用 |
| 闪电贷攻击（Flash Loan） | 2024 年占合约攻击的 83.3% | 利用无抵押借贷放大 oracle 操纵、reentrancy 等漏洞 |
| Oracle 操纵 | 闪电贷攻击中最常见的子类型 | 依赖单一价格源或链上 TWAP 被操纵 |
| 输入验证不足 | 占直接合约攻击的 34.6% | 2021/2022/2024 年均为首要合约漏洞原因 |
| 链下攻击（Off-chain） | 2024 年占被盗资金的 80.5% | 包括私钥被盗、社会工程、钓鱼攻击 |
| Reentrancy（重入攻击） | 经典持续性威胁 | 外部调用前未更新状态，允许递归提款 |

---

## 三、团队红旗

| 红旗 | 严重等级 | 详细说明 |
|------|---------|---------|
| 团队完全匿名且无可验证产品/代码 | :red_circle: 立即放弃 | rug pull 最强预测指标。ZachXBT 揭露的多数骗局均涉及匿名团队 |
| 创始人有多次失败/跑路项目历史 | :red_circle: 立即放弃 | ZachXBT 2024 年揭露 ZKasino 团队成员后续创建 White Rock 继续行骗 |
| 创始人/团队在 TGE 后持续大量抛售代币 | :red_circle: 立即放弃 | 利益不一致的最直接信号 |
| 团队 LinkedIn 资料无法验证或为虚构 | :yellow_circle: 需要解释 | 部分项目使用 AI 生成的虚假团队头像和履历 |
| 核心开发只有 1-2 人 | :yellow_circle: 需要解释 | 单点故障风险高，项目可持续性存疑 |
| 团队拒绝 KYC / 法律实体不明 | :yellow_circle: 需要解释 | 出事后无法追责，法律保护缺失 |
| 社交媒体过度营销，承诺保底收益 | :red_circle: 立即放弃 | "guaranteed returns" 是经典庞氏骗局话术，任何合法项目不会承诺固定回报 |

---

## 四、代币经济学红旗

| 红旗 | 严重等级 | 详细说明 |
|------|---------|---------|
| TGE 解锁比例 > 25% | :yellow_circle: 需要解释 | 研究显示 TGE 解锁 > 25% 的项目首年价格中位跌幅 72%，vs 低于 15% 的仅跌 38% |
| 团队分配 > 30% 且无锁定 | :red_circle: 立即放弃 | 极度不合理的分配，团队可随时砸盘 |
| VC/投资者分配 > 25% 且弱锁定 | :yellow_circle: 需要解释 | 解锁时面临巨大抛压，需验证锁定合约是否链上可查 |
| 无 cliff 期或 cliff < 3 个月 | :red_circle: 立即放弃 | 短 cliff = 团队可早期抛售，无长期承诺 |
| 总锁定期 < 24 个月 | :yellow_circle: 需要解释 | 核心团队标准锁定应 3-4 年，低于 2 年是警告信号 |
| 单月解锁量 > 流通量的 5% | :red_circle: 立即放弃 | 历史数据显示此类事件导致 15-40% 的价格下跌 |
| 代币无明确用途 / 无收入支撑却全流通 | :yellow_circle: 需要解释 | 缺乏价值捕获机制，纯投机标的 |
| 无限供应且无通缩机制 | :yellow_circle: 需要解释 | 长期通胀侵蚀持有者价值 |
| 锁仓合约未上链 / 仅为团队口头承诺 | :red_circle: 立即放弃 | 无链上约束的锁仓承诺毫无约束力 |

---

## 五、运营与社区红旗

| 红旗 | 严重等级 | 详细说明 |
|------|---------|---------|
| Discord/Telegram 大量机器人账号（新号、无头像、定时发言） | :red_circle: 立即放弃 | 虚假社区是骗局标配。检查方法：账号创建日期、发言模式、头像真实性 |
| 社区成员数暴涨但互动质量极低 | :yellow_circle: 需要解释 | 可能购买了僵尸粉，真实活跃度需深入分析 |
| 多个中等 KOL 同时推广同一未知代币 | :red_circle: 立即放弃 | 协调付费推广的典型模式，非有机增长 |
| 项目方禁止社区提问 / 删除负面评论 | :red_circle: 立即放弃 | 健康项目欢迎质疑，删帖禁言是心虚表现 |
| 仅使用 FOMO 营销（"最后机会"、"限时"） | :yellow_circle: 需要解释 | 施压策略是骗局常用手段，制造紧迫感阻止理性判断 |
| 官方频道管理员冒充或被冒充 | :yellow_circle: 需要解释 | Coinbase 安全报告显示这是常见钓鱼入口 |
| 无技术讨论 / 社区全是价格讨论 | :yellow_circle: 需要解释 | 缺乏技术社区参与说明项目缺乏实质内容 |

---

## 六、技术与开发红旗

| 红旗 | 严重等级 | 详细说明 |
|------|---------|---------|
| GitHub 仓库不存在或为空 | :red_circle: 立即放弃 | 声称开源却无代码，基本可确认为骗局 |
| GitHub 超过 6 个月无实质性更新 | :red_circle: 立即放弃 | 项目可能已被放弃 |
| 代码完全 fork 自其他项目且无原创改进 | :yellow_circle: 需要解释 | Santiment 研究表明 fork 会继承所有 commit 历史，制造虚假繁荣。如 ChainCoin fork Bitcoin 后仅 30 次提交 |
| 仅有 README 更新或文档修改，无代码提交 | :red_circle: 立即放弃 | 人为制造"活跃"假象，实际无开发工作 |
| 所有提交来自单一开发者 | :yellow_circle: 需要解释 | 单点故障风险高，且可能意味着团队规模被夸大 |
| 测试网运行超过 12 个月仍未上主网 | :yellow_circle: 需要解释 | 可能遇到无法解决的技术障碍，或项目在拖延 |
| 无测试代码 / 测试覆盖率极低 | :yellow_circle: 需要解释 | 代码质量堪忧，增加漏洞风险 |
| 合约代码闭源 | :red_circle: 立即放弃 | 无法独立验证安全性，违背 Web3 透明原则 |

---

## 七、历史暴雷模式（基于 Rekt.news 排行榜分析）

### 7.1 Rekt.news 排行榜 Top 暴雷事件（截至 2025 年底）

| 排名 | 事件 | 损失 | 攻击类型 |
|-----|------|------|---------|
| 1 | Bybit (2025) | ~$14 亿 | 私钥/社会工程（朝鲜黑客） |
| 2 | Ronin Network | $6.24 亿 | 私钥泄露/验证者被控制 |
| 3 | Poly Network | $6.11 亿 | 访问控制漏洞 |
| 4 | Wormhole | $3.26 亿 | 未初始化代理合约 |
| 5 | Euler Finance | $1.97 亿 | 闪电贷 + 捐赠逻辑漏洞 |

### 7.2 最常见暴雷模式总结

1. **私钥泄露/社会工程攻击** —— 2024 年占被盗资金的 80.5%，Bybit $14 亿事件为最大案例
2. **访问控制漏洞** —— 2025 年损失 $9.53 亿，owner 权限被滥用或被盗
3. **闪电贷 + Oracle 操纵组合攻击** —— 2024 年占合约层面攻击的 83.3%
4. **Rug Pull / Exit Scam** —— 2024 年 3 月 Solana meme coin 热潮中，ZachXBT 追踪 33 个预售项目，12 个确认 rug pull，损失 $2670 万
5. **代理合约升级漏洞** —— Wormhole 事件为典型案例，未初始化的 UUPS 代理被攻击者接管
6. **输入验证不足** —— 占直接合约攻击的 34.6%，贯穿 2021-2024 年

### 7.3 2024-2025 年度统计

- **2024 年全年**：加密行业被盗约 $22 亿
- **2025 年全年**：被盗超 $27 亿（De.Fi 数据）
- **2025 上半年**：加密欺诈损失达 $24.7 亿
- **链下攻击占比上升**：2024 年链下事件占 56.5% 的攻击次数，但造成 80.5% 的资金损失

---

## 八、尽调工具推荐

| 工具 | 用途 | 链接 |
|------|------|------|
| Token Sniffer | 自动化代币合约扫描，检测蜜罐/后门 | tokensniffer.com |
| Honeypot.is | 检测蜜罐代币 | honeypot.is |
| DEXScreener | 实时流动性数据、交易对信息 | dexscreener.com |
| Bubblemaps | 可视化关联钱包，检测集中持仓 | bubblemaps.io |
| Etherscan/BSCScan | 合约验证、交易追踪 | etherscan.io |
| Santiment | GitHub 开发活跃度追踪 | santiment.net |
| Tokenomist (TokenUnlocks) | 代币解锁时间表追踪 | tokenomist.ai |
| Rekt.news | 安全事件数据库 | rekt.news |
| De.Fi REKT Database | DeFi 安全事件数据库 | de.fi/rekt-database |
| RugDoc | DeFi 协议风险评估 | rugdoc.io |

---

## 九、审计机构分级参考

### Tier 1（高可信度）
- **Trail of Bits** —— 使用 Slither、Echidna、Medusa 等自研工具，覆盖合约+系统架构+oracle+升级机制
- **OpenZeppelin** —— 行业标准库作者，审计方法论成熟
- **Consensys Diligence** —— MythX 工具链，以太坊生态深度参与者

### Tier 2（良好）
- **CertiK** —— AI + 形式化验证 + 人工审查三层方法论，市场份额最大
- **Halborn** —— 发布 Top 100 DeFi Hacks 年度报告，渗透测试能力强
- **Quantstamp** —— 自动化审计工具链完善

### 审计评估要点
- 是否有 2 家以上独立审计方？
- 审计范围是否覆盖当前部署版本？
- Critical/High 发现是否全部修复并经复审确认？
- 审计报告是否完整公开（非仅摘要）？

---

## 十、综合评分框架

### 评估维度权重

| 维度 | 权重 | 说明 |
|------|------|------|
| 合约安全 | 30% | 审计状态、权限设计、已知漏洞 |
| 团队信誉 | 20% | 身份可验证性、历史记录、利益绑定 |
| 代币经济学 | 20% | 分配合理性、锁定机制、解锁节奏 |
| 运营与社区 | 15% | 社区真实性、营销方式、透明度 |
| 技术与开发 | 15% | 代码质量、开发活跃度、原创性 |

### 风险等级判定

| 等级 | 条件 | 建议 |
|------|------|------|
| :red_circle: **极高风险** | 命中任意"快速否决清单"条目 | 立即放弃，不再投入时间 |
| :red_circle: **高风险** | 累计 3 个以上 :red_circle: 红旗 | 强烈建议放弃 |
| :yellow_circle: **中等风险** | 累计 3-5 个 :yellow_circle: 红旗 | 需逐条获得合理解释后方可继续 |
| :green_circle: **可接受风险** | 无 :red_circle: 红旗，:yellow_circle: < 3 个 | 可进入深度尽调阶段 |

---

## 参考来源

- [Rekt.news Leaderboard](https://rekt.news/leaderboard) - 加密行业最大安全事件排行榜
- [Halborn Top 100 DeFi Hacks Report 2025](https://www.halborn.com/reports/top-100-defi-hacks-2025) - 2025 年度 DeFi 攻击统计分析
- [CertiK Auditing Methodology](https://www.certik.com/blog/how-we-audit-a-comprehensive-guide-to-certiks-auditing-methodology) - CertiK 审计方法论详解
- [Trail of Bits Software Assurance](https://trailofbits.com/services/software-assurance/) - Trail of Bits 安全审计服务
- [Three Sigma - Upgradeable Contract Security Risks](https://threesigma.xyz/blog/web3-security/upgradeable-contract-security-risks-vulnerabilities) - 可升级合约安全风险分析
- [Three Sigma - 2024 Most Exploited DeFi Vulnerabilities](https://threesigma.xyz/blog/exploit/2024-defi-exploits-top-vulnerabilities) - 2024 年 DeFi 漏洞统计
- [Chainforce - Red Flags in Tokenomics](https://chainforce.tech/learn-tokenomics/red-flags-in-tokenomics/) - 代币经济学红旗识别
- [Hackers stole over $2.7B in crypto in 2025 - TechCrunch](https://techcrunch.com/2025/12/23/hackers-stole-over-2-7-billion-in-crypto-in-2025-data-shows/) - 2025 年加密行业盗窃统计
- [ZachXBT - Wikipedia](https://en.wikipedia.org/wiki/ZachXBT) - ZachXBT 链上调查员资料
- [ZachXBT links White Rock to ZKasino exit scam](https://www.cryptopolitan.com/zachxbt-links-white-rock-to-zkasino-scam/) - ZKasino rug pull 调查案例
- [Blocklr - How to Spot a Crypto Rug Pull](https://blocklr.com/guides/how-to-spot-rug-pull/) - Rug Pull 识别指南
- [Cointelegraph - Honeypot Crypto Scam](https://cointelegraph.com/news/what-is-a-honeypot-crypto-scam-and-how-to-spot-it) - 蜜罐骗局详解
- [Santiment - Development Activity Metrics](https://academy.santiment.net/metrics/development-activity/) - 开发活跃度指标分析
- [OWASP Smart Contract Top 10 - Flash Loan Attacks](https://owasp.org/www-project-smart-contract-top-10/2025/en/src/SC07-flash-loan-attacks.html) - OWASP 智能合约安全 Top 10
- [De.Fi REKT Database](https://de.fi/rekt-database) - DeFi 安全事件数据库
- [Coinbase - How Scammers Target Crypto Communities](https://www.coinbase.com/blog/consumer-protection-tuesday-how-scammers-are-targeting-crypto-communities) - 加密社区诈骗方式分析
- [Nomoslabs - Smart Contract Upgrade Security](https://nomoslabs.io/blog/smart-contract-upgrade-security-proxy-pattern-risks) - 代理合约升级安全风险
- [EEA DeFi Risk Assessment Guidelines](https://entethalliance.org/specs/defi-risks/) - 企业以太坊联盟 DeFi 风险评估指南
