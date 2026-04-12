# 维度二：多玩法热插拔机制

## 1. 玩法模块标准化架构

### 插件式游戏架构（Plugin-based Game Architecture）

开元模式的核心竞争力在于：一个平台框架支持几十种棋牌玩法的动态加载与独立更新。这要求每个玩法是一个独立可部署的"游戏插件"。

```
┌──────────────────────────────────────────────────┐
│               Platform Core (平台核心)            │
│  ┌────────┐ ┌──────────┐ ┌─────────┐ ┌────────┐ │
│  │房间管理 │ │匹配引擎  │ │钱包系统 │ │消息总线 │ │
│  └────┬───┘ └────┬─────┘ └────┬────┘ └───┬────┘ │
│       │          │            │           │      │
│  ═════╪══════════╪════════════╪═══════════╪═══╗  │
│       │    Game Plugin Interface (标准接口)│   ║  │
│  ═════╪══════════╪════════════╪═══════════╪═══╝  │
├───────┼──────────┼────────────┼───────────┼──────┤
│  ┌────▼───┐ ┌────▼───┐ ┌─────▼──┐ ┌─────▼───┐  │
│  │斗地主  │ │德州扑克│ │麻 将   │ │炸金花   │  │
│  │Plugin  │ │Plugin  │ │Plugin  │ │Plugin   │  │
│  └────────┘ └────────┘ └────────┘ └─────────┘  │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌─────────┐  │
│  │二十一点│ │百家乐  │ │牛牛    │ │  ...    │  │
│  │Plugin  │ │Plugin  │ │Plugin  │ │ Plugin  │  │
│  └────────┘ └────────┘ └────────┘ └─────────┘  │
└──────────────────────────────────────────────────┘
```

### 标准化游戏插件接口

每个玩法模块必须实现统一的接口契约：

```typescript
interface IGamePlugin {
  // 元数据
  readonly gameId: string;
  readonly gameName: string;
  readonly version: string;
  readonly minPlayers: number;
  readonly maxPlayers: number;

  // 生命周期
  onLoad(): Promise<void>;           // 插件加载时初始化
  onUnload(): Promise<void>;         // 插件卸载时清理资源
  
  // 房间管理
  createRoom(config: RoomConfig): GameRoom;
  destroyRoom(roomId: string): void;
  
  // 游戏流程
  onPlayerJoin(room: GameRoom, player: Player): JoinResult;
  onPlayerLeave(room: GameRoom, player: Player): void;
  onPlayerAction(room: GameRoom, player: Player, action: GameAction): ActionResult;
  onGameStart(room: GameRoom): void;
  onGameEnd(room: GameRoom): SettlementResult;
  
  // 状态查询
  getGameState(room: GameRoom, viewer: Player): GameStateView;
  getReplayData(room: GameRoom): ReplayData;
}

interface RoomConfig {
  blindLevel: number;         // 底注级别
  maxRounds?: number;         // 最大局数
  timeBank: number;           // 思考时间(秒)
  customRules: Record<string, any>;  // 玩法特定配置
}
```

来源: [Game Programming Patterns - Plugin Architecture](https://gameprogrammingpatterns.com/state.html)
来源: [Microservices Architecture for Gaming Industry](https://fenix.tecnico.ulisboa.pt/downloadFile/1126295043838302/Thesis-SofiaEstrela-84186-22012021.pdf)

## 2. 动态加载与独立更新

### 微服务级隔离

每个玩法运行为独立的微服务容器：

```yaml
# docker-compose片段
services:
  game-doudizhu:
    image: games/doudizhu:v2.3.1
    replicas: 5
    environment:
      GAME_ID: "doudizhu"
      NATS_URL: "nats://message-bus:4222"
    
  game-texas-holdem:
    image: games/texas-holdem:v3.1.0
    replicas: 10
    environment:
      GAME_ID: "texas_holdem"
      NATS_URL: "nats://message-bus:4222"
```

### 热更新流程

```
1. 新版本镜像构建 → 推送到镜像仓库
2. K8s Rolling Update → 逐个替换Pod
3. 旧Pod等待当前牌局结束（Graceful Shutdown）
4. 新Pod接收新牌局请求
5. 全部切换完成 → 旧版本下线

关键：进行中的牌局不受影响，新牌局使用新版本
```

### 玩法注册与发现

```typescript
// 游戏注册中心
interface GameRegistry {
  register(plugin: GamePluginMeta): void;
  unregister(gameId: string): void;
  discover(filter?: GameFilter): GamePluginMeta[];
  getHealth(gameId: string): HealthStatus;
  getCapacity(gameId: string): CapacityInfo;
}

// 插件元数据（注册到服务发现）
interface GamePluginMeta {
  gameId: string;
  gameName: Record<string, string>;  // 多语言名称
  category: "poker" | "mahjong" | "board" | "casual";
  version: string;
  endpoints: {
    rest: string;       // REST API地址
    websocket: string;  // WebSocket地址
    grpc: string;       // gRPC地址
  };
  status: "active" | "maintenance" | "deprecated";
  supportedRegions: string[];
}
```

来源: [mmo-arch - Microservice Game Architecture](https://github.com/game-arch/mmo-arch)
来源: [jforgame - 一站式游戏服务器框架](https://github.com/kingston-csj/jforgame)

## 3. 玩法配置化（规则参数化）

### 配置驱动而非硬编码

将玩法规则中的变量抽象为配置参数，同一套斗地主代码通过不同配置产出"经典斗地主"、"癞子斗地主"、"四人斗地主"等变体：

```json
{
  "gameId": "doudizhu",
  "variant": "classic",
  "rules": {
    "playerCount": 3,
    "deckType": "54cards",
    "landlordCards": 3,
    "bombMultiplier": 2,
    "rocketMultiplier": 4,
    "springMultiplier": 2,
    "maxCallScore": 3,
    "allowRobLandlord": true,
    "enableLaizi": false,
    "laiziCards": []
  },
  "timing": {
    "callTimeout": 15,
    "playTimeout": 30,
    "autoPlayOnTimeout": true
  },
  "settlement": {
    "baseScore": 100,
    "feeRate": 0.05,
    "minBuyIn": 1000,
    "maxBuyIn": 50000
  }
}
```

### 癞子变体配置

```json
{
  "gameId": "doudizhu",
  "variant": "laizi",
  "rules": {
    "playerCount": 3,
    "deckType": "54cards",
    "landlordCards": 3,
    "bombMultiplier": 2,
    "enableLaizi": true,
    "laiziCount": 2,
    "laiziSelection": "random_after_deal",
    "laiziCanFormRocket": false,
    "pureLaiziMultiplier": 8
  }
}
```

### 配置热更新机制

- 配置存储在配置中心（如Consul/Nacos/etcd）
- 修改配置后自动推送到运行中的Game Service
- 新牌局使用新配置，进行中的牌局保持原配置
- 支持A/B测试：对不同用户群使用不同规则配置

来源: [qp - 商用棋牌游戏平台](https://github.com/openinggame/qp)
来源: [PokerKit - Flexible Poker Variants](https://github.com/uoftcprg/pokerkit)

## 4. 游戏生命周期管理

### 统一生命周期状态机

```
[已注册] → [加载中] → [就绪] → [运行中] → [维护中] → [已弃用] → [已卸载]
                        ↑                      |
                        └──────────────────────┘
```

### 灰度发布策略

- **按区域灰度**：新版本先在低流量区域部署
- **按运营商灰度**：指定运营商优先试用新版本
- **按比例灰度**：10%→30%→50%→100%逐步放量
- **快速回滚**：如发现问题，秒级切回旧版本

来源: [Monoliths vs Microservices in Gaming Architecture](https://ascendion.com/insights/monoliths-vs-microservices-in-gaming-architecture-striking-the-right-balance/)
