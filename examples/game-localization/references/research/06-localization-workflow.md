# 06-localization-workflow: 本地化工作流与技术实现

## i18n框架选型

### 游戏引擎内建方案

| 引擎 | 内建系统 | 核心特性 | 适用场景 |
|------|---------|---------|---------|
| Unity | Localization Package | StringTable资产、Smart Strings、Locale系统、Addressables集成 | 中小型到大型项目 |
| Unreal Engine | Localization Dashboard | FText类型、ICU库集成、Localization Target、支持音视频本地化 | 大型3A级项目 |
| Godot | TranslationServer | CSV/PO文件格式、gettext兼容 | 独立/中小项目 |
| Cocos Creator | i18n插件 | JSON/CSV资源文件 | 移动端休闲游戏 |

### 第三方本地化平台（TMS）

| 平台 | 特点 | 适用场景 |
|------|------|---------|
| Crowdin | Git/API集成、OTA更新、AI辅助翻译 | 持续更新型游戏 |
| Lokalise | SDK集成、截图OCR、分支管理 | 移动端优先 |
| Phrase (Memsource) | TM/术语库强大、API丰富 | 大规模多语言 |
| POEditor | 轻量级、GitHub集成 | 中小项目 |
| Gridly | 专为游戏设计、支持结构化数据 | 游戏内容管理 |
| Transifex | 连续本地化、CLI工具 | DevOps集成 |

### 文本格式标准

| 格式 | 特点 | 推荐场景 |
|------|------|---------|
| ICU MessageFormat | 复数/性别/选择语法、CLDR本地化规则 | Unreal引擎、复杂文本 |
| gettext (.po/.pot) | 成熟生态、广泛工具支持 | 开源项目、Godot |
| XLIFF | 行业标准交换格式 | 与翻译公司协作 |
| JSON (key-value) | 简单直观、开发者友好 | Web游戏、轻量项目 |
| CSV/TSV | Excel兼容、非技术人员可编辑 | 快速原型、小团队 |
| StringTable (Unity) | Unity原生、编辑器可视化 | Unity项目首选 |

## 翻译质量控制 — LQA测试

### LQA三大支柱

1. **语言测试（Linguistic Testing）**
   - 翻译准确性
   - 术语一致性
   - 语法/拼写/标点
   - 语气/风格匹配
   - 文化适当性

2. **功能测试（Functional Testing）**
   - 文本截断/溢出检测
   - 字符编码正确性
   - 变量/占位符替换正确
   - 日期/货币/数字格式
   - RTL布局正确性

3. **文化测试（Cultural Testing）**
   - 视觉元素文化适当性
   - 音频/配音质量
   - 本地化后的用户体验流畅度
   - 法规合规性检查

### LQA流程

```
目标设定 → 测试准备 → 执行测试 → 缺陷记录 → 修复实施 → 回归测试 → 签发
  ↑                                                              ↓
  └──────────────── 持续改进循环 ──────────────────────────────────┘
```

**关键原则**：翻译和LQA必须由不同人员执行，消除人为偏见。

### 缺陷分类标准

| 严重级别 | 描述 | 示例 |
|---------|------|------|
| Critical | 阻断游戏/法规违规 | 错误概率公示、崩溃、功能无法使用 |
| Major | 严重影响体验 | 关键UI文本截断、误导性翻译 |
| Minor | 影响质量但可玩 | 语法错误、术语不一致 |
| Preferential | 风格改进 | 更优翻译建议、语气微调 |

### LQA指标

| 指标 | 描述 | 目标 |
|------|------|------|
| Error Rate | 每千字错误数 | <3 (Critical 0) |
| Pass Rate | 一次通过率 | >85% |
| Turnaround | 缺陷修复周转时间 | <48h (Critical <24h) |

## 多语言分支策略

### 方案一：主干集成（推荐）
```
main
  ├── locales/
  │   ├── en/         # 源语言
  │   ├── ja/         # 日语
  │   ├── ko/         # 韩语
  │   ├── zh-Hans/    # 简体中文
  │   ├── zh-Hant/    # 繁体中文
  │   ├── ar/         # 阿拉伯语
  │   └── ...
  └── src/
```
- **优点**：所有语言在同一代码库，易于同步
- **缺点**：仓库体积大，构建时间长
- **适用**：中小型项目，语言数量<15

### 方案二：子模块/外部资源
```
main (代码)
  └── git submodule: locales-repo
      ├── en/
      ├── ja/
      └── ...
```
- **优点**：代码和翻译解耦，翻译团队独立工作
- **缺点**：子模块管理复杂
- **适用**：大型项目，翻译外包

### 方案三：OTA（Over-The-Air）更新
- 翻译资源通过CDN/TMS平台动态下载
- **优点**：无需发版即可更新翻译
- **缺点**：首次加载延迟，离线场景需fallback
- **适用**：Live-service游戏，频繁更新

### 方案四：构建时变体
```
CI/CD Pipeline
  ├── build-ja → Japan APK/IPA
  ├── build-ko → Korea APK/IPA
  └── build-global → Global (multi-lang)
```
- **优点**：包体小，地区特定资源优化
- **缺点**：维护多个构建配置
- **适用**：需要不同地区不同功能/美术的项目

## 2026年趋势：AI辅助本地化

### AI在本地化流程中的角色
- **第一遍翻译**：AI处理高量级标准文本（UI标签、系统消息）
- **人工精修**：专业译者聚焦叙事对话、文化敏感内容
- **成本效益**：AI辅助可降低30-50%翻译成本，加速交付
- **质量控制**：AI用于自动检测术语不一致、格式错误

### 推荐工作流
```
源文本更新 → AI初译 → 人工审校 → 上下文校对(in-context review) → LQA测试 → 发布
                ↑                                                        ↓
                └──── 翻译记忆(TM)更新 ←──── 反馈循环 ────────────────────┘
```

## 持续本地化Pipeline

```
开发者提交代码 → CI提取新增/变更字符串 → 推送到TMS平台
       ↓                                      ↓
  自动化测试 ← 导入翻译文件 ← 翻译完成通知 ← 翻译团队处理
       ↓
  构建多语言版本 → 部署
```

**关键集成点**：
- Git hook自动检测string变更
- TMS的Webhook触发翻译任务
- CI/CD中加入pseudo-localization测试（伪本地化——用带重音的字符替换文本以检测UI问题）
- 自动截图对比工具检测视觉回归

## 来源
- LocalizeDirect: LQA - What Is Game Localization Testing
- Gamedeveloper.com: LQA What is Game Localization Testing
- Allcorrect Games: Game Localization in Unity and Unreal Engine
- Transphere: How to Localize a Mobile Game in 2026
- Gridly: Localization Quality Assurance LQA
- Logrus IT Games: Game LQA - What is LQE and How It Works
