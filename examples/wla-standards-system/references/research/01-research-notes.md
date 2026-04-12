# WLA标准体系与Jean-Luc Moner-Banet 调研笔记

> 调研日期：2026-04-12
> 调研目的：为UEMOA西非彩票数字化运营提供WLA认证路径、标准要求、行业领袖视角的完整参考

---

## 一、WLA-SCS安全控制标准调研

### 1.1 标准版本与演进

- 当前版本：WLA-SCS:2024（2024年10月发布）
- 前版本：WLA-SCS:2020
- 标准文档套件：Standard文档 + Guide to Certification + Code of Practice + Briefing + Changes文档

**来源**：
- [WLA-SCS:2024标准文档](https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Standard.pdf)
- [WLA-SCS:2024认证指南](https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Guide-to-Certification.pdf)
- [WLA-SCS:2024简报](https://www.world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Briefing.pdf)
- [WLA-SCS:2024变更文档](https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Changes.pdf)
- [WLA-SCS:2024操作规范](https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202503_EN_WLA-SCS-2024_Code-of-Practice.pdf)

### 1.2 标准结构详情

WLA-SCS由四个附录组成，指定了博彩运营商和供应商有效管理安全与完整性所需的最低控制：

**Annex A - G Controls（组织控制）**：
- 整合ISO/IEC 27001合规（全球范围）+ 额外30项WLA基础控制
- ISO 27001:2022有93项控制（Annex A），WLA额外叠加30项
- 要求建立安全论坛/高管层面的组织架构监督ISMS
- 安全职能需制定与整体业务一致的安全策略

**Annex B - L Controls（博彩运营控制）**：
- 62项博彩特定安全与完整性控制
- 代表当前博彩运营最佳实践
- L控制对博彩运营商强制适用
- 核心领域：
  - 随机数生成（RNG）：随机性+完整性
  - 支付方式与玩家账户
  - 体育/电竞/赛马博彩
  - 游戏设计与审批：确保合规+完整性+反欺诈

**Annex C - S Controls（供应商控制）**：
- 21项控制
- 适用于：向博彩行业提供产品和服务的供应商 + 自行开发/管理游戏系统的运营商
- 采用风险为基础的方法

**Annex D - M Controls（多辖区游戏控制）**：
- 11项控制
- 适用于跨境游戏参与者

**来源**：
- [WLA-SCS:2024在线版](https://publications.world-lotteries.org/security-and-risk-management/wla-security-control-standard)
- [WLA SCS FAQ](https://www.world-lotteries.org/services/industry-standards/security-and-risk-management/scs-faq)

### 1.3 与ISO 27001的关系

**核心关系**：WLA-SCS以ISO 27001为基础层，叠加博彩行业特定要求。

- SCS Level 2强制要求持有有效ISO/IEC 27001证书
- ISO 27001的ISMS范围必须是全球范围（global scope），覆盖所有博彩相关活动
- 这意味着不能只认证部分业务——必须全覆盖
- SCS Level 1不强制要求ISO 27001，但需要实质性ISMS

**关键差异**：
- ISO 27001允许限定ISMS范围；WLA-SCS要求全球范围
- ISO 27001是3年周期+年度监督审计；WLA-SCS是3年周期+年度强制审计（必须覆盖所有控制项）
- ISO 27001是通用信息安全；WLA-SCS增加62项博彩特定控制

**来源**：
- [FB Pro: WLA-SCS与彩票加固](https://www.fb-pro.com/wla-scs-and-lottery-hardening/)
- [Bulletproof: WLA认证变更](https://content.bulletproofsi.com/wla-certification-changes)

### 1.4 认证级别与审计流程

**Level 1**：
- 适用于WLA Regular Members（运营商）
- 不要求ISO 27001
- 需通过WLA ASE外部评估
- 所有场所必须实地审计

**Level 2**：
- 适用于WLA Regular Members和Associate Members（供应商）
- 强制要求持有有效ISO 27001（全球范围）
- ISO 27001有效性需在每次年度评估时验证
- WLA ASE外部评估

**年度审计要求**：
- 3年认证周期内，每年必须进行控制评估
- 年度审查时抽样审计适用控制项
- 3年内所有适用控制项至少审计一次

**认可的审计服务实体（ASE）**：
- Gaming Associates
- SGS
- TUV NORD
- GLI
- Bulletproof

**来源**：
- [WLA认证指南在线版](https://publications.world-lotteries.org/security-and-risk-management/guide-certification-wla-security-control-standard)
- [WLA Affiliated ASE列表](https://world-lotteries.org/services/industry-standards/security-and-risk-management-2/affiliated-assessment-service-entities)
- [SGS WLA-SCS认证服务](https://www.sgs.com/en-us/services/wla-scs-2020-lottery-certification)
- [Gaming Associates WLA评估](https://gamingassociates.com/wla-assessments/)
- [TUV NORD WLA-SCS认证](https://www.tuv-nord.com/gr/en/pistopoiisi/systems-certification/cybersecurity/wla-scs-certification/)

---

## 二、WLA负责任博彩框架调研

### 2.1 框架结构

4级认证 + 10大项目要素

**4级定义**：
- Level 1 = Assessing（评估/承诺）：入会自动获得
- Level 2 = Planning（规划）：自评估+差距分析+实施计划
- Level 3 = Implementing（实施）：RG原则融入日常运营
- Level 4 = Continuous Improvement（持续改善）：全面成熟+第三方评估+创新

**10大项目要素**：
1. Research（研究）
2. Employee Program（员工项目）
3. Retailer Program（零售商项目）
4. Game Design（游戏设计）
5. Remote Gaming Channels（远程博彩渠道）
6. Advertising & Marketing Communications（广告与营销沟通）
7. Player Education（玩家教育）
8. Treatment Referral（治疗转介）
9. Stakeholder Engagement（利益相关者参与）
10. Reporting & Measurement（报告与测量）

**来源**：
- [WLA RG Framework 2025](https://publications.world-lotteries.org/responsible-gaming/wla-certification-framework-2025)
- [WLA Framework页面](https://www.world-lotteries.org/services/industry-standards/responsible-gaming-framework/framework-2025)
- [WLA RGF提交指南2024](https://publications.world-lotteries.org/guides-articles/rg-certification-the-wla-responsible-gaming-framework-and-certification)

### 2.2 各级别详细要求

**Level 2要求**：
- 完成10要素的自评估
- 确定需要建立哪些RG项目
- 制定实施计划（时间表+预算）
- 提交至WLA独立评审团审查
- 入会后18个月内必须获得

**Level 3要求**：
- 员工培训实施
- 负责任博彩政策有效沟通
- 玩家保护措施实施
- 各要素从"计划"到"执行"的证据

**Level 4要求**：
- 全面发展的RG项目体系
- 遵循全球最佳实践
- 持续改进和创新证据
- 必须有第三方独立评估
- 玩家保护具体措施：ID验证、存款限额、完整交易记录、自排除
- 营销材料全部融入RG信息和求助热线
- 投资研究识别早期博彩伤害

**评审机制**：
- Level 2-4由独立国际评审团审查
- 评审团由企业社会责任领域的国际专家组成
- 提交截止日：每年5月1日和10月1日

**来源**：
- [DigitalRG: WLA认证](https://digitalrg.com/world-lottery-association-certifications)
- [WLA RGF FAQ 2025](https://www.world-lotteries.org/services/industry-standards/responsible-gaming-framework/rgf-faq-2025)
- [WLA RGF Level 4模板](https://world-lotteries.org/volumes/downloads/Download_Center/Responsible_Gaming/Regular_Members/201411_WLA-RGF_Level_4_Template_2021.pdf)
- [NASPL与WLA框架对齐](https://www.nasplinsights.com/post/naspl-and-wla-confirm-alignment-of-responsible-gaming-frameworks)
- [伊利诺伊州彩票获Level 4](https://www.illinoislottery.com/illinois-lottery/press-and-media-center/press-release/2026/2/illinois-lottery-achieves-highest-level-of-responsible-gaming-certification-from-world-lottery-association)
- [俄勒冈彩票获最高认证](https://www.oregonlottery.org/press-releases/wla-certification-2026/)

### 2.3 认证费用与工具支持

- RG认证过程不额外收费，包含在WLA会员服务中
- 所有会员可使用DigitalRG工具
- 启动Level 2认证的会员可获8小时咨询服务
- WLA举办RG认证支持研讨会

**来源**：
- [WLA RG认证支持研讨会](https://publications.world-lotteries.org/blog-posts/wla-webinar-highlights-support-and-tools-for-responsible-gaming-certification)
- [DigitalRG平台](https://digitalrg.com/certifications-and-standards)

---

## 三、WLA认证实操信息

### 3.1 WLA会员申请

**会员类别**：
- Regular Membership：国家授权彩票/体育博彩运营商
- Associate Membership：供应商
- Collaborating Membership：观察员
- Contributorship：深度参与的Associate Members

**费用结构（Regular Members）**：
- 年销售额 < 5亿美元：CHF 7,000/年
- 5亿-10亿美元：CHF 8,400/年
- > 10亿美元：CHF 14,000/年

**申请流程**：
1. 审查条款和条件、资格标准、费用
2. 下载并完成申请表
3. 附带证明文件提交至WLA商务办公室
4. WLA执委会审批

**会员福利**：
- WLS峰会优惠费率
- WLA数据年鉴独家访问权
- WLA Academy培训参与权
- RG认证工具访问
- 奖学金申请资格

**来源**：
- [WLA如何加入](https://world-lotteries.org/stakeholders/membership/how-to-join)
- [WLA会员福利](https://world-lotteries.org/stakeholders/membership/benefits-of-membership)
- [WLA成为会员](https://world-lotteries.org/members/become-a-member)

### 3.2 WLA Academy与奖学金

**WLA Academy**：
- 全年在全球不同地点举办研讨会、工作坊、圆桌会议
- 面向高管、中层管理者和入门级人员
- 行业内外专家分享知识
- 与区域协会（ALA等）联合举办区域主题活动

**奖学金项目**：
- 2013年启动
- 资格：WLA Regular Member机构员工，18岁以上
- 覆盖差旅费用
- 申请方式：活动前6周提交奖学金申请表+活动注册表+酒店表至academy@world-lotteries.org

**来源**：
- [WLA Academy](https://world-lotteries.org/events-education/academy)
- [WLA奖学金](https://world-lotteries.org/events-education/academy/scholarships)
- [WLA奖学金申请表](https://world-lotteries.org/volumes/downloads/Download_Center/Scholarships/WLA-Academy_Scholarship-application-form_NEW-2025.pdf)
- [奖学金项目介绍](https://publications.world-lotteries.org/blog-posts/scholarship-program-benefits-wla-lottery-and-sports-betting-members)
- [WLA研讨会](https://www.world-lotteries.org/events-education/academy/seminars)

---

## 四、Jean-Luc Moner-Banet调研

### 4.1 个人背景

- 全名：Jean-Luc Moner-Banet
- 出生：1964年3月14日，法国蓬塔利耶（Pontarlier）
- 家族：加泰罗尼亚裔
- 国籍：法裔瑞士

**教育**：
- 1983年：微型计算机程序员文凭（贝桑松）
- 公共管理硕士MPA（IDHEAP，洛桑大学）

**职业经历**：
- 机床行业工作
- 1989年：加入彩票终端机制造公司
- 1998年：加入Loterie Romande，负责开发Tactilo（欧洲首个电子彩票）
- 2001年：升任副总经理
- 2006年：获MPA后短暂加入GTech担任欧洲开发总监
- 2007年至今：Loterie Romande CEO

**行业领导角色**：
- 2008年：加入WLA执行委员会
- 2012年：全票当选WLA主席（2年任期）
- 2012-2018年：WLA主席（达到任期上限离任）
- 2013年：入选彩票行业名人堂（Lottery Industry Hall of Fame）
- 2019年：就任WLA体育博彩诚信委员会（BISHRC）主席
- 2025年9月：当选ULIS主席

**来源**：
- [Jean-Luc Moner-Banet维基百科](https://fr.wikipedia.org/wiki/Jean-Luc_Moner-Banet)
- [ULIS选举Moner-Banet为主席](https://ulis.org/news-events/news/ulis-elects-jean-luc-moner-banet-as-president)
- [Gaming Intelligence报道](https://www.gamingintelligence.com/people/17522-loterie-romande-ceo-appointed-as-new-wla-president)
- [WoTA Forum演讲者页面](https://wota-forum.events/speaker/jean-luc-moner-banet/)
- [SBC News报道](https://sbcnews.co.uk/europe/2025/09/19/moner-banet-ulis-president)

### 4.2 公开演讲与立场

#### 反非法博彩（最核心主题）

- 创立ELMS（欧洲彩票监测系统）——第一个合作监测平台
- 推动ELMS → GLMS（2015上线）→ ULIS（2022升级），2025年当选ULIS主席
- 核心论点：在"不过度监管"和"保持合法产品吸引力"之间寻找平衡
- 强调教育投注者选择合法运营商的理由：保护体育完整性+资金回馈公益
- 非法运营商对体育完整性和公益资金的双重威胁

**来源**：
- [WLA增强打击体育赛事操纵](https://publications.world-lotteries.org/magazine-articles/67-03-wla-enhances-efforts-to-combat-sports-competition-manipulation)
- [ULIS选举报道（European Gaming）](https://europeangaming.eu/portal/latest-news/2025/09/22/192032/ulis-elects-jean-luc-moner-banet-as-president/)
- [WLA打击非法博彩委员会主席访谈](https://publications.world-lotteries.org/magazine-articles/66-03-new-committee-chair-discusses-wla-work-to-combat-illegal-gambling-activities)
- [WLA主席访谈：打击非法彩票和博彩运营商](https://publications.world-lotteries.org/magazine-articles/66-01-combatting-illegal-lotteries-and-betting-operators)
- [理解非法博彩对体育完整性的关键](https://world-lotteries.org/insights/editorial/blog/understanding-illegal-betting-is-key-to-maintaining-the-integrity-of-sport)

#### 数字化转型

- 2010年推出Loterie Romande线上平台
- 2011年推出首个智能手机博彩APP
- 近年推动零售数字化：LoRoPlay终端、QR码、无现金券
- 在ULIS主席就职时强调"技术转型影响体育博彩社区"
- 务实路径：数字化补充而非替代零售

**来源**：
- [PGRI访谈：未来防护IT架构与策略](https://www.publicgaming.com.publicgamingresearchinstitute.com/news-categories/lottery/11800-pgri-interviews-jean-luc-moner-banet-chief-executive-officer-loterie-suisse-romande-the-it-architecture-and-strategy-to-future-proof-your-business)
- [EL访谈：Loterie Romande第五次RG认证](https://publicgaming.com/news-categories/responsible-gaming/9057-el-interview-with-jean-luc-moner-banet-ceo-of-loterie-romande-loro-rg-certified-for-fifth-time)

#### ESG与可持续发展

- 将可持续发展融入彩票运营
- 推动减少环境影响
- 100%利润归公益本身就是ESG典范

#### 对新兴/小市场的参考价值

- 30+年在~200万人口市场运营的经验
- 证明小市场可以达到全球最高标准（RG Level 4连续五次）
- 对UEMOA的直接参考价值

### 4.3 PGRI访谈要点

PGRI对Moner-Banet的专题访谈聚焦"未来防护IT架构与策略"，显示其对技术基础设施现代化的重视。

**来源**：
- [PGRI数字图书馆：Moner-Banet文章合集](https://www.pgridigitallibrary.com/articles-jean-luc-moner-banet.html)

---

## 五、Loterie Romande调研

### 5.1 基本信息

- 全称：Societe de la Loterie de la Suisse Romande
- 成立：1937年
- 总部：瑞士
- 运营区域：瑞士法语区6州（Vaud, Fribourg, Valais, Neuchatel, Geneva, Jura）
- 服务人口：~200万（法语区）
- 63%的法语区瑞士人口是Loterie Romande玩家

### 5.2 财务数据

**2024年**：
- 毛博彩收入：CHF 4.382亿
- 公益分配：CHF 2.582亿（历史最高）
- 受益：近5,000个项目和协会
- 受益领域：体育、文化、社会行动、环境

**2023年**：
- 公益分配：CHF 2.437亿
- 零售佣金：CHF 8,000万（支付给2,400个零售商）

**利润分配机制**：
- 100%利润分配给非营利机构
- 各州分配按人口和毛博彩收入计算，各占50%权重
- 严格遵守联邦、州际和州级立法

**来源**：
- [WLA报道：2023年年度结果](https://world-lotteries.org/insights/news/member-news/loterie-romandes-annual-results-for-2023-243-7-million-francs-for-welfare-sport-culture-and-the-environment)
- [PGRI报道：2024年年度结果](https://publicgaming.com/news-categories/financial/14522-switzerland-loterie-romande-2024-results-exceptional-support-of-258-2-million-francs-to-support-social-action-sport-culture-and-the-environment)
- [WLA报道：2023年年度报告](https://world-lotteries.org/insights/news/member-news/loterie-romande-2023-annual-report)

### 5.3 数字化实践

**历史里程碑**：
- 1998年：开发Tactilo（欧洲首个电子彩票）
- 2010年：线上博彩平台上线
- 2011年：首个智能手机博彩APP

**近期零售数字化（2023-2024）**：
- 与adesso合作推进零售数字化
- LoRoPlay终端：用于体育博彩的新一代零售终端
- 移动应用生成QR码，兼容传统终端和新LoRoPlay系统
- 券管理平台：减少现金使用
- 云基础设施迁移：现代化、可扩展、高可用的云架构

**来源**：
- [adesso: Loterie Romande零售数字化](https://www.adesso.ch/en/news/press/loterie-romande-accelerates-its-retail-digitization-with-adesso-schweiz.jsp)
- [adesso成功案例](https://www.adesso.ch/en/company/success-stories-how-we-work/loterieromande-loro.jsp)

### 5.4 负责任博彩成就

- WLA RG Level 4认证——连续第五次获得
- 代表全球负责任博彩最高标准
- 对小市场运营商是极具说服力的标杆

**来源**：
- [EL访谈：第五次RG认证](https://publicgaming.com/news-categories/responsible-gaming/9057-el-interview-with-jean-luc-moner-banet-ceo-of-loterie-romande-loro-rg-certified-for-fifth-time)

---

## 六、ALA与非洲彩票生态调研

### 6.1 ALA基本信息

- 全称：Association des Loteries d'Afrique（非洲彩票协会）
- 性质：非营利组织
- 会员：14个非洲国家的彩票运营商，来自13个国家
- 目标：成为泛大陆组织
- WLA地位：五大区域协会之一（ALA, APLA, CIBELAE, EL, NASPL）

### 6.2 WLA合作与资源

- WLA执委会有ALA代表席位
- 2026年与WLA合作举办治理与诚信研讨会
- 计划2026年系列培训：运营最佳实践、负责任博彩框架、网络安全
- WLA Academy与ALA联合举办区域主题活动
- WLA关于非洲彩票创新与可持续发展的专题文章

**来源**：
- [ALA与WLA治理培训研讨会2026](https://focusgn.com/africa/african-lotteries-association-strengthens-governance-training-with-wla-webinar)
- [WLA区域协会页面](https://www.world-lotteries.org/about-us/partners/regional-associations)
- [WLA非洲彩票创新与可持续发展](https://www.world-lotteries.org/insights/editorial/blog/innovation-and-sustainable-development-for-lotteries-in-africa)

---

## 七、WLA会议与国际资源调研

### 7.1 世界彩票峰会（WLS）

- 每2年举办一次
- WLS 2024：巴黎，2024年10月，1000+参会者
- WLS 2026：悉尼，达令港，2026年11月9-12日
- 内容：商务议程+行业演讲+供应商展会+社交活动
- 规模：800-1500人，全球最大彩票行业盛会

### 7.2 WLA宣传倡导（Advocacy）

- WLA积极开展行业宣传倡导工作
- 代表合法彩票行业在国际层面发声
- 推动负责任监管框架

**来源**：
- [WLS 2024](https://publications.world-lotteries.org/world-lottery-summit/wls-2024)
- [WLA活动日历](https://world-lotteries.org/events-education/events-calendar)
- [WLA世界彩票峰会](https://www.world-lotteries.org/events-education/world-lottery-summit)
- [WLA宣传倡导](https://www.world-lotteries.org/services/advocacy)
- [WLA关于我们](https://world-lotteries.org/about-us)

---

## 八、体育博彩诚信调研

### 8.1 ELMS → GLMS → ULIS演进

- 2011年：WLA和EL开始合作创建全球体育博彩监测系统
- 基础：欧洲彩票监测系统（ELMS）——Moner-Banet是创始人之一
- 2015年：GLMS（Global Lottery Monitoring System）上线
- 2022年：升级为ULIS（United Lotteries for Integrity in Sports）
- ULIS使命：团结全球国家授权彩票运营商，保护体育赛事免受操纵和非法博彩威胁

### 8.2 WLA BISHRC

- Betting Integrity on Sports & Horse Racing Committee
- Moner-Banet自2019年担任主席
- 协调跨境打击假球和非法博彩

**来源**：
- [WLA GLMS：全球体育诚信倡导](https://www.world-lotteries.org/insights/editorial/blog/glms-championing-sports-integrity-worldwide)
- [ULIS选举Moner-Banet](https://ulis.org/news-events/news/ulis-elects-jean-luc-moner-banet-as-president)
- [Gaming Intelligence: ULIS选举](https://www.gamingintelligence.com/responsible-gambling/sports-integrity/219970-ulis-elects-former-wla-president-jean-luc-moner-banet/)

---

## 九、关键文档下载链接汇总

### WLA-SCS:2024

| 文档 | 链接 |
|------|------|
| 标准全文 | https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Standard.pdf |
| 认证指南 | https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Guide-to-Certification.pdf |
| 操作规范 | https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202503_EN_WLA-SCS-2024_Code-of-Practice.pdf |
| 简报 | https://www.world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Briefing.pdf |
| 变更说明 | https://world-lotteries.org/volumes/downloads/Download_Center/Security/WLA_SCS_2024/202410_EN_WLA-SCS-2024_Changes.pdf |

### WLA负责任博彩

| 文档 | 链接 |
|------|------|
| RGF 2025框架 | https://publications.world-lotteries.org/responsible-gaming/wla-certification-framework-2025 |
| RGF提交指南2024 | https://publications.world-lotteries.org/guides-articles/rg-certification-the-wla-responsible-gaming-framework-and-certification |
| Level 4模板 | https://world-lotteries.org/volumes/downloads/Download_Center/Responsible_Gaming/Regular_Members/201411_WLA-RGF_Level_4_Template_2021.pdf |
| RGF FAQ 2025 | https://www.world-lotteries.org/services/industry-standards/responsible-gaming-framework/rgf-faq-2025 |

### WLA会员

| 文档 | 链接 |
|------|------|
| 如何加入 | https://world-lotteries.org/stakeholders/membership/how-to-join |
| 会员福利 | https://world-lotteries.org/stakeholders/membership/benefits-of-membership |
| 奖学金申请表 | https://world-lotteries.org/volumes/downloads/Download_Center/Scholarships/WLA-Academy_Scholarship-application-form_NEW-2025.pdf |

---

## 十、待深入调研方向

1. **WLA-SCS:2024完整控制项清单**：需要下载标准全文PDF获取所有控制项的详细描述
2. **UEMOA国家中已有WLA会员的彩票运营商**：确认是否有西非同行已在WLA体系内
3. **WLA认证的非洲成功案例**：ALA 14个会员中哪些已获得SCS或RG认证
4. **FDJ在非洲的运营经验**：FDJ对合作伙伴WLA认证的具体要求
5. **Allwyn的合规要求**：Allwyn在不同市场对合作伙伴的标准要求
6. **WLA-SCS外部评估的实际周期和费用**：需要直接联系ASE获取报价
7. **DigitalRG平台的具体功能**：用于RG自评估和认证提交的工具详情
