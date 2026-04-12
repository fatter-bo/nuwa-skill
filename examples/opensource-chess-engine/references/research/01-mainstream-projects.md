# 01-主流开源棋牌项目梳理

## 按语言分类

### Go语言

#### 1. Nakama（heroiclabs/nakama）
- **Stars**: 12.5k
- **License**: Apache-2.0
- **定位**: 通用游戏后端服务器，内置多人对战、匹配、排行榜、聊天、社交
- **最新版本**: v3.38.0（2026.03）
- **社区活跃度**: 极高，持续维护，Heroic Labs商业支持
- **客户端SDK**: .NET/C#、Unity、JavaScript、Java/Android、Unreal、Godot、Defold、Swift/iOS
- **数据库**: CockroachDB或Postgres兼容
- **扩展**: 支持Lua、TypeScript/JavaScript、Go运行时
- **适用场景**: 通用游戏后端，棋牌需自行实现游戏逻辑层
- **来源**: https://github.com/heroiclabs/nakama

#### 2. Leaf（name5566/leaf）
- **Stars**: 5.5k
- **License**: Apache-2.0
- **定位**: 轻量级Go游戏服务器框架
- **核心特性**: 极易上手、可靠性高、多核支持、模块化设计
- **架构**: Module-Based，每个Module运行在独立goroutine
- **消息路由**: Gate模块统一接入，路由到不同业务Module
- **注意**: 最后更新2016年，社区不活跃，适合学习参考
- **来源**: https://github.com/name5566/leaf

#### 3. Pitaya（topfreegames/pitaya）
- **Stars**: 2.8k
- **License**: MIT
- **定位**: 可伸缩的分布式游戏服务器框架（Wildlife Studios出品）
- **最新版本**: v2.11.21（2026.01）
- **服务发现**: etcd
- **RPC**: NATS + gRPC
- **可观测性**: OpenTracing（Jaeger）、Prometheus、Statsd
- **部署**: Docker原生支持
- **来源**: https://github.com/topfreegames/pitaya

#### 4. Nano（lonng/nano）
- **Stars**: 3.2k
- **License**: MIT
- **定位**: 轻量级高性能Go游戏服务器网络库
- **核心特性**: 内置分布式方案、组件化架构、WebSocket+TCP
- **消息类型**: Request/Response/Notify/Push四种
- **序列化**: Protocol Buffers
- **来源**: https://github.com/lonng/nano

### Java

#### 5. ioGame（iohao/iogame）
- **Stars**: 1.2k
- **License**: AGPL-3.0
- **定位**: 无锁异步事件驱动的Java Netty网络框架
- **性能**: 单线程1152万次/秒业务操作
- **协议**: 同时支持TCP/WebSocket/UDP，可扩展KCP/QUIC
- **序列化**: Protobuf/JSON/自定义灵活切换
- **分布式**: 无需第三方中间件即可支持集群
- **代码生成**: 自动为Godot/Unity/UE/Cocos/React/Vue等生成SDK
- **部署**: ~15MB JAR，亚秒级启动
- **来源**: https://github.com/iohao/iogame

#### 6. jforgame（kingston-csj/jforgame）
- **Stars**: 1.1k
- **License**: Apache-2.0
- **定位**: 一站式Java游戏服务器开发框架
- **最新版本**: v3.3.0（2026.03）
- **核心特性**: Socket/WebSocket接入、自定义ORM、热更新、跨进程通信
- **线程模型**: Actor-Based，防止线程负载不均
- **编解码**: JSON/Protobuf/JavaBeans
- **来源**: https://github.com/kingston-csj/jforgame

### Node.js

#### 7. casino-server（floatinghotpot/casino-server）
- **Stars**: 1.1k
- **License**: MIT
- **定位**: 基于Redis+Node.js+Socket.io的在线棋牌服务器
- **核心特性**: 跨平台、Redis消息总线、PM2集群、事件日志
- **已实现玩法**: 聊天、三张牌（金花）、德州扑克
- **待完成**: 斗地主、21点
- **Unity支持**: socket.io Unity3D客户端库
- **来源**: https://github.com/floatinghotpot/casino-server

## 项目对比矩阵

| 项目 | 语言 | Stars | License | 分布式 | 棋牌专用 | 活跃维护 |
|------|------|-------|---------|--------|----------|----------|
| Nakama | Go | 12.5k | Apache-2.0 | 原生支持 | 否（通用） | 极活跃 |
| Leaf | Go | 5.5k | Apache-2.0 | 需自建 | 否 | 停更 |
| Pitaya | Go | 2.8k | MIT | 原生支持 | 否 | 活跃 |
| Nano | Go | 3.2k | MIT | 内置 | 否 | 中等 |
| ioGame | Java | 1.2k | AGPL-3.0 | 原生支持 | 否（通用） | 活跃 |
| jforgame | Java | 1.1k | Apache-2.0 | 支持 | 否（通用） | 活跃 |
| casino-server | Node.js | 1.1k | MIT | Redis集群 | 是 | 低 |

## 关键观察

1. **没有"完美"的棋牌开源项目**：真正高星的项目（Nakama 12.5k）是通用游戏后端，棋牌专用项目（casino-server）星数和维护度较低
2. **Go语言生态最丰富**：4个主流框架，社区最活跃
3. **Java框架更"重"但功能更全**：ioGame和jforgame开箱即用组件多
4. **Node.js适合快速原型**：casino-server可直接跑起来体验，但生产环境需大量改造
5. **License注意**：ioGame使用AGPL-3.0，商业使用需注意传染性
