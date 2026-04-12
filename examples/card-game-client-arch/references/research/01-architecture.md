# 维度一：通用架构设计

## 1. MVC/MVVM 在棋牌游戏中的实践

### MVC 模式
- Model: 游戏数据（牌局状态、玩家手牌、筹码、计时器）
- View: UI渲染层（牌桌、手牌、动画、特效）
- Controller: 游戏逻辑（出牌校验、回合管理、结算计算）

Unity中MonoBehaviour天然充当View角色，负责捕获用户输入和更新视觉表现。严格的MVC在游戏中并不完全实用，需要灵活混合数据驱动与组件化架构。

来源: [Unity MVC/MVP](https://unity.com/how-to/build-modular-codebase-mvc-and-mvp-programming-patterns)

### MVVM 模式
MVVM在UI密集型游戏（如棋牌）中特别适合：
- Model: 纯数据层（GameState, PlayerState, CardData）
- ViewModel: 数据转换与业务逻辑（将服务器数据转换为UI可消费的格式）
- View: 纯渲染（只响应ViewModel的变化）

优势：UI和逻辑完全解耦，换皮只需替换View层。

来源: [MVVM in Game Dev](https://dev.to/devsdaddy/organizing-architecture-for-games-on-unity-laying-out-the-important-things-that-matter-4d4p)

### 推荐：MVVM + 事件驱动的混合架构
棋牌游戏最适合的架构是MVVM为主干，事件系统为通信机制：
- ViewModel通过事件通知View更新
- 网络消息通过事件分发到各子系统
- 状态变化通过事件驱动动画播放

## 2. 有限状态机（FSM）设计

### 棋牌游戏核心状态流转
```
[空闲/等待] → [匹配中] → [准备阶段] → [发牌阶段] → [出牌阶段] → [结算阶段] → [空闲/等待]
                                                         ↑              |
                                                         └──────────────┘
```

### 层次状态机（HFSM）
棋牌游戏适合使用层次状态机，将状态分层：
- 顶层：Idle → InGame → Settlement
- InGame内部：Dealing → Playing → Waiting → ShowDown
- Playing内部：MyTurn → OtherTurn → ActionSelection

来源: [Hierarchical FSM in Cocos](https://discuss.cocos2d-x.org/t/tutorial-hierarchical-finite-state-machine/23211)
来源: [Game Programming Patterns - State](https://gameprogrammingpatterns.com/state.html)

### 状态机实现要点
- 每个状态定义 onEnter / onUpdate / onExit
- 转换条件明确，避免隐式跳转
- 支持状态栈（用于弹窗、暂停等临时状态）
- TypeScript中可用枚举+Map实现轻量FSM

## 3. 消息驱动的事件系统

### 事件总线设计
```typescript
// 事件类型枚举
enum GameEvent {
    CARD_DEALT, CARD_PLAYED, TURN_CHANGED,
    GAME_STARTED, GAME_ENDED, PLAYER_JOINED,
    RECONNECTED, ANIMATION_COMPLETE
}

// 订阅/发布模式
EventBus.on(GameEvent.CARD_DEALT, handler)
EventBus.emit(GameEvent.CARD_DEALT, { cards, player })
```

### 事件优先级
- 网络事件 > 游戏逻辑事件 > UI事件 > 动画事件
- 关键事件（断线、结算）不可被取消

## 4. 多玩法共用的抽象层设计

### 核心抽象接口
```typescript
interface IGameRule {
    validatePlay(cards: Card[], context: GameContext): boolean
    getHandRank(cards: Card[]): HandRank
    getNextPlayer(context: GameContext): Player
    checkWinCondition(context: GameContext): WinResult | null
}

interface ITableLayout {
    getSeatPositions(playerCount: number): SeatPosition[]
    getCardDealTarget(seatIndex: number): Vec3
    getPlayZone(): Rect
}

interface ICardRenderer {
    createCard(data: CardData): Node
    playDealAnimation(card: Node, target: Vec3): Promise<void>
    playFlipAnimation(card: Node): Promise<void>
}
```

### 架构分层
1. **基础层**：网络通信、资源管理、事件系统、音效管理
2. **框架层**：状态机、牌桌管理、手牌管理、动画队列
3. **规则层**：具体玩法规则（斗地主、德州扑克、麻将等）
4. **表现层**：UI组件、动画资源、音效资源

换玩法只需替换规则层 + 表现层资源，基础层和框架层完全复用。

来源: [Card Game Architecture](https://bennycheung.github.io/game-architecture-card-ai-1)
来源: [DEV Community - Building Scalable Card Games](https://dev.to/krishanvijay/building-scalable-real-time-multiplayer-card-games-3kn6)
