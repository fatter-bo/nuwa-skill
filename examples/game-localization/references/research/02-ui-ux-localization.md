# 02-ui-ux-localization: UI/UX本地化适配

## 文本膨胀率（Text Expansion）

从英语翻译到其他语言时的平均文本长度变化：

| 目标语言 | 膨胀率 | 设计建议 |
|---------|-------|---------|
| 德语 | +30%~40% | 按钮/标签预留最大空间 |
| 法语 | +20%~30% | 中等膨胀 |
| 西班牙语 | +20%~30% | 中等膨胀 |
| 俄语 | +20%~35% | 西里尔字母需专用字体 |
| 阿拉伯语 | +25%~30% | 需RTL全面适配 |
| 葡萄牙语 | +15%~25% | 巴西vs葡萄牙差异 |
| 日语 | -10%~+10% | 字符更紧凑但需更大字号 |
| 中文 | -20%~-10% | 最紧凑，但每字信息密度高 |
| 韩语 | -10%~+5% | 谚文组合字母需专门排版 |
| 泰语 | +15%~20% | 声调标记需额外行高 |

**核心原则**：UI设计从一开始就预留30-40%的文本扩展空间。

## RTL语言适配（阿拉伯语、希伯来语、波斯语、乌尔都语）

全球RTL语言使用者超过8.2亿人。

### 必须镜像的元素
- 导航栏方向（菜单从右侧开始）
- 返回/前进箭头方向翻转
- 进度条方向（从右到左填充）
- 列表/卡片布局方向
- 滑动手势方向
- 文本对齐（右对齐为默认）

### 不需要镜像的元素
- 时钟图标（全球通用顺时针）
- 播放/暂停按钮
- 对勾/叉号
- 数字（阿拉伯数字在RTL文本中仍为LTR）
- 品牌logo
- 电话图标

### 技术实现
- CSS: `direction: rtl` + `transform: scaleX(-1)` 选择性镜像
- Unity: TextMeshPro支持RTL，需设置`isRightToLeftText`
- Unreal: Slate UI支持RTL布局

## 字体选择策略

### CJK字体（中日韩）
- **字符集规模**：CJK统一表意文字超50,000字符
- **字体大小**：CJK字符密度高，需要比拉丁字母大20-30%的字号才能保持可读性
- **推荐**：Noto Sans CJK（Google开源，覆盖中日韩三种变体）
- **注意**：同一个汉字在中日韩三国的写法可能不同（如"骨"字）

### 阿拉伯语字体
- 字母有独立/首/中/尾四种形态
- 需要支持连字（ligature）
- 推荐：Noto Naskh Arabic、Cairo

### 泰语字体
- 声调标记在字母上方/下方叠加
- 需要额外行高（至少1.5x）
- 推荐：Noto Sans Thai

### 通用方案
- **Google Noto字族**：覆盖全球所有书写系统的开源字体
- **Fallback链**：主字体→语言专用字体→Noto→系统默认

## 按钮尺寸与触控热区

| 标准 | 最小触控目标 | 来源 |
|------|------------|------|
| Apple HIG | 44x44pt | iOS Human Interface Guidelines |
| Material Design | 48x48dp | Google Material Design |
| WCAG 2.1 | 44x44 CSS px | W3C Web Accessibility |

### 地区差异考量
- **日本**：偏好精致、紧凑的UI，信息密度高
- **中国**：习惯信息密集界面，"信息墙"不被视为负面
- **东南亚**：低端设备屏幕小，需要更大的触控区域
- **欧美**：偏好简洁、留白充足的界面

## 货币与数字格式

| 地区 | 千位分隔符 | 小数点 | 货币符号位置 | 示例 |
|------|----------|--------|------------|------|
| 美国 | 逗号 | 点 | 前置 | $1,234.56 |
| 德国 | 点 | 逗号 | 后置 | 1.234,56 € |
| 法国 | 空格 | 逗号 | 后置 | 1 234,56 € |
| 日本 | 逗号 | 无小数 | 前置 | ¥1,234 |
| 韩国 | 逗号 | 无小数 | 后置 | 1,234원 |
| 中国 | 逗号 | 点 | 前置 | ¥1,234.56 |
| 印度 | 特殊分组 | 点 | 前置 | ₹1,23,456.78 |
| 阿拉伯 | 逗号 | 点 | 后置(多数) | 1,234.56 د.إ |

### 日期格式
| 地区 | 格式 | 示例 |
|------|------|------|
| 美国 | MM/DD/YYYY | 04/12/2026 |
| 欧洲 | DD/MM/YYYY | 12/04/2026 |
| 日本 | YYYY年MM月DD日 | 2026年04月12日 |
| 中国 | YYYY-MM-DD | 2026-04-12 |
| 韩国 | YYYY.MM.DD | 2026.04.12 |

## 来源
- Gridly: 7 Localization Best Practices for Game UI Design
- Ubertesters: Best Localization Testing Practices for RTL Languages
- POEditor: RTL Localization Creating Inclusive Interfaces
- Laoret: Managing Text Expansion for Software and Game Localization
- IGDA Localization SIG: Best Practices for Game Localization v22
