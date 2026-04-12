# 维度六：可复用组件库

## 1. 标准扑克牌组件（52+2张）

### 牌面资源管理
```typescript
class PokerDeckAssets {
    // 资源命名规范: {suit}_{rank}.png
    // spade_A.png, heart_2.png, ..., joker_big.png, joker_small.png
    // 牌背: card_back.png（支持多款牌背皮肤）

    private atlas: SpriteAtlas; // 合图，减少DrawCall

    getSpriteFrame(card: PokerCard): SpriteFrame {
        const suitName = ['spade', 'heart', 'club', 'diamond'][card.suit];
        const rankName = card.rank <= 10 ? card.rank.toString()
            : ['J', 'Q', 'K', 'A'][card.rank - 11];
        return this.atlas.getSpriteFrame(`${suitName}_${rankName}`);
    }

    getBackSpriteFrame(skinId: string = 'default'): SpriteFrame {
        return this.atlas.getSpriteFrame(`back_${skinId}`);
    }
}
```

### 扑克牌节点组件
```typescript
@ccclass('PokerCardView')
class PokerCardView extends Component {
    @property(Sprite) face: Sprite;
    @property(Sprite) back: Sprite;
    @property(Node) selectIndicator: Node;

    private _data: PokerCard;
    private _faceUp: boolean = false;
    private _selected: boolean = false;

    init(data: PokerCard, faceUp: boolean = false) {
        this._data = data;
        this.face.spriteFrame = PokerDeckAssets.inst.getSpriteFrame(data);
        this.setFaceUp(faceUp);
    }

    setFaceUp(faceUp: boolean, animate: boolean = false) {
        this._faceUp = faceUp;
        if (animate) {
            this.playFlipAnimation(faceUp);
        } else {
            this.face.node.active = faceUp;
            this.back.node.active = !faceUp;
        }
    }

    setSelected(selected: boolean) {
        this._selected = selected;
        // 选中时上移20像素
        tween(this.node).to(0.1, {
            position: new Vec3(this.node.position.x,
                selected ? this._baseY + 20 : this._baseY, 0)
        }).start();
    }
}
```

来源: [CardHouse Unity Framework](https://github.com/pipeworks-studios/CardHouse)

## 2. 麻将牌组件（136/144张）

### 麻将牌资源管理
```typescript
class MahjongTileAssets {
    // 资源命名: {type}_{value}.png
    // wan_1.png ~ wan_9.png (万子)
    // tiao_1.png ~ tiao_9.png (条子)
    // tong_1.png ~ tong_9.png (筒子)
    // feng_1.png ~ feng_4.png (东南西北)
    // jian_1.png ~ jian_3.png (中发白)
    // 花牌(可选): hua_1.png ~ hua_8.png

    // 不同视角的牌面:
    // 立牌(手牌): tile_stand_{type}_{value}.png
    // 横牌(吃碰杠): tile_lie_{type}_{value}.png
    // 倒牌(出牌区): tile_flat_{type}_{value}.png
    // 牌背: tile_back_stand.png, tile_back_lie.png
}
```

### 麻将牌状态
```typescript
enum TileState {
    IN_HAND,      // 手牌（立牌）
    DISCARDED,    // 弃牌（平放）
    MELDED,       // 副露（横放，吃/碰/杠）
    CONCEALED,    // 暗杠
    DRAWN         // 刚摸的牌（手牌末尾，稍有间隔）
}

@ccclass('MahjongTileView')
class MahjongTileView extends Component {
    @property(Sprite) tileSprite: Sprite;
    @property(Node) glowEffect: Node; // 可选牌高亮

    private _state: TileState;

    setState(state: TileState) {
        this._state = state;
        switch (state) {
            case TileState.IN_HAND:
                this.node.angle = 0;
                this.node.scale = new Vec3(1, 1, 1);
                break;
            case TileState.DISCARDED:
                this.node.scale = new Vec3(0.7, 0.7, 1);
                break;
            case TileState.MELDED:
                // 碰杠区域的牌稍小
                this.node.scale = new Vec3(0.75, 0.75, 1);
                break;
        }
    }

    setHighlight(enable: boolean) {
        this.glowEffect.active = enable;
    }
}
```

## 3. 筹码组件

```typescript
@ccclass('ChipDisplay')
class ChipDisplay extends Component {
    @property(Label) amountLabel: Label;
    @property([SpriteFrame]) chipSprites: SpriteFrame[] = [];
    // chipSprites[0]=1, [1]=5, [2]=25, [3]=100, [4]=500, [5]=1000

    private _amount: number = 0;

    setAmount(amount: number, animate: boolean = true) {
        if (animate) {
            // 数字滚动动画
            const from = this._amount;
            const duration = Math.min(0.5, Math.abs(amount - from) / 1000);
            const obj = { value: from };
            tween(obj).to(duration, { value: amount }, {
                onUpdate: () => {
                    this.amountLabel.string = this.formatAmount(Math.floor(obj.value));
                }
            }).start();
        } else {
            this.amountLabel.string = this.formatAmount(amount);
        }
        this._amount = amount;
    }

    private formatAmount(amount: number): string {
        if (amount >= 1000000) return (amount / 1000000).toFixed(1) + 'M';
        if (amount >= 1000) return (amount / 1000).toFixed(1) + 'K';
        return amount.toString();
    }

    // 筹码堆叠视觉
    updateChipStack(amount: number) {
        const denominations = [1000, 500, 100, 25, 5, 1];
        let remaining = amount;
        // 贪心算法分解筹码面值
        denominations.forEach((denom, i) => {
            const count = Math.floor(remaining / denom);
            remaining %= denom;
            // 显示对应面值的筹码图标×count
        });
    }
}
```

## 4. 计时器组件

```typescript
@ccclass('TurnTimer')
class TurnTimer extends Component {
    @property(Label) timeLabel: Label;
    @property(Sprite) progressRing: Sprite;  // 环形进度条
    @property(Node) urgentEffect: Node;      // 紧急效果（闪烁/变红）

    private totalTime: number = 0;
    private remainingTime: number = 0;
    private isRunning: boolean = false;
    private urgentThreshold: number = 5; // 小于5秒进入紧急状态

    start(seconds: number) {
        this.totalTime = seconds;
        this.remainingTime = seconds;
        this.isRunning = true;
        this.urgentEffect.active = false;
    }

    update(dt: number) {
        if (!this.isRunning) return;

        this.remainingTime -= dt;
        if (this.remainingTime <= 0) {
            this.remainingTime = 0;
            this.isRunning = false;
            EventBus.emit(GameEvent.TIMER_EXPIRED);
        }

        // 更新显示
        this.timeLabel.string = Math.ceil(this.remainingTime).toString();
        this.progressRing.fillRange = this.remainingTime / this.totalTime;

        // 紧急状态
        if (this.remainingTime <= this.urgentThreshold && this.remainingTime > 0) {
            this.urgentEffect.active = true;
            // 每秒闪烁 + 音效
            if (Math.floor(this.remainingTime) !== Math.floor(this.remainingTime + dt)) {
                this.playTickSound();
            }
        }
    }

    // 服务器时间同步校正
    syncWithServer(serverRemaining: number) {
        const drift = Math.abs(this.remainingTime - serverRemaining);
        if (drift > 1) {
            // 偏差超过1秒，直接矫正
            this.remainingTime = serverRemaining;
        } else if (drift > 0.2) {
            // 小偏差，渐进矫正
            this.remainingTime += (serverRemaining - this.remainingTime) * 0.5;
        }
    }
}
```

## 5. 聊天组件

```typescript
@ccclass('GameChat')
class GameChat extends Component {
    // 快捷短语（预定义，避免内容审核问题）
    static QUICK_PHRASES = [
        { id: 1, text: '快点吧，等到花都谢了' },
        { id: 2, text: '不好意思，我手滑了' },
        { id: 3, text: '大哥，你真厉害' },
        { id: 4, text: '不要走，再来一局' },
        { id: 5, text: '你的牌打得太好了' },
        { id: 6, text: '同意' },
    ];

    // 表情面板
    @property(Node) emojiPanel: Node;
    // 快捷短语列表
    @property(Node) phrasePanel: Node;
    // 聊天气泡容器
    @property(Node) bubbleContainer: Node;

    sendPhrase(phraseId: number) {
        this.socket.send(new ChatMessage({ type: 'phrase', content: phraseId }));
    }

    sendEmoji(emojiId: string) {
        this.socket.send(new ChatMessage({ type: 'emoji', content: emojiId }));
    }

    showBubble(seatIndex: number, content: string, duration: number = 3) {
        const seat = this.getSeat(seatIndex);
        const bubble = instantiate(this.bubblePrefab);
        bubble.getComponent(Label).string = content;
        bubble.setPosition(seat.position.x, seat.position.y + 100);
        this.bubbleContainer.addChild(bubble);

        tween(bubble)
            .delay(duration)
            .to(0.3, { opacity: 0 })
            .call(() => bubble.destroy())
            .start();
    }
}
```

## 6. 结算组件

```typescript
interface SettlementData {
    players: {
        seatIndex: number;
        nickname: string;
        avatar: string;
        scoreChange: number; // 正为赢，负为输
        handCards: Card[];   // 手牌展示
        handRank?: string;   // 牌型名称（如"同花顺"）
        isWinner: boolean;
    }[];
    totalPot?: number;       // 底池总额（德州扑克）
    roundInfo: string;       // 本局信息
}

@ccclass('SettlementPanel')
class SettlementPanel extends Component {
    show(data: SettlementData) {
        // 1. 显示所有玩家手牌（翻牌动画）
        // 2. 胜负高亮（赢家放大+金色边框，输家灰色）
        // 3. 分数变化动画（+1200 / -500）
        // 4. 如有大牌型，播放演出动画
        // 5. 底部按钮：[再来一局] [换桌] [退出]
    }
}
```

## 7. 战绩回放组件

```typescript
interface ReplayData {
    gameType: string;
    startTime: number;
    players: PlayerInfo[];
    events: GameEvent[];  // 按时间排序的所有游戏事件
    result: SettlementData;
}

@ccclass('ReplayPlayer')
class ReplayPlayer extends Component {
    private events: GameEvent[] = [];
    private currentIndex: number = 0;
    private speed: number = 1; // 1x, 2x, 4x

    load(replayData: ReplayData) {
        this.events = replayData.events;
        this.currentIndex = 0;
        // 初始化牌桌到起始状态
    }

    play() { this.isPlaying = true; }
    pause() { this.isPlaying = false; }
    setSpeed(speed: number) { this.speed = speed; }

    seekTo(eventIndex: number) {
        // 从头重放到指定事件
        this.resetToInitial();
        for (let i = 0; i <= eventIndex; i++) {
            this.applyEvent(this.events[i], false); // 不播动画
        }
        this.currentIndex = eventIndex;
    }

    update(dt: number) {
        if (!this.isPlaying) return;
        // 按速度播放下一个事件
        this.eventTimer += dt * this.speed;
        while (this.currentIndex < this.events.length) {
            const event = this.events[this.currentIndex];
            if (event.timestamp <= this.eventTimer) {
                this.applyEvent(event, true); // 播放动画
                this.currentIndex++;
            } else {
                break;
            }
        }
    }

    private applyEvent(event: GameEvent, animate: boolean) {
        switch (event.type) {
            case 'deal': this.animateDeal(event, animate); break;
            case 'play': this.animatePlay(event, animate); break;
            case 'draw': this.animateDraw(event, animate); break;
            case 'settle': this.showSettlement(event); break;
        }
    }
}
```

来源: [CardHouse Framework](https://github.com/pipeworks-studios/CardHouse)
来源: [Cocos Creator Game Development](https://docs.cocos.com/creator/3.8/manual/en/)
