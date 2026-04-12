---
name: card-game-client-arch
description: |
  棋牌游戏客户端架构专家。覆盖从牌桌渲染到手牌管理到出牌动画到结算系统的全流程。
  基于Cocos Creator和Unity输出可复用的棋牌客户端框架，换规则换皮即可出新游戏。
  触发词：「棋牌架构」「棋牌客户端」「牌桌开发」「出牌动画」「棋牌框架」「手牌管理」
---

# 棋牌游戏客户端架构

> 核心目标：构建一个"棋牌客户端脚手架"——基础层和框架层完全复用，换规则层+表现层资源即可出新游戏。

## Skill 规则

- 所有架构建议必须兼顾 Cocos Creator 和 Unity 两套引擎的实现可行性
- 以"可复用"为第一优先级：抽象层接口稳定，具体玩法通过实现接口接入
- 网络通信方案以 WebSocket + Protobuf 为生产标准，同时提供 JSON Mock 方案用于开发期
- 动画系统以 Tween 为核心，避免依赖重度动画编辑器，保持代码可控性
- 性能敏感场景（牌型判断、胡牌检测）优先使用查表法或位运算，避免暴力枚举

---

## 回答工作流

| 触发指令 | 行动 |
|---------|------|
| 「设计棋牌架构」 | 输出四层架构方案（基础层/框架层/规则层/表现层）+ 状态机设计 + 事件系统 |
| 「设计牌桌布局」 | 根据玩法人数输出座位算法（圆桌/半圆/方桌）+ 视角管理 + 适配方案 |
| 「设计手牌系统」 | 输出数据结构 + 排序算法 + 拖拽交互 + 合法性校验 + 牌型识别 |
| 「设计出牌动画」 | 输出动画队列系统 + 发牌/翻牌/出牌/演出动画方案 + Tween封装 |
| 「设计网络通信」 | 输出WebSocket管理 + 断线重连 + 心跳 + 协议格式 + 预测与校验 |
| 「设计组件库」 | 输出扑克/麻将牌组件 + 筹码/计时器/聊天/结算/回放组件规格 |
| 「评估XXX方案」 | 按六大维度审计方案，输出问题清单与改进建议 |

---

## 一、四层架构与状态机

### 1.1 四层分离架构

棋牌客户端的可复用性取决于分层是否干净。推荐四层架构：

```
┌──────────────────────────────────────────┐
│          表现层 (Skin Layer)              │  ← 牌面资源、UI皮肤、音效、特效
│          可替换：换皮换主题               │
├──────────────────────────────────────────┤
│          规则层 (Rule Layer)              │  ← 具体玩法规则（斗地主/德扑/麻将）
│          可替换：换玩法                   │
├──────────────────────────────────────────┤
│          框架层 (Framework Layer)         │  ← 牌桌管理、手牌管理、动画队列、状态机
│          稳定复用                         │
├──────────────────────────────────────────┤
│          基础层 (Foundation Layer)        │  ← 网络通信、事件系统、资源管理、音效管理
│          稳定复用                         │
└──────────────────────────────────────────┘
```

**核心抽象接口**：

```typescript
// 规则层接口——每种玩法实现这组接口即可接入框架
interface IGameRule {
    validatePlay(cards: Card[], context: GameContext): boolean;
    getHandRank(cards: Card[]): HandRank;
    getNextPlayer(context: GameContext): Player;
    checkWinCondition(context: GameContext): WinResult | null;
    sortHand(cards: Card[]): Card[];
}

interface ITableLayout {
    getSeatPositions(playerCount: number): SeatPosition[];
    getCardDealTarget(seatIndex: number): Vec3;
    getPlayZone(): Rect;
    getDiscardZone(seatIndex: number): Rect;
}

interface ICardRenderer {
    createCard(data: CardData): Node;
    playDealAnimation(card: Node, target: Vec3): Promise<void>;
    playFlipAnimation(card: Node): Promise<void>;
    playDiscardAnimation(card: Node, target: Vec3): Promise<void>;
}
```

换玩法 = 实现 `IGameRule` + `ITableLayout` + `ICardRenderer` 三个接口 + 提供资源包。

### 1.2 层次状态机（HFSM）

棋牌游戏的回合制特性天然适合有限状态机。推荐层次状态机以管理嵌套状态：

```
顶层状态机:
  [Idle] → [Matching] → [InGame] → [Settlement] → [Idle]

InGame 内嵌状态机:
  [Dealing] → [Playing] → [ShowDown]
       ↑                      |
       └──────────────────────┘  (下一轮)

Playing 内嵌状态机:
  [WaitingMyTurn] → [ActionSelection] → [WaitingResponse] → [WaitingOtherTurn]
```

每个状态实现三个钩子：`onEnter()`、`onUpdate(dt)`、`onExit()`。支持状态栈处理临时中断（如弹窗、暂停、断线重连遮罩）。

### 1.3 事件驱动通信

模块间通信统一走事件总线，禁止直接引用：

```typescript
// 事件分级
enum EventPriority {
    NETWORK = 0,    // 网络事件最高优先
    GAME_LOGIC = 1, // 游戏逻辑次之
    UI = 2,         // UI更新
    ANIMATION = 3   // 动画最低
}

// 典型事件流：服务器广播"玩家B出牌"
// Network层 → emit(CARD_PLAYED) → GameLogic更新状态
//                                → UI更新手牌显示
//                                → Animation播放出牌动画
//                                → Sound播放音效
```

关键原则：网络消息到UI更新的链路中，每个环节只做自己的事，通过事件串联。

---

## 二、牌桌渲染系统

### 2.1 座位布局算法

三种标准布局覆盖绝大多数棋牌玩法：

**圆桌布局**（适用于3-10人，如德州扑克）：

```typescript
function circularSeats(count: number, radius: number, center: Vec2): Vec2[] {
    const seats: Vec2[] = [];
    const startAngle = -Math.PI / 2; // 自己在底部
    for (let i = 0; i < count; i++) {
        const angle = startAngle + (2 * Math.PI * i) / count;
        seats.push(new Vec2(
            center.x + radius * Math.cos(angle),
            center.y + radius * Math.sin(angle)
        ));
    }
    return seats;
}
```

**半圆布局**（适用于3人场，如斗地主）：自己固定底部中央，其余玩家均匀分布在上半弧。

**方桌布局**（适用于4人麻将）：下/右/上/左四个固定位置，分别对应自己/下家/对家/上家。

### 2.2 视角映射

服务器用绝对座位号，客户端用相对座位号（自己永远是0号）：

```typescript
function toLocalSeat(serverSeat: number, mySeat: number, total: number): number {
    return (serverSeat - mySeat + total) % total;
}
```

- 本地玩家始终在屏幕底部
- 手牌正面只展示本地玩家，其他人显示牌背
- 麻将中对家牌180度旋转，左右家90度/270度旋转

### 2.3 响应式适配策略

- 基准设计分辨率 1920x1080（横屏）
- Canvas适配模式选 `SHOW_ALL` 或 `FIXED_WIDTH`
- 座位半径随屏幕缩放比动态调整
- 安全区处理：手牌区域避开刘海/底部横条
- 强制横屏锁定

---

## 三、手牌管理系统

### 3.1 数据结构

**扑克牌**：ID编码 `id = suit * 13 + (rank - 2)`，0-51为标准52张，52/53为大小王。位掩码表示法用于快速牌型判断——每张牌对应一个素数，手牌乘积作为哈希键。

**麻将牌**：1x34数组编码，`handCount[i]` 表示第i种牌的持有数量。索引0-8万子、9-17条子、18-26筒子、27-30风牌、31-33箭牌。每种牌最多4张实例。

### 3.2 排序规则

排序规则因玩法而异，抽象为 `IGameRule.sortHand()` 接口：
- 斗地主：大王 > 小王 > 2 > A > K > ... > 3
- 德州扑克：按点数降序，同点数按花色
- 麻将：万子 → 条子 → 筒子 → 风牌 → 箭牌，各类内部按数值升序

### 3.3 手牌交互

**扇形展开**：牌数多时自动压缩间距，保持可触摸区域不小于44px。通过抛物线函数为手牌添加微弧度，视觉更自然。

**选牌方式**：
- 扑克（斗地主）：滑动多选，选中牌上移20px，支持取消和反选
- 麻将：点选+上滑出牌，上滑距离超过阈值触发打牌

### 3.4 牌型识别与合法性校验

**客户端预校验流程**：
1. 用户选牌 → 识别牌型（统计点数频率，匹配已知牌型模式）
2. 与上家出牌比较（同类型、同长度、更大点数）
3. 校验通过 → 乐观发送请求 + 播放动画
4. 校验失败 → 高亮不合法的牌 + 提示原因

**高性能评估算法**：
- 扑克（德州）：Cactus Kev查表法，7462种不同手牌等级，素数乘积+完美哈希，单次评估微秒级
- 麻将胡牌检测：花色切分+递归回溯，查表优化后百万次/秒；支持癞子的拆解算法优于查表法
- 特殊牌型（七对、十三幺）独立判断，不走通用路径

---

## 四、动画与演出系统

### 4.1 动画队列

所有游戏动画进入统一队列，顺序执行，避免动画冲突：

```typescript
class AnimationDirector {
    private queue: AnimTask[] = [];

    enqueue(task: AnimTask) {
        this.queue.push(task);
        if (!this.isPlaying) this.playNext();
    }

    // 支持跳过（玩家点击屏幕快进）
    skip() { this.currentTask?.fastForward(); }
}
```

可并行的动画（如多张牌同时翻开）在单个 `AnimTask` 内用 `Promise.all` 管理。

### 4.2 标准动画库

| 动画类型 | 时长 | 缓动函数 | 要点 |
|---------|------|----------|------|
| 发牌 | 0.3s/张，间隔0.08s | cubicOut | 从牌堆飞向各座位，自己的牌到达后翻面 |
| 出牌 | 0.2-0.3s | backOut | 先微放大强调，再飞向出牌区缩小落定 |
| 翻牌 | 0.4s | sineIn/sineOut | X轴缩放到0→切换纹理→恢复，或3D场景下Y轴旋转90度 |
| 筹码入池 | 0.4s/组，间隔0.05s | quadIn | 多个筹码图标依次飞向底池，到达后销毁 |
| 胡牌演出 | 1.5-5s（按等级） | 多段组合 | 普通=文字提示，特殊=横幅+粒子，史诗=震屏+Spine |
| 数字变化 | 0.3-0.5s | linear | 分数/筹码数滚动动画，通过tween驱动中间值 |

### 4.3 Tween工具封装

```typescript
// Promise化封装
function tweenAsync<T>(target: T, tw: Tween<T>): Promise<void> {
    return new Promise(resolve => tw.call(() => resolve()).start());
}

// 常用预设
const Anim = {
    popIn:  (node, d = 0.3) => tween(node).set({ scale: Vec3.ZERO })
                .to(d, { scale: Vec3.ONE }, { easing: 'backOut' }),
    flyTo:  (node, pos, d = 0.4) => tween(node)
                .to(d, { position: pos }, { easing: 'cubicOut' }),
    shake:  (node, intensity = 5, times = 5) => { /* 左右交替位移 */ },
    fadeOut: (node, d = 0.3) => tween(node).to(d, { opacity: 0 }),
};
```

### 4.4 演出等级

按牌型稀有度分四级，配置化管理：
- **普通**（1.5s）：文字提示，无特效
- **特殊**（2.5s）：全屏横幅 + 粒子喷射
- **史诗**（3.5s）：横幅 + 粒子 + 屏幕震动
- **传奇**（5s）：专属Spine动画 + 全屏特效 + 延迟揭示

玩家可点击屏幕跳过当前演出。

---

## 五、网络通信层

### 5.1 WebSocket管理

```
生命周期: [未连接] → connect → [连接中] → onopen → [已连接] → onclose → [断开]
                                                      ↓
                                            heartbeat超时 → reconnect
```

核心要点：
- 使用二进制帧（`binaryType = 'arraybuffer'`）配合Protobuf
- 消息包含序列号和确认号，用于重连后断点续传
- 断线期间缓存待发消息，重连后批量发送

### 5.2 断线重连

**指数退避 + 抖动**：起始500ms，倍增，上限30s，叠加30%随机抖动。最多尝试12次（约覆盖2分钟）。

**重连后状态恢复**：
1. 客户端发送重连请求（携带sessionId + 最后确认的序列号）
2. 服务器回复全量状态快照 + 断线期间的增量事件
3. 客户端重放增量事件，恢复到当前状态
4. 移除重连遮罩，恢复交互

**UI提示**：断线时覆盖半透明遮罩 + "正在重连(第N次)..."；成功后Toast"网络已恢复"；全部失败后弹窗"[重试] [返回大厅]"。

### 5.3 心跳检测

间隔15秒发送心跳包，5秒无回复视为超时。心跳响应中携带服务器时间戳，用于计算RTT和校正本地时钟。

### 5.4 协议格式

| 阶段 | 格式 | 理由 |
|------|------|------|
| 开发期 | JSON | 可读性好，方便抓包调试 |
| 生产环境 | Protobuf | 包体小（约JSON的1/3-1/5），解析快 |
| 极致性能 | FlatBuffers | 零拷贝反序列化，适合高频小消息 |

封装统一的序列化接口（`encode/decode`），通过配置一键切换格式。

### 5.5 客户端预测

棋牌游戏延迟容忍度较高（不像FPS），预测策略相对保守：
- **乐观出牌**：客户端预校验通过后立即播放动画，等服务器确认
- **服务器确认**：预测正确则无额外操作
- **服务器拒绝**：回滚到出牌前快照，撤销动画，提示原因
- **状态哈希校验**：关键操作后双端计算状态哈希，不一致时请求全量同步

### 5.6 Mock Server

开发期提供本地Mock Server：
- 模拟可配置的网络延迟（50-200ms）
- 实现基本游戏规则，支持单人调试全流程
- 通过统一Socket接口适配，配置一行切换真实/Mock服务器
- 支持导出/导入录像数据用于回放测试

---

## 六、可复用组件库

### 6.1 牌面组件

**扑克牌组件**（52+2张）：
- 合图资源管理（单张SpriteAtlas），命名规范 `{suit}_{rank}.png`
- 正面/牌背切换，支持多款牌背皮肤
- 选中状态（上移）、禁用状态（灰色）、高亮状态（发光边框）
- 翻转动画内置

**麻将牌组件**（136/144张）：
- 三套视角资源：立牌（手牌）、横牌（副露）、平放（弃牌）
- 状态切换：手牌/弃牌/副露/暗杠/刚摸入
- 可打牌高亮提示

### 6.2 通用UI组件

| 组件 | 功能 | 要点 |
|------|------|------|
| **筹码显示** | 数量展示+堆叠视觉+飞入动画 | 金额自动格式化（1.2K/3.5M），贪心算法分解面值 |
| **回合计时器** | 环形进度条+倒计时数字 | 紧急状态（<5秒）红色闪烁+音效，支持服务器时间校正 |
| **快捷聊天** | 预定义短语+表情面板+气泡显示 | 预定义短语避免内容审核问题，气泡自动消失 |
| **结算面板** | 手牌展示+胜负高亮+分数动画 | 翻牌动画→牌型标注→分数滚动→操作按钮 |
| **战绩回放** | 事件驱动回放+变速+跳转 | 基于事件流重演，支持1x/2x/4x倍速和任意跳转 |

### 6.3 组件设计原则

1. **数据驱动**：组件只接收数据和配置，不依赖具体玩法逻辑
2. **事件通信**：组件通过事件对外通信，不直接调用外部方法
3. **样式可配**：颜色/字号/间距通过配置注入，不硬编码
4. **动画可选**：所有动画提供 `animate: boolean` 开关，回放/快进场景可关闭

---

## 核心方法论框架

### 框架一：四层分离架构（Foundation-Framework-Rule-Skin）

基础层和框架层为稳定内核，规则层和表现层为可替换外壳。新增一种棋牌玩法的工作量 = 实现三个规则接口 + 制作一套美术资源。框架层的牌桌管理、手牌管理、动画队列、网络通信对所有玩法透明复用。

### 框架二：层次状态机驱动的回合管理

用三级嵌套状态机管理游戏生命周期。顶层管理大厅/游戏/结算，中层管理发牌/出牌/摊牌，底层管理当前回合内的细粒度状态。状态转换由服务器消息驱动，客户端状态机被动响应。

### 框架三：乐观预测-服务器权威的混合模型

客户端做预校验和乐观渲染以获得即时反馈，服务器保持最终权威。偏差通过状态哈希快速检测，不一致时全量同步。棋牌游戏延迟容忍度高，此模型足以覆盖所有场景。

### 框架四：事件总线解耦的模块通信

所有子系统（网络/逻辑/UI/动画/音效）通过事件总线通信，无直接引用。事件分四级优先级（网络>逻辑>UI>动画），保证关键消息不被阻塞。

### 框架五：查表法优先的性能策略

牌型识别和胡牌检测是高频操作，优先使用预计算查表法。扑克用素数乘积+完美哈希（7462种等级），麻将用花色切分+查表（百万次/秒）。只在查表无法覆盖的场景（如带癞子的复杂规则）才回退到递归回溯。

### 框架六：动画队列与演出分级

所有游戏动画进入统一队列顺序执行，避免视觉冲突。演出效果按牌型稀有度分四级配置，从纯文字到全屏CG。玩家可随时点击跳过，不阻塞游戏流程。

### 框架七：协议格式与环境适配

开发期用JSON（可读调试），生产用Protobuf（高效传输），通过统一序列化接口一行配置切换。Mock Server与真实服务器共享同一个Socket适配层接口。

---

## 决策启发式

1. **"三个接口"原则**：评估架构可复用性时，检查新增一种玩法是否只需实现 `IGameRule` + `ITableLayout` + `ICardRenderer` 三个接口。如果需要修改框架层代码，说明抽象不够。

2. **"状态机先行"原则**：开始编码前先画完整的状态流转图。如果一个状态有超过5个出口转换，说明需要引入子状态机拆分。

3. **"服务器说了算"原则**：所有影响游戏结果的判定（洗牌、发牌、胜负）必须在服务器执行。客户端的校验只是体验优化，不可作为最终依据。

4. **"动画不阻塞逻辑"原则**：游戏逻辑状态更新不等待动画播完。动画是"追赶"逻辑状态的视觉表现。断线重连后可以跳过所有未播动画直接渲染最终状态。

5. **"80ms发牌节奏"原则**：发牌动画单张飞行时间0.3秒，牌间间隔80ms。过快（<50ms）视觉混乱，过慢（>150ms）用户失去耐心。此节奏经大量棋牌产品验证。

6. **"座位号永远转换"原则**：服务器传输绝对座位号，客户端显示相对座位号（自己=0）。所有座位相关逻辑必须经过 `toLocalSeat()` 转换，否则视角会错乱。

7. **"断线30秒内无感"原则**：30秒内的断线重连应做到用户几乎无感知——遮罩延迟1秒才出现，重连后补播关键动画（而非全部），计时器从服务器时间校正后继续。

8. **"查表兜底递归"原则**：牌型判断先查表，查表未命中再回退到递归回溯。常见牌型（>95%的情况）由查表覆盖，递归只处理罕见边界情况。

9. **"预校验≠真校验"原则**：客户端预校验用于即时反馈（灰掉不可出的牌、高亮可出的牌），但发送到服务器的出牌请求不附带客户端判断结果，由服务器独立重算。

10. **"演出可跳过"原则**：所有演出动画（胡牌特效、大牌型展示）必须支持玩家点击跳过。演出是加分项不是必要流程，阻塞出牌会引发强烈负面体验。

---

## 参考资源

- [Unity MVC/MVP 模式指南](https://unity.com/how-to/build-modular-codebase-mvc-and-mvp-programming-patterns)
- [Game Programming Patterns - State](https://gameprogrammingpatterns.com/state.html)
- [Cocos Creator Tween 系统文档](https://docs.cocos.com/creator/3.8/manual/en/tween/)
- [CardHouse - Unity 开源卡牌框架](https://github.com/pipeworks-studios/CardHouse)
- [Cactus Kev 扑克手牌评估器](http://suffe.cool/poker/evaluator.html)
- [qipai - 棋牌胡牌算法（多语言实现）](https://github.com/hufans/qipai)
- [Client-Side Prediction - Gabriel Gambetta](https://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html)
- [Real-Time Card Games Architecture](https://developersvoice.com/blog/practical-design/realtime-card-games-net-architecture-guide/)
- [Building Scalable Real-Time Multiplayer Card Games](https://dev.to/krishanvijay/building-scalable-real-time-multiplayer-card-games-3kn6)
- [WebSocket Reconnection Guide](https://websocket.org/guides/reconnection/)
- [Protobuf vs FlatBuffers 对比](https://www.oreateai.com/blog/protobuf-vs-flatbuffers-choosing-the-right-serialization-framework-for-realtime-applications/f410bfacd86a9235f73cf2747c5026aa)
- [Card Game Architecture Model](https://bennycheung.github.io/game-architecture-card-ai-1)

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
