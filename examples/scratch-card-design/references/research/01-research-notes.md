# 即开票/刮刮乐产品设计 - 调研笔记

> 调研日期：2026-04-12
> 调研目的：为西非彩票数字化运营Skill收集行业数据、标准和最佳实践

---

## 1. 奖级结构与RTP数据

### 1.1 RTP（返奖率）行业标准

**线下实体即开票：**
- 行业范围：50%-75%，主流区间 **60%-70%**
- 美国明尼苏达州法定最低：60%
- 明尼苏达州实际范围：63%-74%，取决于面值
- UK National Lottery Scratchcard：60%-70%
- 面值越高，RTP通常越高

**线上即开票（e-Scratch）：**
- 行业范围：80%-96%
- 典型值：90%左右，因省去印刷/物流/零售商佣金成本

**RTP by面值参考（美国市场）：**
| 面值 | RTP范围 | 说明 |
|------|---------|------|
| $1 | ~60-63% | 最低tier |
| $2 | ~62-65% | |
| $5 | ~65-68% | 中间tier |
| $10 | ~68-71% | |
| $20 | ~70-73% | |
| $30 | ~72-74% | 最高tier |

来源：
- https://www.mnlottery.com/blog/scratch-game-payouts-by-price-point-2
- https://www.gambling.com/us/online-casinos/strategy/top-scratch-cards-with-the-highest-rtp-3722400
- https://scratchcard-winners.co.uk/player-guides/what-is-scratchcard-rtp/
- https://www.kingcasino.com/blog/chances-of-winning-on-a-scratch-card/

### 1.2 整体中奖概率

- $1-$2面值票：约1:4.5到1:5.0
- UK National Lottery "Just To Say"（低面值）：1:3.82
- UK National Lottery "Good Luck"：1:3.86
- 高面值票（$10+）可达1:3到1:4
- 行业通用范围：**1:3到1:5**

来源：
- https://www.national-lottery.co.uk/c/files/scratchcards/justtosay.pdf
- https://www.national-lottery.co.uk/c/files/scratchcards/goodluck.pdf
- https://www.jackpot.com/blog/lottery-tips/how-to-pick-scratchoffs

### 1.3 奖级分布

- 典型奖级数量：**5-10级**
- 底层奖级（面值返回或小额）占总中奖票90-99%+
- 头奖概率极低：可达1:11,701,050
- 头奖与面值倍数关系（观察数据）：
  - $1票头奖：$5,000-$10,000（5,000x-10,000x）
  - $5票头奖：$50,000-$100,000（10,000x-20,000x）
  - $10票头奖：$100,000-$500,000（10,000x-50,000x）
  - $20票头奖：$500,000-$2,000,000（25,000x-100,000x）
  - $30票头奖：$1,000,000+

来源：
- https://scratchsmarter.com/scratch-academy/detailed-odds/
- https://www.palottery.pa.gov/games/instant-games/instant-games-prize-chances.aspx

### 1.4 UK National Lottery Game Procedures具体数据

**"Just To Say"游戏：**
- 总奖金价值 = 票面总值的59.98%
- 总奖金价值：GBP 8,337,386
- 整体中奖概率：1:3.82

**"£500 Selection"游戏：**
- 总奖金价值 = 票面总值的70%
- 总奖金价值：GBP 63,013,365

来源：
- https://www.national-lottery.co.uk/c/files/scratchcards/justtosay.pdf
- https://www.national-lottery.co.uk/c/files/scratchcards/500selection.pdf

---

## 2. 印刷安全技术

### 2.1 制造工艺

- 大卷纸stock（每卷最重2000磅）逐层涂覆
- **14个主要工序**从纸到成品
- 印刷技术：柔版印刷（Flexo），CMYK+多PMS色
- 1990年代后可在刮开层上方印四色图像
- 主要供应商：Scientific Games、IGT、Pollard Banknote

来源：
- https://www.madehow.com/Volume-4/Instant-Lottery-Ticket.html
- https://www.tresu.com/industries/lottery

### 2.2 安全层级

**乳胶覆盖层（Scratch-off Coating）：**
- 成分：碳黑颜料或铝膏混合丙烯酸树脂
- 溶剂：甲乙酮（MEK）
- 功能：完全遮挡下方数字，但可轻松刮除

**SecurTag安全涂层（Scientific Games专利）：**
- 位于刮开涂层下方的秘密涂层
- 肉眼不可见，但通过专用设备可识别
- 用于零售端真伪验证

**序列号系统：**
- 可使用十六进制系统
- 随机或顺序编码
- 不同奖级/地区可分配不同前缀
- 实现可追溯性和合法性验证

**条码验证：**
- 条码 + 3-4位安全码（隐藏在刮开层下）双重验证
- 零售终端扫码 + 手动输入安全码确认中奖

**防入侵保护：**
- 防透光、防物理撬开、防电气检测
- 防冷冻、防加热、防X射线
- 防化学溶剂、防磁性检测、防美术复制

**其他安全特征：**
- UV/IR安全油墨：可见光下透明，紫外灯下荧光
- 微文字（Microtext）：极小文字，肉眼难辨
- 全息标识（Hologram）：3D光学图像，无法用任何已知印刷技术复制
- Void if Removed：移除后留下"VOID"痕迹
- 黑线叠印（Black Line Overprint）

**Scientific Games HD Games技术：**
- 高DPI复杂印刷工艺
- Dimension产品：全息效果（Cracked Ice, Stella图案）
- 冷箔（Cold Foil）技术

来源：
- https://www.madehow.com/Volume-4/Instant-Lottery-Ticket.html
- https://www.arojet.net/a-news-the-role-of-security-features-in-lottery-ticket-printing-machines.html
- https://world-lotteries.org/insights/editorial/blog/50-years-of-science-built-into-every-game-behind-the-scenes-of-the-worlds-largest-instant-game-operations
- https://www.scientificgames.com/news/media-releases/scientific-games-expands-north-american-lottery-instant-scratch-game-innovation-with-new-printing-technology/

---

## 3. 线上即开票（e-Scratch）架构

### 3.1 服务器端预判定 vs RNG实时判定

**实体票：预判定（Pre-determined）**
- 每张票在印刷时已确定输赢
- 随机性发生在印刷和包装环节

**线上票：两种模式**

模式A - 实时RNG判定：
- 玩家点击时RNG实时生成结果
- 刮开动画纯粹是视觉效果
- 预先确定的是总体奖级结构（如100个GBP10奖、5个GBP500奖）

模式B - 奖池预判定：
- 预生成有限数量的"虚拟票"放入奖池
- 玩家购票时从池中随机抽取一张
- 分两种子类型：
  - **Depleting Pool（消耗型）**：奖池中的票用完即止，类似实体票
  - **Non-depleting/Replenishing Pool（补充型）**：奖池自动补充，持续运营

**Apollo RGS（Remote Game Server）：**
- 处理游戏托管、管理和资产交付
- 提供实时游戏结果、奖级和游戏流程
- 动态向玩家设备推送动画和结果
- 支持Depleting和Replenishing两种奖池机制

来源：
- https://www.wizardslots.com/blog/are-instant-win-games-fixed
- https://www.fortunegames.com/blog/instant-win-games-are-they-fixed-or-predetermined
- https://allwyn-lotterysolutions.com/solutions/digital-game-delivery/
- https://www.intralot.com/gaming-solution/interactive-games/e-instant-win-games/

### 3.2 GLI标准

**GLI-19：交互式游戏系统标准**
- 适用范围：在线游戏平台、远程体育博彩、虚拟和真人赌场游戏、管理/监控/交易系统
- 核心内容：
  - RNG要求（不可预测性、无偏差）
  - 机械RNG规范
  - 游戏公平性
  - 游戏支付百分比
  - 赔率和非现金奖励
- 许多司法管辖区接受GLI-19认证或以其为基础
- 官方文档：https://gaminglabs.com/wp-content/uploads/2020/07/GLI-19-Interactive-Gaming-Systems-v3.0.pdf

**GLI认证服务：**
- 独立测试实验室进行最佳实践测试
- 包括预印刷、干样本、湿样本测试
- 基于标准操作程序和客户特定要求

来源：
- https://gaminglabs.com/services/igaming/random-number-generator-rng/
- https://gaminglabs.com/gli-standards/
- https://gaminglabs.com/services/professional-services/lottery-gaming-practice/

---

## 4. UK National Lottery透明度机制

### 4.1 公开信息

- **每张票背面印有**：整体中奖概率、游戏编号（4位数字格式如"1234-0000000-000"中前4位）
- **官网公布**：
  - 每款游戏的头奖数量及剩余数量
  - 头奖剩余数在1个工作日内更新
  - 注意：头奖以下的奖级剩余数不公布
  - 每款游戏初始发行的各级奖项数量
- **Game Procedures文件公开**：
  - 每款游戏有独立的PDF文件
  - 包含完整奖级表、总票数、各级奖项数量、RTP百分比
  - 公开地址格式：https://www.national-lottery.co.uk/c/files/scratchcards/{gamename}.pdf

### 4.2 退市机制

- 所有头奖被认领后启动退市程序
- 零售商可继续销售已激活的库存
- 未激活库存被收回
- 退市后有兑奖期限（通常180天至1年）

### 4.3 UK Gambling Commission监管

- 由Gambling Commission监管National Lottery运营
- 通过信息自由法可获取相关文件

来源：
- https://www.national-lottery.com/national-lottery-scratchcard-prizes
- https://www.national-lottery.co.uk/games/gamestore/scratchcards/rules
- https://www.gamblingcommission.gov.uk/about-us/freedomofinformation/national-lottery-scratchcards
- https://gamblingnews.uk/betting/2026-03-24/top-prizes-left-on-lottery-scratch-cards-uk-millions-remain-unclaimed-as-data-reveals-the-odds/

---

## 5. 西非市场与LONASE

### 5.1 UEMOA经济背景

- UEMOA八国：贝宁、布基纳法索、科特迪瓦、几内亚比绍、马里、尼日尔、塞内加尔、多哥
- 统一货币：FCFA（西非法郎）
- 固定汇率：655.957 FCFA = 1 EUR（与欧元挂钩）

**塞内加尔人均收入：**
- 2024 GDP per capita：约$1,524-$1,760（名义）
- 2025 GDP per capita：约$1,921（名义）
- 日均收入（名义）：约$4.82-$5.27/天
- 换算FCFA：约2,900-3,200 FCFA/天（按1 USD ≈ 600 FCFA）
- PPP调整后：约$12.32/天

### 5.2 非洲彩票市场

- 2024年非洲彩票市场规模：$56亿
- 预计2032年达$113.2亿
- CAGR：9.2%（2026-2032）

### 5.3 LONASE信息

- 全称：Loterie Nationale Senegalaise
- 成立：1966年
- 1987年成为国企，同时作为监管机构负责许可和监管
- 产品：彩票（Sen Loto Jackpot）、体育竞猜（PMU赛马）、即开票、在线博彩

### 5.4 Dare-Dare品牌

**关键信息（来自LudWin官网）：**
- LudWin、FDJ Gaming Solutions(FGS)和LONASE达成协议开发即开票产品线
- **Dare-Dare是LONASE所有即开票的品牌名**
- LONASE总裁Lat Diop表示将全力重振即开票板块

**已发布游戏：**
1. **Numero Porte Bonheur（幸运号码）**
   - 面值：100 FCFA
   - 头奖：100,000 FCFA（1000倍）
   - 设计理念：基于"幸运数字信仰"和"颜色偏好"两个文化洞察
2. **But（进球）**
   - 足球主题即开票

来源：
- https://ludwingroup.com/blog/2021/11/22/la-loterie-nationale-senegalaise-lance-sa-gamme-de-loterie-instantanee-dare-dare-avec-ludwin-et-fdj-gaming-solutions/
- https://ludwingroup.com/fr/2021/01/19/massa-vitae-toutor-condimentum-lacinia-quis/
- https://senego.com/lonase-lancement-du-nouveau-jeu-de-grattage-numero-porte-bonheur-photos_1293991.html
- https://www.rsesenegal.com/2021/01/13/gambling-in-senegal/

### 5.5 面值策略推导

基于日均收入2,900-3,200 FCFA：
| 面值 | 占日均收入比 | 定位 |
|------|-------------|------|
| 100 FCFA | ~3% | 大众入门，冲动购买，零钱消费 |
| 200 FCFA | ~6% | 轻度参与 |
| 500 FCFA | ~16% | 中端，有意购买 |
| 1,000 FCFA | ~32% | 高端，目标性购买 |

---

## 6. 产品生命周期管理

### 6.1 上市前

**审批流程：**
- 独立测试实验室进行合规性评估
- 检查项包括：
  - 当前法规合规
  - 版权检查
  - 社会影响分析（敏感视觉/负责任博彩）
  - 游戏机制和奖级结构验证
  - 基于可见信息的不可预测性测试
  - 印刷商内部质量安全测试
  - 彩票运营商质量安全测试
  - 独立实验室测试

**Monte Carlo模拟：**
- 用于模拟彩票购买的随机性系统
- 考虑客户分布和需求分布（正态分布）
- 验证奖级结构在大量销售下的表现
- 参考：https://jmsallan.netlify.app/blog/2024-10-04-analyzing-a-lottery-with-monte-carlo-simulation/

### 6.2 运营中

**监控指标：**
- 各级奖项剩余数量
- 销售进度 vs 预期
- 头奖认领情况
- 零售端库存分布

### 6.3 退市

**触发条件：**
- 所有头奖已被认领
- 记录在案的商业原因（此时可能仍有未认领奖项包括头奖）
- 销售达到预定目标
- 产品老化/市场疲劳

**退市流程：**
- 停止向零售商分发新库存
- 已激活库存可继续销售至售罄
- 未激活库存收回
- 设定兑奖截止期（美国通常180天至1年）

来源：
- https://gaminglabs.com/services/professional-services/lottery-gaming-practice/
- https://www.texaslottery.com/export/sites/lottery/Games/Scratch_Offs/closing.html
- https://www.palottery.pa.gov/scratch-offs/prizes-remaining.aspx

---

## 7. 技术供应商生态

### 7.1 主要印刷/技术供应商

| 供应商 | 角色 | 备注 |
|--------|------|------|
| Scientific Games | 印刷、技术、游戏设计 | 全球最大，1989年起服务FDJ |
| IGT (International Game Technology) | 印刷、系统 | 与Scientific Games并列双巨头 |
| Pollard Banknote | 即开票印刷 | 北美主要供应商 |
| FDJ Gaming Solutions | B2B方案、策略咨询 | FDJ集团的B2B子公司，服务LONASE |
| LudWin | 全方案供应商 | 服务LONASE和LONAGUI |
| Intralot | e-Instant解决方案 | 希腊公司，全球业务 |
| Allwyn | 数字游戏交付 | Apollo RGS |
| Skilrock | 彩票软件和iGaming平台 | RNG认证方案 |

### 7.2 LONASE的供应商关系

- LudWin + FDJ Gaming Solutions：联合开发Dare-Dare品牌
- FDJ Gaming Solutions提供：Interactive Factory、零售转型、即开票现代化、体育博彩
- LudWin提供：从游戏设计到部署的全链路服务，包括物流、品牌、营销

来源：
- https://www.fdj-gaming-solutions.com/services/
- https://ludwingroup.com/our-solutions/
- https://www.scientificgames.com/
- https://www.intralot.com/gaming-solution/interactive-games/e-instant-win-games/

---

## 8. WLA安全标准

- WLA-SCS:2020（世界彩票协会安全控制标准）
- 涵盖：审计、游戏设计和编程、安全条码、刮开涂层、安全分销
- 多层重叠安全系统：防火墙、网络和其他安全设备
- 游戏数据使用高级加密系统完全加密
- 参考：https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2020/CoP-2020_V1.2.pdf
