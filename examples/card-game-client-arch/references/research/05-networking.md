# 维度五：网络通信层

## 1. WebSocket长连接管理

### 连接生命周期
```
[未连接] → connect() → [连接中] → onopen → [已连接] → onclose → [已断开]
                                                         ↓
                                              heartbeat → [心跳超时] → reconnect()
```

### WebSocket封装
```typescript
class GameSocket {
    private ws: WebSocket | null = null;
    private url: string;
    private reconnectAttempts = 0;
    private maxReconnectAttempts = 12;
    private heartbeatTimer: number = 0;
    private messageQueue: Uint8Array[] = []; // 断线期间消息缓存

    connect(url: string) {
        this.url = url;
        this.ws = new WebSocket(url);
        this.ws.binaryType = 'arraybuffer';

        this.ws.onopen = () => {
            this.reconnectAttempts = 0;
            this.startHeartbeat();
            this.flushMessageQueue();
            EventBus.emit(NetEvent.CONNECTED);
        };

        this.ws.onmessage = (event) => {
            this.resetHeartbeat();
            const packet = Packet.decode(event.data);
            this.dispatch(packet);
        };

        this.ws.onclose = (event) => {
            this.stopHeartbeat();
            if (!event.wasClean) {
                this.scheduleReconnect();
            }
            EventBus.emit(NetEvent.DISCONNECTED, event.code);
        };
    }
}
```

来源: [DEV - Building Scalable Card Games](https://dev.to/krishanvijay/building-scalable-real-time-multiplayer-card-games-3kn6)

## 2. 断线重连

### 指数退避策略
```typescript
private scheduleReconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
        EventBus.emit(NetEvent.RECONNECT_FAILED);
        return;
    }

    // 指数退避 + 随机抖动
    const baseDelay = 500; // 起始500ms
    const maxDelay = 30000; // 上限30s
    const delay = Math.min(
        baseDelay * Math.pow(2, this.reconnectAttempts),
        maxDelay
    );
    const jitter = delay * 0.3 * Math.random(); // 30%随机抖动

    this.reconnectAttempts++;
    EventBus.emit(NetEvent.RECONNECTING, {
        attempt: this.reconnectAttempts,
        maxAttempts: this.maxReconnectAttempts,
        nextDelay: delay + jitter
    });

    setTimeout(() => this.connect(this.url), delay + jitter);
}
```

来源: [WebSocket Reconnection Guide](https://websocket.org/guides/reconnection/)
来源: [DEV - Exponential Backoff](https://dev.to/hexshift/robust-websocket-reconnection-strategies-in-javascript-with-exponential-backoff-40n1)

### 重连后状态恢复
```typescript
// 重连成功后请求全量状态快照
async onReconnected() {
    // 1. 发送重连请求（带上最后一条确认的序列号）
    this.send(new ReconnectRequest({
        sessionId: this.sessionId,
        lastSeqNum: this.lastAckedSeqNum
    }));

    // 2. 服务器回复：全量状态快照 + 断线期间的增量事件
    // 3. 客户端重放增量事件，恢复到当前状态
    // 4. 恢复UI渲染
}
```

### 重连UI提示
```
断线: 显示半透明遮罩 + "网络连接断开，正在重连..." + 进度指示
重连中: 显示重连尝试次数 "第3次重连中..."
重连成功: 遮罩消失 + Toast "网络已恢复"
重连失败: 弹窗 "网络连接失败，是否重试？" [重试] [返回大厅]
```

## 3. 心跳检测

```typescript
private heartbeatInterval = 15000; // 15秒发一次心跳
private heartbeatTimeout = 5000;   // 5秒无回复视为超时

startHeartbeat() {
    this.heartbeatTimer = setInterval(() => {
        this.send(new HeartbeatRequest({ timestamp: Date.now() }));

        // 超时检测
        this.heartbeatTimeoutId = setTimeout(() => {
            console.warn('Heartbeat timeout');
            this.ws?.close(4000, 'heartbeat_timeout');
        }, this.heartbeatTimeout);
    }, this.heartbeatInterval);
}

onHeartbeatResponse(res: HeartbeatResponse) {
    clearTimeout(this.heartbeatTimeoutId);
    // 计算RTT
    this.rtt = Date.now() - res.clientTimestamp;
    // 同步服务器时间
    this.serverTimeDelta = res.serverTimestamp - Date.now() + this.rtt / 2;
}
```

## 4. 协议格式选择

### Protobuf vs JSON 对比

| 维度 | Protobuf | JSON | FlatBuffers |
|------|----------|------|-------------|
| 包体大小 | 小（二进制编码） | 大（文本格式） | 最小 |
| 编解码速度 | 快 | 慢 | 最快（零拷贝） |
| 可读性 | 差（需工具） | 好（人类可读） | 差 |
| Schema演进 | 支持（字段编号） | 不需要schema | 支持 |
| 工具链成熟度 | 高 | 高 | 中 |
| 适用场景 | 正式环境 | 开发调试 | 极致性能需求 |

### 推荐方案
- **开发阶段**：用JSON，方便调试和抓包分析
- **生产环境**：用Protobuf，包体小、解析快
- **策略**：封装统一的序列化接口，通过配置切换

来源: [Protobuf vs FlatBuffers](https://www.oreateai.com/blog/protobuf-vs-flatbuffers-choosing-the-right-serialization-framework-for-realtime-applications/f410bfacd86a9235f73cf2747c5026aa)

### 消息协议设计
```protobuf
// 通用消息头
message PacketHeader {
    uint32 cmd_id = 1;       // 协议号
    uint32 seq_num = 2;      // 序列号
    uint64 timestamp = 3;    // 时间戳
    uint32 ack_seq = 4;      // 确认号
}

// 示例：出牌请求
message PlayCardRequest {
    repeated uint32 card_ids = 1;    // 出的牌
    uint32 seat_index = 2;           // 座位号
}

// 示例：出牌广播
message PlayCardNotify {
    uint32 seat_index = 1;
    repeated uint32 card_ids = 2;
    uint32 next_seat = 3;            // 下一个出牌者
    uint32 remaining_time = 4;       // 剩余思考时间(秒)
}
```

## 5. 客户端预测与服务器校验

### 棋牌游戏的预测策略
棋牌游戏不像FPS需要激进的预测，但仍可在以下场景做乐观更新：

```typescript
class OptimisticUpdater {
    // 乐观出牌：立即播放动画，等服务器确认
    playCard(cards: Card[]) {
        // 1. 客户端预校验
        if (!this.validator.validate(cards)) return;

        // 2. 乐观更新UI（立即播放出牌动画）
        const predictId = this.nextPredictId++;
        this.predictions.set(predictId, {
            cards,
            handSnapshot: this.hand.clone()
        });
        this.view.playCardAnimation(cards);

        // 3. 发送请求到服务器
        this.socket.send(new PlayCardRequest({
            cardIds: cards.map(c => c.id),
            predictId
        }));
    }

    // 服务器确认
    onServerConfirm(predictId: number) {
        this.predictions.delete(predictId);
        // 预测正确，无需额外处理
    }

    // 服务器拒绝（回滚）
    onServerReject(predictId: number, reason: string) {
        const prediction = this.predictions.get(predictId);
        if (prediction) {
            // 回滚到出牌前状态
            this.hand.restore(prediction.handSnapshot);
            this.view.rollbackAnimation();
            this.showError(reason);
        }
        this.predictions.delete(predictId);
    }
}
```

### 状态哈希校验
```typescript
// 每次关键操作后计算状态哈希
function computeStateHash(gameState: GameState): string {
    const data = JSON.stringify({
        hands: gameState.hands,
        discards: gameState.discards,
        scores: gameState.scores,
        turn: gameState.currentTurn
    });
    return sha256(data);
}

// 服务器在广播中附带stateHash，客户端比对
onGameStateUpdate(update: GameStateUpdate) {
    this.applyUpdate(update);
    const localHash = computeStateHash(this.gameState);
    if (localHash !== update.stateHash) {
        // 状态不一致，请求全量同步
        this.requestFullSync();
    }
}
```

来源: [Client Prediction & Reconciliation](https://www.gabrielgambetta.com/client-side-prediction-server-reconciliation.html)
来源: [Real-Time Card Games Architecture](https://developersvoice.com/blog/practical-design/realtime-card-games-net-architecture-guide/)

## 6. Mock Server

### 本地Mock Server架构
```typescript
class MockGameServer {
    private gameState: GameState;
    private rules: IGameRule;

    // 模拟网络延迟
    private latency = { min: 50, max: 200 };

    async handleMessage(packet: Packet): Promise<Packet> {
        await this.simulateLatency();

        switch (packet.cmdId) {
            case CMD.PLAY_CARD:
                return this.handlePlayCard(packet.data);
            case CMD.DRAW_CARD:
                return this.handleDrawCard(packet.data);
            // ...
        }
    }

    private simulateLatency(): Promise<void> {
        const delay = this.latency.min +
            Math.random() * (this.latency.max - this.latency.min);
        return new Promise(resolve => setTimeout(resolve, delay));
    }
}

// 开发时通过配置切换真实服务器/Mock服务器
const socket = config.useMock
    ? new MockSocketAdapter(new MockGameServer())
    : new WebSocketAdapter(config.serverUrl);
```

### Mock数据生成
```typescript
class MockDataGenerator {
    // 生成随机手牌
    static generateHand(gameType: 'poker' | 'mahjong', count: number): Card[] { ... }
    // 生成模拟对手行为
    static simulateOpponentAction(gameState: GameState): Action { ... }
    // 生成录像回放数据
    static generateReplayData(rounds: number): ReplayData { ... }
}
```
