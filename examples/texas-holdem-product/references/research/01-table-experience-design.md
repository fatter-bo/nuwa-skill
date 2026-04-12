# 维度一：牌桌体验设计

## 1. UI/UX 行业标准与设计原则

### 牌桌核心界面元素
德州扑克App牌桌UI需包含以下必备元素：
- **座位区**：圆桌布局（2-10人），每个座位显示头像、昵称、筹码量、当前状态（思考中/已弃牌/全下）
- **公共牌区**：居中展示，翻牌/转牌/河牌依次呈现，配合翻牌动画
- **底池区**：实时显示主池与边池金额
- **操作区**：弃牌/跟注/加注三大按钮 + 加注滑块/输入框
- **计时器**：每人操作倒计时（通常15-30秒），时间银行机制

来源: [Poker House App Design Case Study](https://limeup.io/projects/poker-house/)
来源: [PokerStars Mobile App Design](https://lackabane.com/pokerstars-mobile-app-experience-design)

### PokerStars 设计方法论
PokerStars进行了全面的移动App改版，核心方法论：
- 基于玩家动机与行为的分析（motivational psychology of mobile poker players）
- 功能按使用频率和重要性分级设计
- 横屏/竖屏双模式支持
- 多层级游戏过滤的大厅信息架构
- 设备无关的统一设计规范（iOS/Android一致性）

来源: [Lackabane - PokerStars Mobile App Experience Design](https://lackabane.com/pokerstars-mobile-app-experience-design)

### 视觉设计最佳实践
- **深色背景为主**：减少长时间游戏的视觉疲劳（PokerStars经典绿色牌桌布）
- **高对比度信息**：筹码数字、底池金额使用高对比色
- **动效克制**：发牌/翻牌/推池动画流畅但不过度，避免影响决策速度
- **牌面可读性**：牌面设计需在小屏幕上清晰可辨（四色牌可选）

来源: [Premium Poker Game UI/UX Design](https://www.slideshare.net/slideshow/premium-poker-game-ui-ux-design-high-stakes-casino-app-concept/286144898)

## 2. 快速操作系统

### 预设按钮（Pre-action Buttons）
在轮到玩家操作前，可提前选择动作：
- **自动弃牌 (Auto Fold)**：当手牌较弱时提前选择
- **跟注任何 (Call Any)**：无论对手加注多少都跟注
- **过牌/弃牌 (Check/Fold)**：若可过牌则过牌，否则弃牌
- **预设加注额**：提供1/2底池、3/4底池、满池、2x前注等快捷按钮

产品价值：预设按钮直接影响手/小时（hands per hour）指标，是提升游戏节奏的核心UX组件。

### 快速加注交互
- **滑块 + 数字输入**：滑块拖动选择金额，点击数字可精确输入
- **快捷倍数按钮**：Min Raise / 2x / 3x / Pot / All-in
- **底池比例按钮**：1/3 Pot / 1/2 Pot / 2/3 Pot / Full Pot
- **记忆功能**：记住玩家常用的加注尺寸偏好

### 自动操作（Auto-Actions）
- **自动补盲 (Auto Post Blinds)**：避免每手牌确认
- **自动买入 (Auto Top-Up)**：筹码低于设定值时自动补到最大买入
- **自动坐回 (Auto Sit Back)**：离开后自动回到座位
- **静音/免打扰模式**：关闭聊天和表情通知

## 3. 多桌同时游戏的UI设计

### 桌面端多桌方案
- **平铺模式 (Tiled)**：所有牌桌等分屏幕空间，同时可见
- **层叠模式 (Cascaded)**：牌桌窗口层叠，当前操作桌置顶
- **标签页模式 (Tabbed)**：标签栏切换，节省屏幕空间

来源: [Jurojin Poker - Multi Table Software](https://jurojinpoker.com/)

### 移动端多桌方案
- **滑动切换**：左右滑动在不同牌桌间切换
- **缩略图导航**：屏幕边缘显示其他牌桌缩略图
- **操作优先弹出**：当某桌轮到你操作时，自动切换或弹出通知
- **Mini Table View**：将非活跃桌缩小为卡片视图

### 多桌性能优化
- 非活跃牌桌降低渲染帧率（30fps → 15fps或更低）
- 操作到时提醒的优先级队列管理
- 内存复用：共享牌面资源、动画资源池

来源: [AIST - Multi Table Poker Management Software](https://www.aistechnolabs.com/multi-table-poker-management-software)

## 4. 横屏 vs 竖屏设计

### 横屏（Landscape）
- 传统PC端延续的体验，信息展示更完整
- 适合多桌玩家、长session的认真玩家
- 牌桌、操作区、聊天框可同屏布局

### 竖屏（Portrait）
- 移动优先的新趋势，单手操作友好
- 牌桌压缩为上半屏，操作区在下半屏
- 信息密度降低，需折叠/展开次要信息
- PokerStars和GGPoker均支持双模式

产品决策点：竖屏模式的普及度正在快速增长，新产品应以竖屏为首要设计模式。
