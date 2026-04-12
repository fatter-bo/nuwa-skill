# 02-服务端架构对比

## 核心架构模块

棋牌服务器通常包含以下核心模块：

### 1. 房间管理（Room Management）

| 框架 | 实现方式 | 特点 |
|------|---------|------|
| Nakama | Match API（权威匹配+Relayed匹配） | 服务端权威模式下所有数据由服务器验证和广播 |
| Pitaya | 自定义Handler+Group广播 | 需开发者自行实现房间抽象 |
| ioGame | 内置房间概念+动态绑定 | 玩家绑定到逻辑服后请求自动路由 |
| casino-server | Redis Key存储房间状态 | 简单直接，房间信息存Redis |
| Leaf | Module内自行管理 | 无内置房间，需自建 |

**最佳实践**：
- 房间生命周期：创建 -> 等待 -> 游戏中 -> 结算 -> 销毁
- 每个房间独立goroutine/线程，避免跨房间锁竞争
- 房间状态持久化到Redis/DB，支持服务器重启恢复

### 2. 匹配系统（Matchmaking）

| 框架 | 实现方式 |
|------|---------|
| Nakama | 内置Matchmaker，支持属性匹配+自定义算法 |
| Pitaya | 无内置，需自行实现 |
| ioGame | 无内置，需自行实现 |
| casino-server | 简单的房间列表选择 |

**棋牌匹配关键要素**：
- 按金币/积分等级匹配
- 快速匹配 vs 自建房间
- 机器人填充（人数不足时加Bot）
- 匹配超时处理

### 3. 消息通信

| 框架 | 协议 | 序列化 |
|------|------|--------|
| Nakama | WebSocket + gRPC | Protobuf + JSON |
| Pitaya | TCP/WebSocket | Protobuf |
| ioGame | TCP/WebSocket/UDP | Protobuf/JSON |
| Nano | TCP/WebSocket | Protobuf |
| casino-server | Socket.io（WebSocket） | JSON |
| Leaf | TCP/WebSocket | 自定义 |

**通信架构选型建议**：
- **WebSocket**是棋牌首选：双向实时、浏览器原生支持、穿透性好
- **Protobuf**优于JSON：包体小50-80%，解析快5-10倍
- 心跳检测必须有：建议5-15秒间隔
- 消息编号+确认机制：防止丢包导致状态不同步

### 4. 数据持久化

| 框架 | 存储方案 |
|------|---------|
| Nakama | CockroachDB/PostgreSQL |
| ioGame | 无依赖（内存+自定义） |
| jforgame | 自定义ORM（多数据源+自动建表） |
| casino-server | Redis |
| Pitaya | 自定义（推荐Redis+MySQL） |

**棋牌数据持久化策略**：
- **热数据**（在线状态、房间信息）-> Redis
- **温数据**（玩家资产、游戏记录）-> MySQL/PostgreSQL
- **冷数据**（历史牌局回放）-> MongoDB/对象存储
- 写入策略：先写缓存，定时/定量批量落库

### 5. 分布式部署

| 框架 | 方案 | 服务发现 | RPC |
|------|------|---------|-----|
| Nakama | 原生集群 | 内置 | gRPC |
| Pitaya | etcd+NATS | etcd | NATS/gRPC |
| ioGame | Broker网关集群 | 内置 | 内置 |
| Nano | 内置分布式 | 内置 | 自定义 |
| Leaf | 单进程 | 无 | 无 |
| casino-server | PM2+Redis | 无 | Redis Pub/Sub |

**分布式架构典型拓扑**：
```
客户端 -> 网关集群(Gateway) -> 大厅服(Lobby) -> 游戏逻辑服(Game Logic)
                            -> 匹配服(Match)
                            -> 聊天服(Chat)
                            -> 排行榜服(Rank)
```

### 6. 架构模式对比

#### A. 单体架构（Leaf、casino-server）
- 优点：简单、延迟低、调试方便
- 缺点：无法水平扩展、单点故障
- 适用：小规模（<1000 CCU）

#### B. 微服务架构（Pitaya、ioGame）
- 优点：独立扩展、故障隔离、灵活部署
- 缺点：网络开销、运维复杂度高
- 适用：中大规模（1000-100000 CCU）

#### C. 混合架构（Nakama）
- 优点：单机开发、集群部署，代码不变
- 缺点：框架耦合度高
- 适用：全规模

## 关键架构决策清单

1. **通信协议选择**：WebSocket（推荐）vs TCP vs Socket.io
2. **序列化格式**：Protobuf（推荐）vs JSON vs 自定义二进制
3. **状态管理**：服务端权威（推荐）vs 客户端预测+服务端校验
4. **部署模式**：单体起步 -> 拆分微服务（渐进式）
5. **数据库选型**：Redis（热）+ MySQL/PG（温）+ 对象存储（冷）
