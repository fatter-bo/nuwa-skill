# 05-二次开发指南

## 一、添加新玩法

### 步骤清单

以在已有框架上添加"斗地主"玩法为例：

#### Step 1：定义规则引擎
```
/games/doudizhu/
├── engine.go          // 实现GameEngine接口
├── deck.go            // 54张牌定义
├── hand_type.go       // 牌型判断（单/对/三带/炸弹/顺子/飞机...）
├── hand_comparator.go // 牌型大小比较
├── state.go           // 游戏状态定义
└── engine_test.go     // 单元测试（覆盖所有牌型和边界）
```

#### Step 2：定义协议消息
```protobuf
message DoudizhuState {
    int32 round = 1;
    string landlord_id = 2;
    repeated Card last_play = 3;
    repeated PlayerInfo players = 4;
}

message PlayCardsRequest {
    repeated Card cards = 1;
}

message PlayCardsResponse {
    bool success = 1;
    string error = 2;
    DoudizhuState new_state = 3;
}
```

#### Step 3：注册Handler
- 在框架的路由层注册新玩法的消息处理器
- Nakama：注册Match Handler
- ioGame：注册Action方法
- Pitaya：注册Handler组件

#### Step 4：配置房间参数
```json
{
    "game_type": "doudizhu",
    "player_count": 3,
    "base_score": 100,
    "timeout_seconds": 30,
    "allow_bot": true,
    "bot_difficulty": "medium"
}
```

#### Step 5：测试验证
- 单元测试：规则引擎100%覆盖
- 集成测试：多客户端模拟完整对局
- 压力测试：1000+并发房间

### 各框架添加新玩法的工作量评估

| 框架 | 预计工作量 | 说明 |
|------|-----------|------|
| Nakama | 3-5天 | Match Handler模式清晰，文档完善 |
| ioGame | 2-4天 | Action注册简单，类MVC模式 |
| Pitaya | 3-5天 | 需理解Handler/Remote概念 |
| jforgame | 3-5天 | 有案例参考 |
| casino-server | 1-3天 | Node.js快速迭代，但代码质量需自行把控 |
| Leaf | 5-7天 | 框架偏底层，需自建更多基础设施 |

## 二、替换通信协议

### 从Socket.io迁移到原生WebSocket

常见场景：casino-server使用Socket.io，但生产环境想用原生WebSocket减少依赖。

**迁移步骤**：
1. 抽象Transport层接口
2. 实现WebSocket Transport
3. 统一消息编解码（建议从JSON切到Protobuf）
4. 处理重连逻辑（Socket.io自带，原生WS需自建）
5. 心跳机制替换

**注意**：
- Socket.io的namespace/room概念需要自行实现
- 断线重连的session恢复是最大工作量
- 建议引入消息序号机制保证有序性

### 从JSON切换到Protobuf

**收益**：
- 包体大小减少50-80%
- 解析速度提升5-10倍
- 强类型，编译期发现错误

**成本**：
- 需要维护.proto文件
- 客户端需要集成Protobuf库
- 调试不如JSON直观（建议开发环境保留JSON选项）

## 三、接入自定义经济系统

### 架构建议

```
+------------------+     +------------------+     +------------------+
|   游戏逻辑服      | --> |   经济中间件      | --> |   支付/银行服务   |
|  (扣除/奖励请求)  |     |  (校验/风控/记账) |     |  (实际资金操作)   |
+------------------+     +------------------+     +------------------+
```

### 接入步骤

1. **定义经济接口**
```go
type EconomyService interface {
    GetBalance(playerID string) (int64, error)
    Deduct(playerID string, amount int64, reason string) error
    Award(playerID string, amount int64, reason string) error
    Transfer(fromID, toID string, amount int64) error
    GetTransactionLog(playerID string, limit int) ([]Transaction, error)
}
```

2. **实现适配器**：对接自有支付系统或第三方支付SDK
3. **集成到游戏流程**：
   - 入座时冻结底注
   - 每手结束时结算
   - 离座时解冻剩余
4. **事务保证**：使用Redis分布式锁或数据库事务，确保扣款和发奖原子性

### 关键风控规则
- 单局最大输赢限额
- 日累计输赢限额
- 异常赢率告警（连续N局全赢）
- 同IP/设备多账号检测

## 四、性能压测

### 压测目标设定

| 指标 | 建议基准 | 说明 |
|------|---------|------|
| CCU | 目标值x1.5 | 同时在线用户数 |
| 消息延迟 | P99 < 200ms | 从发送到收到响应 |
| 吞吐量 | 每秒消息数 | 根据CCU和操作频率计算 |
| 错误率 | < 0.1% | 消息丢失或处理失败 |
| 内存 | 稳定不增长 | 长时间运行无泄漏 |
| CPU | 峰值 < 80% | 留余量应对突发 |

### 压测工具选择

| 工具 | 适用场景 |
|------|---------|
| 自研Bot客户端 | 最精准，模拟真实玩家行为 |
| k6 + WebSocket插件 | 通用WebSocket压测 |
| Locust | Python编写压测脚本 |
| wrk/vegeta | HTTP API压测 |

### 自研Bot压测步骤

1. **编写Bot客户端**：模拟登录、匹配、对局、结算完整流程
2. **参数化配置**：并发数、操作间隔、思考时间
3. **阶梯加压**：100 -> 500 -> 1000 -> 2000 逐步递增
4. **持续运行**：至少30分钟/轮，观察内存泄漏
5. **采集指标**：CPU、内存、网络IO、消息延迟、GC频率

### 常见性能瓶颈

| 瓶颈 | 表现 | 解决方案 |
|------|------|---------|
| GC压力 | Go/Java STW导致卡顿 | 减少小对象分配、对象池 |
| 锁竞争 | CPU使用率不高但吞吐上不去 | 无锁数据结构、分片锁 |
| 序列化 | CPU占比高在编解码 | 换Protobuf、预分配Buffer |
| 数据库 | 查询慢、连接池满 | 读写分离、缓存、批量写入 |
| 网络IO | 带宽打满 | 压缩、精简消息、差量同步 |
| 内存泄漏 | 持续增长 | pprof分析、检查goroutine泄漏 |

### 压测报告模板

```
━━━━━━━━━━━━━━━━━━━━━
[项目名] 压测报告
日期: YYYY-MM-DD
━━━━━━━━━━━━━━━━━━━━━
环境: 4C8G x 3台
CCU目标: 5000
实际峰值CCU: 5200

消息延迟:
  P50: 15ms
  P90: 45ms
  P99: 120ms

吞吐量: 12,000 msg/s
错误率: 0.02%
CPU峰值: 65%
内存峰值: 3.2GB（稳定）

瓶颈: Redis连接数不足
建议: 连接池从50增加到200
━━━━━━━━━━━━━━━━━━━━━
```
