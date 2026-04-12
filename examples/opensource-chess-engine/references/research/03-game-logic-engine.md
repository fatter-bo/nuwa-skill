# 03-游戏逻辑引擎

## 规则引擎抽象设计

### 核心原则

规则引擎（Rules Engine）应当是独立于UI和网络层的自包含代码模块：
- 控制所有操作的合法性验证
- 管理游戏数据和状态
- 可独立于任何UI和OS环境运行和测试
- UI不负责规则判断，UI需要向规则引擎发起询问

来源：https://doc.asmodee.net/rules-engine

### 分层架构

```
+----------------------------+
|      表现层 (Visual)       |  -- 动画、音效、UI
+----------------------------+
|   视觉游戏逻辑 (Visual GL) |  -- 状态机、行为树
+----------------------------+
|   纯游戏逻辑 (Pure GL)     |  -- 规则校验、状态计算
+----------------------------+
|      数据层 (Repository)   |  -- 持久化、回放
+----------------------------+
```

来源：https://bennycheung.github.io/game-architecture-card-ai-1

### 棋牌规则引擎接口设计

```go
// 通用棋牌规则引擎接口
type GameEngine interface {
    // 初始化一局游戏
    Init(config GameConfig) *GameState
    // 获取当前可执行的合法操作
    GetLegalActions(state *GameState, playerID string) []Action
    // 执行一个操作，返回新状态
    ApplyAction(state *GameState, action Action) (*GameState, error)
    // 判断游戏是否结束
    IsTerminal(state *GameState) bool
    // 获取结算结果
    GetResult(state *GameState) *GameResult
}
```

关键设计点：
- **不可变状态**：每次ApplyAction返回新State而非修改原State，便于回放和并发
- **操作合法性**：GetLegalActions先过滤，ApplyAction再二次校验
- **可序列化**：GameState必须可序列化为JSON/Protobuf，用于网络传输和持久化

## 状态同步

### 方案对比

| 方案 | 原理 | 适用场景 |
|------|------|---------|
| 帧同步（Lockstep） | 同步操作指令，各端独立计算 | RTS、MOBA |
| 状态同步（State Sync） | 服务端计算，广播完整状态 | 棋牌（推荐） |
| 预测回滚（Rollback） | 客户端预测+服务端校验+回滚 | 格斗、FPS |

**棋牌推荐：状态同步**
- 棋牌是回合制，延迟要求不高（<500ms可接受）
- 服务端权威计算，杜绝作弊
- 每次操作后广播完整桌面状态（手牌对其他人隐藏）
- 消息示例：
  ```json
  {
    "type": "state_update",
    "round": 3,
    "currentPlayer": "player2",
    "publicCards": ["♠A", "♥K", "♦10"],
    "myHand": ["♠J", "♥Q"],
    "pot": 1500,
    "players": [
      {"id": "player1", "chips": 800, "bet": 200, "status": "folded"},
      {"id": "player2", "chips": 1200, "bet": 300, "status": "active"},
      {"id": "player3", "chips": 500, "bet": 0, "status": "waiting"}
    ]
  }
  ```

### 信息隐藏

棋牌的核心难点是**不完全信息**：
- 每个玩家只能看到自己的手牌
- 服务端维护完整状态（上帝视角）
- 向每个玩家发送**定制化视图**（Personalized View）
- 绝对不能将其他玩家手牌发送给客户端——即使"隐藏"也不行（可被抓包）

## 状态机设计

### 棋牌游戏状态机（以德州扑克为例）

```
[等待入座] -> [准备阶段] -> [发手牌] -> [翻牌前下注]
    -> [发翻牌] -> [翻牌后下注]
    -> [发转牌] -> [转牌后下注]
    -> [发河牌] -> [河牌后下注]
    -> [摊牌/结算] -> [等待下一局]
```

状态机（State Machine）是管理回合制游戏不同阶段的骨干模式：
- 有限的状态集合
- 同一时刻只能处于一个状态
- 每个状态定义允许的操作和转换条件

来源：https://gameprogrammingpatterns.com/state.html

### 命令模式（Command Pattern）

每个游戏操作封装为Command对象：
- AI和玩家使用完全相同的Action接口
- 支持操作历史记录和回放
- 支持撤销（如果规则允许）

```go
type Command struct {
    PlayerID  string
    Type      string  // "bet", "fold", "call", "raise", "check"
    Amount    int     // 仅bet/raise时有值
    Timestamp int64
}
```

## 操作合法性校验

### 双重校验策略

1. **客户端预校验**：即时反馈，减少无效请求
2. **服务端最终校验**：权威判定，防作弊

### 常见校验项

| 校验项 | 说明 |
|--------|------|
| 轮次校验 | 是否轮到该玩家操作 |
| 操作类型 | 当前阶段是否允许该操作 |
| 金额校验 | 下注是否在合法范围内（最小加注/全押） |
| 时间校验 | 是否在操作时限内 |
| 状态校验 | 玩家是否已弃牌/离线 |
| 序列号校验 | 防止重复提交（幂等性） |

## AI Bot行为树

### 棋牌Bot设计层级

```
根节点（Selector）
├── 检查是否必须操作（超时自动弃牌）
├── 强牌策略（Sequence）
│   ├── 评估手牌强度
│   └── 加注/全押
├── 中等牌策略（Sequence）
│   ├── 评估期望值
│   └── 跟注/小加注
└── 弱牌策略（Sequence）
    ├── 计算弃牌代价
    └── 弃牌/诈唬（概率触发）
```

### Bot设计要点

1. **分级难度**：初级Bot随机决策，高级Bot计算期望值
2. **模拟思考时间**：不能瞬间响应（暴露是机器人），加1-5秒随机延迟
3. **行为多样性**：偶尔做非最优决策（诈唬/慢打）增加真实感
4. **不能作弊**：Bot只能使用自己的手牌信息，不能读取对手手牌
5. **经济平衡**：Bot赢率不能过高或过低，需要调整参数控制
