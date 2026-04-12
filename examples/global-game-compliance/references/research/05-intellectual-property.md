# 05 知识产权合规调研

调研时间：2026-04-12
调研范围：游戏高仿的侵权边界、玩法/美术/音乐/代码版权、商标风险、开源许可证

---

## 1. 游戏知识产权的法律框架

### 游戏的IP组成

一款游戏是多种知识产权的复合体：

| IP类型 | 保护对象 | 保护方式 |
|--------|----------|----------|
| 版权（Copyright） | 源代码、美术资产、音乐/音效、剧情/对话文本、UI设计的具体表达 | 自动产生，无需注册 |
| 商标（Trademark） | 游戏名称、Logo、标志性角色名称、标语 | 需注册（注册更强保护） |
| 专利（Patent） | 独创的技术方案或交互方法 | 需申请授权 |
| 商业秘密（Trade Secret） | 服务器端算法、匹配算法、运营数据 | 通过保密措施保护 |

### 核心原则：Idea vs Expression

版权法保护的是"表达"（expression），不保护"思想"（idea）。

- **不受版权保护**：游戏规则、玩法机制、游戏类型、数值系统设计思路
- **受版权保护**：这些规则和机制的具体视觉/代码表达

这就是为什么大量"类似"游戏合法存在——Match-3的玩法思想不受保护，但Candy Crush的具体糖果设计、地图界面、动画效果受保护。

## 2. 高仿游戏的侵权边界

### 安全区（一般不构成侵权）

| 行为 | 原因 | 示例 |
|------|------|------|
| 使用相同玩法类型 | 游戏规则是"思想"，不受版权保护 | 做一款新的Match-3游戏 |
| 使用相同数学模型 | 概率分布、数值公式属于数学原理 | 使用德州扑克标准概率 |
| 实现相同游戏规则 | 规则本身不受保护 | 实现麻将的标准规则 |
| 使用类似的UI布局 | 功能性布局一般不受保护 | 牌桌布局（玩家位置+公共牌区） |
| 使用相似的游戏流程 | 流程属于"方法"，一般不受版权保护 | 发牌→下注→开牌的流程 |

### 危险区（可能侵权）

| 行为 | 风险 | 说明 |
|------|------|------|
| 复制美术资产 | 高（版权侵权） | 直接使用他人的角色设计、背景图、图标 |
| 复制音乐/音效 | 高（版权侵权） | 使用他人的BGM、特效音 |
| 反编译复制代码 | 高（版权侵权+可能违反DMCA/CFAA） | 反编译他人App并复用代码 |
| 使用相似名称 | 中-高（商标侵权） | "Candy Smash" vs "Candy Crush" |
| 复制角色设计 | 高（版权侵权） | 使用高度相似的角色外观 |
| 复制具体关卡设计 | 中（版权可能保护"选择与编排"） | 逐关复制关卡数据 |
| 抄袭UI的具体视觉设计 | 中（超越功能性的装饰性设计可能受保护） | 完全一致的配色/图标/动画 |

### 关键判例参考

| 案例 | 结果 | 意义 |
|------|------|------|
| Tetris Holding v. Xio Interactive (2012) | 原告胜 | 即使玩法不受保护，"look and feel"的整体相似性可构成侵权 |
| Spry Fox v. LOLApps (Triple Town) (2012) | 和解（倾向原告） | 游戏规则的"具体选择与编排"可能受保护 |
| DaVinci Editrice v. Ziko Games (Bang!) (2014) | 被告胜 | 简单的扑克牌游戏规则不受版权保护 |
| AIPPI 2025大会讨论 | 持续争议 | 游戏机制的版权保护仍是全球争议焦点 |

## 3. 商标风险管理

### 高风险行为

| 行为 | 风险等级 | 建议 |
|------|----------|------|
| 游戏名称含知名品牌词 | 极高 | 绝对避免 |
| 游戏名称与知名游戏相似 | 高 | 需商标检索确认 |
| 在ASO关键词中使用竞品名 | 中-高 | Apple/Google可能下架 |
| 在描述中提及竞品做对比 | 中 | 可能触发DMCA投诉 |
| 使用知名游戏角色名称 | 高 | 角色名可能有商标保护 |

### 商标检索建议

在命名前，至少检索以下商标数据库：

- 美国：USPTO TESS (https://tmsearch.uspto.gov)
- 欧盟：EUIPO eSearch (https://euipo.europa.eu)
- 中国：中国商标网 (https://sbj.cnipa.gov.cn)
- 日本：J-PlatPat (https://www.j-platpat.inpit.go.jp)
- 国际：WIPO Global Brand Database (https://branddb.wipo.int)

## 4. 开源许可证合规

### 游戏开发常用开源组件的许可证风险

| 许可证 | 风险等级 | 关键义务 | 常见于 |
|--------|----------|----------|--------|
| MIT | 低 | 保留版权声明即可 | 大量JS/TS库 |
| Apache 2.0 | 低 | 保留声明 + 注明修改 + 专利授权 | Google库、引擎组件 |
| BSD 2/3-Clause | 低 | 保留声明 | 底层库 |
| LGPL 2.1/3.0 | 中 | 动态链接一般OK；静态链接需提供用户替换能力 | 部分C/C++库 |
| GPL 2.0/3.0 | 高 | **传染性**——使用GPL代码的项目整体需开源 | Copyleft库、部分引擎 |
| AGPL 3.0 | 极高 | 即使通过网络提供服务也需开源 | 部分服务端组件 |
| Creative Commons (CC) | 视子类型 | CC BY: 需归属；CC BY-SA: 传染；CC BY-NC: 禁商用 | 美术/音频资源 |
| Unlicense/CC0 | 极低 | 无义务 | 部分小工具 |

### 合规最佳实践

1. **建立SBOM**（Software Bill of Materials）：记录所有第三方依赖及其许可证
2. **自动化检查**：使用工具（如FOSSA、Snyk、license-checker）扫描依赖树
3. **GPL隔离**：如必须使用GPL组件，确保动态链接或进程隔离
4. **资源许可审计**：美术/音频资源需确认商用授权（特别注意CC BY-NC不可商用）
5. **NOTICE文件**：在App中包含所有开源组件的归属声明
6. **AI生成内容风险**：AI生成的代码/美术可能包含训练数据中的受保护内容——需人工审查

## 5. 棋牌游戏特殊IP风险

### 传统棋牌

- 扑克、麻将、象棋、围棋等传统游戏规则属于**公共领域**，任何人可自由使用
- 但特定变体的品牌名称可能有商标（如"Texas Hold'em"一般视为通用术语，但某些商业变体名称可能有商标）

### 现代桌游/卡牌游戏

- 规则本身不受版权保护，但规则手册的文字表达受保护
- 卡牌上的美术设计受版权保护
- 游戏名称通常有商标保护
- 独创的游戏组件设计可能有外观设计专利

## 6. 来源

- Wikipedia: Intellectual property protection of video games — https://en.wikipedia.org/wiki/Intellectual_property_protection_of_video_games
- Wikipedia: Video game clone — https://en.wikipedia.org/wiki/Video_game_clone
- Perkins Coie: Rules of the Game — https://perkinscoie.com/insights/publication/rules-game-are-rules-and-mechanics-video-games-copyrightable
- FKKS: How Courts View Copyright Protection For Video Games — https://fkks.com/news/how-courts-view-copyright-protection-for-video-games
- Gamedeveloper: Analysis Clone Games & Fan Games Legal Issues — https://www.gamedeveloper.com/game-platforms/analysis-clone-games-fan-games----legal-issues
- Asia IP: AIPPI 2025 Protecting creativity in gaming — https://www.asiaiplaw.com/section/cover-story/aippi-2025-protecting-creativity-in-the-us200-billion-global-gaming-arena
- ScienceDirect: Videogames and their clones — https://www.sciencedirect.com/science/article/abs/pii/S0267364916300796
