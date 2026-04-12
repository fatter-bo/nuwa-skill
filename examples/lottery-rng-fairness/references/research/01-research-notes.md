# 彩票RNG与可证公平性 · 调研笔记

> 调研时间：2026-04-12
> 调研目的：为西非彩票数字化运营（e-Scratch、KENO、Pick 3、Cash Chrono、UEMOA联合乐透）提供RNG选型、认证、架构、区块链方案、降级策略的技术决策依据。

---

## 1. RNG技术类型

### 1.1 HRNG/TRNG（硬件真随机数生成器）

- **原理**：从物理现象提取熵——热噪声、量子效应（光子极化）、大气噪声、放射性衰变等
- **特点**：真随机，不可预测，不可复现
- **限制**：产出速率低（kbps到Mbps级），成本高
- **代表产品**：
  - ID Quantique Quantis（量子随机，瑞士）：用于高端彩票和政府系统
  - Intel RDRAND/RDSEED：CPU内置，基于热噪声，便捷但吞吐有限
  - RANDOM.ORG：基于大气噪声，提供API服务
- **在彩票中的角色**：作为种子生成器，不直接用于高频出票

Sources:
- [ID Quantique Gaming & Lotteries](https://www.idquantique.com/random-number-generation/applications/gaming-and-lotteries/)
- [RANDOM.ORG Introduction to Randomness](https://www.random.org/randomness/)
- [RNG Meaning in iGaming - Skilrock](https://www.skilrock.com/blogs/understanding-rng-and-its-importance-in-the-lottery-ecosystem)

### 1.2 PRNG（伪随机数生成器）

- **原理**：数学算法从初始种子（seed）生成确定性序列，看起来随机但可复现
- **特点**：速度极快（Gbps级），给定相同种子输出相同序列
- **风险**：如果种子泄露或算法可逆，输出完全可预测
- **代表算法**：
  - Mersenne Twister（MT19937）：广泛使用但**不安全**，624个输出即可反推内部状态
  - Linear Congruential Generator：简单快速，但周期短、可预测
- **在彩票中**：**单独使用不满足GLI认证要求**，必须配合安全种子源

### 1.3 CSPRNG（密码学安全伪随机数生成器）

- **原理**：PRNG + 密码学安全保证——无法从任意多输出反推内部状态
- **特点**：速度快（Mbps-Gbps级），计算上不可预测
- **代表算法**：
  - AES-256-CTR模式：NIST推荐，硬件加速广泛
  - ChaCha20：Google采用，软件性能优异
  - Fortuna：自适应重播种设计，抗种子泄露
- **在彩票中**：配合HRNG种子，是行业主流方案

### 1.4 混合方案（行业主流）

**HRNG种子 -> CSPRNG扩展**：
- HRNG以低速率持续产生真随机比特
- CSPRNG以HRNG输出为种子（初始化+定期重播种）
- CSPRNG高速输出用于实际游戏
- 重播种策略：每N次生成或每T时间间隔，取较先到达者

这种混合方案兼顾：
- 真随机性（来自HRNG种子）
- 高吞吐量（来自CSPRNG算法）
- 密码学安全（来自CSPRNG的不可预测性）
- 认证友好（GLI认可混合方案）

Sources:
- [WLA: Random chance is the essence of the lottery](https://world-lotteries.org/insights/editorial/blog/random-chance-is-the-essence-of-the-lottery)
- [CDC Gaming: RNGs – What Are They, and Are They Random?](https://cdcgaming.com/rngs-what-are-they-and-are-they-random/)
- [Wikipedia: Random number generation](https://en.wikipedia.org/wiki/Random_number_generation)

---

## 2. 认证标准

### 2.1 GLI-19（Interactive Gaming Systems）

- **版本**：v3.0（2024年6月发布）
- **覆盖**：所有在线互动游戏系统（不含体育博彩），包括iLottery和e-instant产品
- **核心要求**：
  - RNG算法源码审查（完整可编译的最终版本）
  - RNG统计测试（NIST SP 800-22 + 额外测试）
  - 玩家账户管理（PAM）安全性
  - 财务交易完整性和准确性
  - 数据保护和安全通信
  - 运营完整性（日志、审计、异常处理）
- **认证流程**：提交源码和文档 -> GLI内部审查 -> 统计测试 -> 漏洞评估 -> 报告 -> 证书
- **机构**：Gaming Laboratories International（总部新泽西），400+司法管辖区认可

Sources:
- [GLI-19 v3.0 Standard PDF](https://gaminglabs.com/wp-content/uploads/2024/06/GLI-19-Interactive-Gaming-Systems-v3.0.pdf)
- [GLI RNG Certification Service](https://gaminglabs.com/services/igaming/random-number-generator-rng/)
- [GLI Lottery Professional Services](https://gaminglabs.com/services/professional-services/lottery-gaming-practice/)

### 2.2 GLI-26

- **注意**：GLI-26 v1.1实际为"Wireless Gaming Systems Standards"（无线游戏系统标准），主要面向赌场无线博彩设备
- **即开票相关**：GLI的即开票测试服务独立于编号标准，包括预印刷样本测试、干样测试、湿样测试等
- **实际适用**：西非e-Scratch应主要依据GLI-19（在线互动系统）+ GLI即开票专项测试流程

### 2.3 WLA-SCS（Security Control Standard）

- **最新版本**：WLA-SCS:2024
- **性质**：全球彩票行业唯一国际认可的安全控制标准
- **认证级别**：
  - Level 1：符合WLA-SCS所有适用控制项
  - Level 2：ISO/IEC 27001认证 + WLA-SCS所有适用控制项（供应商只能申请Level 2）
- **资格要求**：仅WLA正式会员（彩票运营商）和准会员（供应商）可申请
- **有效期**：3年
- **审计方**：经WLA批准的第三方审计师，最终证书由WLA颁发
- **与ISO 27001关系**：Level 2要求先通过ISO 27001，WLA-SCS在此基础上增加彩票行业特定控制项
- **认证机构**：SGS等经批准的审计机构可执行审计

Sources:
- [WLA-SCS:2024 Standard](https://publications.world-lotteries.org/security-and-risk-management/wla-security-control-standard)
- [WLA Guide to Certification](https://publications.world-lotteries.org/security-and-risk-management/guide-certification-wla-security-control-standard)
- [WLA SCS FAQ](https://www.world-lotteries.org/services/industry-standards/security-and-risk-management/scs-faq)
- [SGS WLA-SCS Certification](https://www.sgs.com/en/services/wla-scs-2020-lottery-certification)

### 2.4 认证机构对比

| 机构 | 成立 | 总部 | 优势 | 彩票专项 | 西非适用性 |
|------|------|------|------|---------|-----------|
| **GLI** | 1989 | 新泽西 | 全球最大，400+管辖区，数学家团队强 | iLottery、e-instant、RNG专项 | 首选——LONACI等认可 |
| **BMM Testlabs** | 1981 | 拉斯维加斯 | 最古老，40年信誉，美国市场最强 | 有彩票能力但以赌场为主 | 备选 |
| **iTech Labs** | 2004 | 澳大利亚 | 亚太强，流程相对灵活（2023年被GLI收购） | RNG测试能力强 | 可用于预评估 |
| **eCOGRA** | 2003 | 英国 | 玩家保护导向，ISO 17025认证 | 偏iGaming，彩票经验有限 | 补充认证 |

### 2.5 认证成本与时间

- **GLI RNG认证**：$10,000-$50,000/平台，取决于游戏数量和复杂度
- **GLI加急**：额外25-40%费用，可缩短25-40%时间
- **WLA-SCS审计**：$20,000-$80,000（含ISO 27001如需Level 2）
- **认证周期**：首次6-18个月（包含准备+提交+测试+修正），变更重测3-6个月
- **注意**：GLI不公开标准定价，以上为行业估算

Sources:
- [What is GLI certification and its business value](https://wizards.us/blog/what-is-gli-certification/)
- [How to Get RNG Software Certification - Jurisprudential](https://www.jurisprudential.eu/en/kak-projti-sertifikacziyu-softa-rng/)
- [RNG Certification - Key2Law](https://key2law.com/en/licences/rng-certification/rng-certification)
- [Seals of approval - Slotegrator](https://slotegrator.pro/analytical_articles/seals-of-approval-gain-players-trust-with-certified-games/)

---

## 3. 服务器端预判定架构

### 3.1 完整技术流程

**阶段1 - 奖池生成**：
- HRNG产生种子 -> CSPRNG扩展生成N个随机数 -> 映射到奖金等级
- 奖池设计遵循游戏数学模型（预设返奖率、各等级中奖概率）
- 生成后整个奖池加密存储

**阶段2 - HSM存储与签名**：
- HSM对奖池做数字签名（绑定奖池内容+时间戳）
- 每张票的结果AES-256加密
- 可选：生成Merkle Tree，Root Hash可上链或公开发布

**阶段3 - 购买映射**：
- 玩家购买 -> 系统分配下一张可用票（严格FIFO或随机映射）
- 映射关系由HSM签名（玩家ID + 票号 + 时间戳）
- 不可篡改审计日志记录完整映射过程

**阶段4 - 前端展示**：
- 解密对应票的结果 -> 传输到客户端 -> 刮开动画
- 结果附带验证信息（Merkle Proof或签名）

**阶段5 - 审计验证**：
- 玩家验证：Merkle Proof证明我的票在预判定奖池中
- 审计方验证：完整奖池+签名链+映射日志
- 监管方验证：HSM审计日志+密钥使用记录

### 3.2 安全威胁模型

**威胁1 - 内部人员篡改**（最高风险）：
- 案例：Eddie Tipton事件（Multi-State Lottery Association程序员植入后门，预测中奖号码长达6年）
- 对策：HSM双人控制（M-of-N密钥分片）、操作审计日志、Merkle Root预发布、代码签名+完整性校验、定期第三方审计

**威胁2 - RNG种子预测**：
- 攻击：分析PRNG输出反推种子或内部状态
- 对策：使用CSPRNG（不可从输出反推）、HRNG种子（不可预测）、定期重播种（限制单个种子暴露窗口）

**威胁3 - 时间戳篡改**：
- 攻击：修改开奖/生成时间以伪造顺序
- 对策：HSM内置安全时钟（不可外部修改）、NTP多源校准、时间戳链式签名（每个签名包含前一个签名的哈希）

**威胁4 - 网络中间人攻击**：
- 攻击：拦截玩家-服务器通信，替换结果
- 对策：TLS 1.3双向证书认证、结果附带HSM签名、端到端加密

**威胁5 - 物理入侵**：
- 攻击：直接访问机房，替换RNG硬件或HSM
- 对策：机房物理安防（门禁+监控+入侵检测）、HSM防篡改封装（开箱即销毁密钥）、双站点部署

Sources:
- [Schneier: Insider Attack on Lottery Software](https://www.schneier.com/blog/archives/2017/08/insider_attack_.html)
- [Schneier: Hacking Lottery Machines](https://www.schneier.com/blog/archives/2016/04/hacking_lottery.html)
- [Szrek RNG Security](https://szrek.com/security/)
- [Szrek Deconstructing the RNG Black Box](https://szrek.com/deconstructing_the_rng_black_box/)
- [Szrek Nonrepudiation and Draw Integrity](https://szrek.com/nonrepudiation-draw-integrity/)

### 3.3 HSM供应商对比

**Thales Luna Network HSM 7**：
- 价格：$10,000-$50,000/台（基础款$6,000起，高吞吐$25,000+）
- 年维护：硬件成本15-20%
- 认证：FIPS 140-2 Level 3，Common Criteria EAL4+
- 特点：行业标杆，全球支持网络，彩票行业部署最多
- 适用：大型彩票运营中心

**Utimaco SecurityServer**：
- 价格：$8,000-$40,000/台
- 认证：FIPS 140-2 Level 3
- 特点：德国制造，欧洲合规友好，可定制
- 适用：UEMOA联合乐透（欧洲标准偏好）

**AWS CloudHSM**：
- 价格：$1.45/小时（约$1,050/月，$12,600/年）
- 认证：FIPS 140-2 Level 3
- 特点：云原生，按需扩展，无硬件维护，API友好
- 限制：**断网时不可用**——西非场景不可作为唯一HSM
- 适用：灾备/验证节点，初期成本控制

Sources:
- [PeerSpot: AWS CloudHSM vs Thales Luna HSM](https://www.peerspot.com/products/comparisons/aws-cloudhsm_vs_thales-luna-hsm)
- [HSM Pricing Guide](https://www.esignglobal.com/blog/hsm-hardware-security-module-pricing-deployment)
- [CTO Club: Top 25 HSM Vendors](https://thectoclub.com/tools/best-hsm-vendors/)

---

## 4. 区块链/VRF方案

### 4.1 Commit-Reveal方案

**核心原理**：
1. Commit阶段：运营商生成秘密值S -> 计算H=Hash(S) -> H上链（不可更改）
2. Play阶段：玩家参与（投注/购买），此时只知道H不知道S
3. Reveal阶段：运营商公开S -> 任何人验证Hash(S)==H -> 证明S在玩家参与前已确定

**在e-Scratch中的应用**：
- 奖池生成后计算Merkle Root -> Root Hash上链 -> 玩家购买后可验证票在预判定奖池中
- 成本：仅1次链上交易（存储32bytes哈希），约$0.01-$5（取决于链和gas）
- 安全性：Binding Property（运营商不能改S）+ Hiding Property（玩家不能推导S）

**限制**：
- 不适用实时开奖（KENO/Pick 3等）——每次开奖都上链成本太高且增加延迟
- 依赖运营商最终Reveal——如果运营商拒绝Reveal，需要超时退款机制

Sources:
- [Commit-Reveal Scheme in Solidity - Speedrun Ethereum](https://speedrunethereum.com/guides/commit-reveal-scheme)
- [Commit-Reveal in Cadence - Flow](https://developers.flow.com/blockchain-development-tutorials/native-vrf/commit-reveal-cadence)
- [What is a Commit-Reveal Scheme - Gate.com](https://www.gate.com/learn/articles/what-is-a-commit-reveal-scheme-in-blockchain/862)

### 4.2 Chainlink VRF

**工作原理**：
1. 消费者合约向VRF Coordinator发送请求
2. VRF节点收到请求 -> 结合区块数据（不可预测）+ 节点私钥（秘密）
3. 生成随机数 + 密码学证明（proof）
4. 链上合约验证proof -> 接受随机数

**版本演进**：VRF v2.5（最新）
- 端到端延迟：约2秒
- 支持LINK和原生代币支付（原生代币费率更高）
- 订阅模式降低单次Gas
- 可配置回调gas上限

**成本结构**：
- Gas费（回调交易）+ LINK Premium（按回调gas百分比计算）
- 订阅模式：预充值余额，消费者合约从余额扣费
- 直接资助模式：每次请求单独支付
- 估算单次请求：$0.25-$5（取决于链和gas价格）

**在彩票中的实际应用**：
- PancakeSwap Lottery V2：使用Chainlink VRF，每次开奖公开可审计
- PoolTogether V5：VRF选择无损彩票中奖者
- Megapot（Base链）：VRF驱动日开奖

**局限性**：
- 依赖链下计算和VRF Coordinator合约交互
- 网络拥堵时延迟可能增加
- 需要LINK代币支付（西非获取和管理LINK增加运营复杂度）

### 4.3 其他区块链随机性方案

**RANDAO**：
- 基于多方参与的Commit-Reveal
- 去中心化、社会化、可验证的随机数生成
- 以太坊信标链已内置RANDAO（每个slot的proposer贡献随机性）

**Oasis Network VRF**：
- 基于可信执行环境（TEE）的随机数生成
- 隐私保护特性

### 4.4 监管接受度

- **积极地区**：马耳他、直布罗陀、库拉索——17+运营商在公链发布provably fair哈希（EGBA数据）
- **渐进地区**：欧盟——MiCA框架下逐步建规则，22%会员投注已上链结算
- **观望地区**：西非/UEMOA——国家彩票（LONACI等）持谨慎态度，优先认可GLI/WLA
- **保守地区**：美国——州彩票委员会更认可传统GLI认证

**结论**：在西非场景中，区块链应定位为"增信层"（e-Scratch的Merkle Root锚定）而非"基础层"。监管机构认可的是GLI认证和HSM签名，不是链上哈希。

Sources:
- [Chainlink VRF Documentation](https://docs.chain.link/vrf)
- [Chainlink VRF v2.5 Blog](https://blog.chain.link/introducing-vrf-v2-5/)
- [How Provably Fair Technology is Changing Online Gambling](https://bettoblock.com/provably-fair-technology-online-gambling/)
- [Block3 Finance: Are Crypto Betting Platforms Really Fair?](https://www.block3finance.com/are-crypto-betting-platforms-really-fair-understanding-provably-fair-systems)
- [Blockchain-Enabled Secure and Fair Online Lottery Systems](https://www.researchgate.net/publication/389828903_Blockchain-Enabled_Secure_and_Fair_Online_Lottery_Systems_Using_Smart_Contracts)

---

## 5. NIST SP 800-22随机性测试

### 5.1 NIST STS测试套件

**版本**：SP 800-22 Rev. 1a（Update 1）
**测试项数**：15项（原16项，Lempel-Ziv压缩测试因缺陷被移除）

完整测试列表：
1. **Frequency (Monobit)**：0和1的总体比例是否接近50:50
2. **Frequency within a Block**：M-bit块内0/1比例均匀性
3. **Runs**：连续相同bit的游程长度分布
4. **Longest Run of Ones in a Block**：最长连续1的长度分布
5. **Binary Matrix Rank**：随机矩阵的秩分布（检测线性依赖）
6. **Discrete Fourier Transform (Spectral)**：频域分析检测周期性
7. **Non-overlapping Template Matching**：特定非重叠模式出现频率
8. **Overlapping Template Matching**：特定重叠模式出现频率
9. **Maurer's Universal Statistical**：序列可压缩性（信息熵近似）
10. **Linear Complexity**：序列的线性复杂度分布
11. **Serial**：所有可能m-bit模式的出现频率均匀性
12. **Approximate Entropy**：相邻长度模式的条件概率一致性
13. **Cumulative Sums (Cusums)**：前向和后向累积和的最大偏移
14. **Random Excursions**：随机游走中各状态访问次数分布
15. **Random Excursions Variant**：随机游走状态访问次数的扩展变体

**通过标准**：
- 显著性水平 alpha = 0.01
- 单序列：P-value >= 0.01 即通过
- 多序列：通过比例应在 (1-alpha) +/- 3*sqrt(alpha*(1-alpha)/N) 范围内
- 建议最少样本：1,000,000 bits/序列，1000个序列

### 5.2 Diehard/Dieharder测试

**Diehard**（Marsaglia, 1995）：
- 最早的系统性随机性测试电池之一
- 包含Birthday Spacings、Overlapping Permutations、Ranks、Monkey Tests等

**Dieharder**（Brown, 2006+）：
- Diehard的扩展版本
- 整合NIST STS + 原始Diehard测试
- 总计30项测试，76个变体
- 比NIST STS更严格
- 开源工具，可命令行运行

**三大测试电池比较**：
| 电池 | 测试数 | 严格度 | 用途 |
|------|--------|--------|------|
| NIST STS | 15 | 标准 | GLI认证必须 |
| Dieharder | 30(76变体) | 更严格 | 加分项/内部QA |
| FIPS 140-1/2 | 4 | 基础 | 硬件模块认证 |

Sources:
- [NIST SP 800-22 Rev 1a PDF](https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-22r1a.pdf)
- [NIST CSRC SP 800-22](https://csrc.nist.gov/pubs/sp/800/22/r1/upd1/final)
- [GitHub: NIST Statistical Test Suite](https://github.com/terrillmoore/NIST-Statistical-Test-Suite)

---

## 6. 西非基础设施约束

### 6.1 互联网连接

**2024年西非海缆断裂事件**：
- 2024年3月，WACS、MainOne、SAT-3、ACE等海底光缆受损
- 科特迪瓦连接骤降至约4%
- 利比里亚降至17%
- 贝宁14%、加纳25%
- 修复需数周到数月
- 影响范围：西非和中非多国数百万用户

**常态性问题**：
- 依赖少数海底光缆，单点故障风险高
- 内陆国家（马里、布基纳法索、尼日尔）更依赖邻国过境带宽
- 世界银行评估：撒哈拉以南非洲数字基础设施远落后于其他地区

### 6.2 电力供应

**关键数据**：
- 尼日尔：年均断电288次
- 尼日利亚：2024年全国电网崩溃12次，每次持续数小时到数天
- 科特迪瓦：2024年意外电力危机（作为西非主要电力供应国）
- 企业年损失：尼日利亚超N5.5万亿（不可靠电力）
- 应对：大量企业依赖柴油发电机备用电源

### 6.3 对彩票RNG系统的影响

**断网影响**：
- 在线彩票系统完全不可用
- 云HSM（AWS CloudHSM）不可访问
- 区块链锚定无法执行
- 跨站点同步中断

**断电影响**：
- 服务器/POS终端停机
- RNG硬件断电（HRNG需要持续运行维持熵池）
- HSM可能进入锁定状态（取决于配置）

**低带宽影响**：
- 大批量票数据同步困难
- 实时开奖结果传输延迟
- 区块链交易确认时间增加

### 6.4 降级方案设计原则

1. **本地优先**：关键组件（RNG、HSM、票库）必须有本地实例
2. **离线可运营**：断网时能继续售票和开奖（受限模式）
3. **联网即同步**：恢复连接后自动对账和同步
4. **分级降级**：根据故障程度自动调整服务级别
5. **审计可追溯**：离线期间的所有操作必须有完整日志，联网后可审计

Sources:
- [Africa Internet Outage - Data Center Knowledge](https://www.datacenterknowledge.com/outages/africa-internet-outage-risks-leaving-millions-offline-for-weeks)
- [2024 Internet outage in Africa - Wikipedia](https://en.wikipedia.org/wiki/2024_Internet_outage_in_Africa)
- [VOA: Recent outages highlight need for stronger African internet](https://www.voanews.com/a/recent-outages-highlight-need-for-stronger-african-internet/7703598.html)
- [Top 10 African Countries with Worst Power Outages](https://www.africanexponent.com/top-10-african-countries-with-the-worst-power-outages-in-2024-2025/)
- [Africa Infrastructure Knowledge Portal](https://infrastructureafrica.opendataforafrica.org/kquobdg/)

---

## 7. 关键供应商和参考方案

### 7.1 Szrek2Solutions（可审计RNG先驱）

- 成立：2003年，创始人Irena和Walter Szrek（20+年彩票系统经验）
- 核心创新：数字签名即RNG种子——用HSM数字签名的输出作为随机数种子
- 验证机制：Systemic Independent Audit——独立系统用相同数据重新生成结果，一致则未篡改
- HSM内置不可篡改计数器，每次签名递增，每个随机数都可追溯
- 部署：20+国家彩票运营商（自2005年起）
- 产品：Trusted Draw（开奖）、Trusted Instant（即开票）、Trusted Audit（审计）

**对西非的意义**：Szrek模式证明了"不用区块链也能证明公平"——通过数字签名和独立审计实现可验证性。这对区块链接受度有限的西非监管环境非常重要。

Sources:
- [Szrek2Solutions Official](https://szrek.com/)
- [Szrek Solutions](https://szrek.com/solutions/)
- [Szrek Products](https://szrek.com/products/)
- [WLA: 20 years of deploying automated lottery RNG systems](https://world-lotteries.org/insights/news/member-news/20-years-of-deploying-automated-lottery-rng-systems-around-the-world)

### 7.2 西非彩票运营环境

**科特迪瓦（Ivory Coast）**：
- LONACI：国家彩票运营商，独家经营权（2020年法律No. 2020-480）
- 业务：彩票开奖、即开票、体育博彩、PMU赛马、在线平台（Parions Direct）
- 监管框架：2020年5月27日法律统一了全部博彩领域

**UEMOA区域**：
- 贝宁国家彩票（Loterie Nationale du Benin S.A）准备在BRVM区域证券交易所上市
- WLA正在为非洲彩票运营商提供监管和财政框架支持

Sources:
- [Ivory Coast iGaming Overview - 3S.INFO](https://3s.info/en/reviews/ivory-coast-igaming-overview/)
- [WLA Addressing Africa's regulatory needs](https://www.world-lotteries.org/insights/editorial/blog/addressing-africas-imminent-regulatory-and-fiscal-needs-in-an-exceptional-way)
- [Loterie Nationale du Benin BRVM listing](https://gamblingtalk.net/news/loterie-nationale-du-benin-set-to-become-first-lottery-company-listed-on-brvm)

---

## 8. 核心结论

### 8.1 架构决策总结

| 产品 | 核心RNG | 公平性证明方式 | 区块链角色 |
|------|---------|-------------|-----------|
| e-Scratch | HRNG种子+CSPRNG预生成 | 服务器端预判定+Merkle Root | 锚定层（Commit-Reveal） |
| KENO/Pick 3/Cash Chrono | HRNG种子+CSPRNG实时 | HSM签名+独立审计（Szrek模式） | 不需要 |
| UEMOA联合乐透 | 双HRNG+CSPRNG | 多国见证+HSM签名+视频直播 | 可选锚定 |

### 8.2 为什么区块链更适合e-Scratch而非开奖型彩票

1. **e-Scratch的信任问题**：玩家需要相信"结果在我购买前就确定了，运营商没有在我购买后改结果"——区块链的时间戳锚定完美解决这个问题
2. **开奖型彩票的信任问题**：玩家需要相信"开奖号码是随机的"——这通过认证RNG+HSM签名+独立审计解决，区块链不增加价值但增加延迟和成本
3. **频率差异**：e-Scratch奖池生成是批量操作（每批一次上链），成本可接受。KENO每4分钟/Cash Chrono每2分钟上链则成本过高
4. **监管偏好**：西非监管机构认可GLI认证，对区块链持观望态度

### 8.3 优先行动建议

1. **立即**：选择HRNG+CSPRNG混合方案，开始NIST SP 800-22测试
2. **3个月内**：完成HSM选型采购（建议Thales Luna本地+AWS CloudHSM灾备）
3. **6个月内**：提交GLI-19认证申请（含RNG源码、数学模型、安全文档）
4. **同步进行**：e-Scratch预判定架构开发+Merkle Root区块链锚定PoC
5. **12-18个月**：完成GLI认证+WLA-SCS Level 1评估启动
