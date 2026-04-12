# 维度四：动画与演出系统

## 1. 发牌动画

### 发牌动画序列
```typescript
async function dealCards(deck: Node, targets: SeatInfo[], cardsPerPlayer: number) {
    const dealQueue: { card: Node, target: Vec3, delay: number }[] = [];
    let dealIndex = 0;

    // 逐张发牌，每轮给每个玩家发一张
    for (let round = 0; round < cardsPerPlayer; round++) {
        for (const seat of targets) {
            dealQueue.push({
                card: createCardNode(),
                target: seat.handPosition,
                delay: dealIndex * 0.08 // 每张牌间隔80ms
            });
            dealIndex++;
        }
    }

    // 执行发牌动画
    const promises = dealQueue.map(item =>
        new Promise<void>(resolve => {
            tween(item.card)
                .delay(item.delay)
                .call(() => playDealSound())
                .to(0.3, {
                    position: item.target,
                    scale: new Vec3(1, 1, 1)
                }, { easing: 'cubicOut' })
                .call(() => resolve())
                .start();
        })
    );

    await Promise.all(promises);
}
```

### 关键设计要点
- 发牌间隔 60-100ms，太快看不清，太慢影响节奏
- 牌从牌堆位置飞向目标位置，使用cubicOut缓动
- 自己的牌到达后翻转显示正面，其他人的牌保持牌背
- 音效与每张牌到达同步

## 2. 出牌动画

### 扑克出牌
```typescript
async function playCardAnimation(card: Node, playZone: Vec3) {
    // 先放大强调
    await tweenPromise(card,
        tween(card)
            .to(0.1, { scale: new Vec3(1.2, 1.2, 1) })
            .to(0.2, {
                position: playZone,
                scale: new Vec3(0.8, 0.8, 1)
            }, { easing: 'backOut' })
    );
    // 手牌重新排列（带动画）
    this.relayoutHandCards(0.2);
}
```

### 麻将出牌
```typescript
async function discardTile(tile: Node, discardZone: Vec3) {
    // 上移 → 飞向出牌区 → 缩小落定
    await tweenPromise(tile,
        tween(tile)
            .to(0.1, { position: new Vec3(tile.position.x, tile.position.y + 30, 0) })
            .to(0.25, { position: discardZone, scale: new Vec3(0.7, 0.7, 1) }, { easing: 'quadOut' })
    );
}
```

## 3. 翻牌动画

### 3D翻转效果（2D模拟）
```typescript
function flipCard(card: Node, frontSprite: SpriteFrame, duration: number = 0.4) {
    const halfDuration = duration / 2;
    tween(card)
        // 前半段：缩小X到0（模拟翻转到侧面）
        .to(halfDuration, { scale: new Vec3(0, 1, 1) }, { easing: 'sineIn' })
        .call(() => {
            // 中间点切换纹理
            card.getComponent(Sprite).spriteFrame = frontSprite;
        })
        // 后半段：从0恢复到正常（模拟翻转到正面）
        .to(halfDuration, { scale: new Vec3(1, 1, 1) }, { easing: 'sineOut' })
        .start();
}
```

来源: [Card Flip in Unity](https://metaxyntax.neocities.org/entries/4)
来源: [Cocos2D Flip Animation](https://jademind.com/blog/posts/cocos2d-flip-sprite-animation/)

### Y轴旋转翻牌（3D引擎）
```typescript
// Unity / Cocos Creator 3D
function flipCard3D(card: Node, duration: number = 0.5) {
    tween(card)
        .to(duration / 2, { eulerAngles: new Vec3(0, 90, 0) }, { easing: 'sineIn' })
        .call(() => switchToFrontTexture(card))
        .to(duration / 2, { eulerAngles: new Vec3(0, 0, 0) }, { easing: 'sineOut' })
        .start();
}
```

## 4. 胡牌/大牌型演出

### 演出等级系统
```typescript
enum ShowLevel {
    NORMAL = 0,    // 普通胡牌：简单文字提示
    SPECIAL = 1,   // 特殊牌型：全屏横幅 + 粒子
    EPIC = 2,      // 史诗牌型：全屏动画 + 音效 + 震屏
    LEGENDARY = 3  // 传奇牌型：CG动画 + 全屏特效
}

const SHOW_CONFIG = {
    [ShowLevel.NORMAL]: { duration: 1.5, particles: false, shake: false },
    [ShowLevel.SPECIAL]: { duration: 2.5, particles: true, shake: false },
    [ShowLevel.EPIC]: { duration: 3.5, particles: true, shake: true },
    [ShowLevel.LEGENDARY]: { duration: 5.0, particles: true, shake: true, spine: true }
};
```

### 演出队列管理
```typescript
class ShowDirector {
    private queue: ShowTask[] = [];
    private isPlaying = false;

    enqueue(task: ShowTask) {
        this.queue.push(task);
        if (!this.isPlaying) this.playNext();
    }

    private async playNext() {
        if (this.queue.length === 0) {
            this.isPlaying = false;
            EventBus.emit(GameEvent.SHOW_COMPLETE);
            return;
        }
        this.isPlaying = true;
        const task = this.queue.shift();
        await task.execute();
        this.playNext();
    }

    // 跳过当前演出（玩家点击屏幕）
    skip() {
        if (this.currentTask) {
            this.currentTask.fastForward();
        }
    }
}
```

## 5. 筹码动画

```typescript
// 筹码从玩家飞向底池
async function chipsToPot(fromPos: Vec3, potPos: Vec3, amount: number) {
    const chipCount = Math.min(Math.ceil(amount / 100), 10); // 最多10个筹码图标
    const promises = [];

    for (let i = 0; i < chipCount; i++) {
        const chip = createChipNode();
        chip.setPosition(fromPos);
        promises.push(new Promise<void>(resolve => {
            tween(chip)
                .delay(i * 0.05)
                .to(0.4, { position: potPos }, { easing: 'quadIn' })
                .call(() => {
                    chip.destroy();
                    resolve();
                })
                .start();
        }));
    }

    await Promise.all(promises);
    // 更新底池数字（数字滚动动画）
    this.animateNumber(this.potLabel, oldAmount, oldAmount + amount, 0.3);
}
```

## 6. 表情系统

```typescript
interface EmoteConfig {
    id: string;
    type: 'static' | 'animated' | 'spine';
    resource: string;
    duration: number;       // 显示时长
    position: 'above_seat' | 'center_screen' | 'flying';
}

class EmoteManager {
    play(seatIndex: number, emoteId: string) {
        const config = EMOTE_CONFIGS[emoteId];
        const seat = this.getSeat(seatIndex);
        const emoteNode = this.createEmote(config);

        // 在座位上方显示，自动消失
        emoteNode.setPosition(seat.position.x, seat.position.y + 80);
        tween(emoteNode)
            .to(0.2, { scale: new Vec3(1, 1, 1) }, { easing: 'backOut' })
            .delay(config.duration)
            .to(0.3, { opacity: 0 })
            .call(() => emoteNode.destroy())
            .start();
    }
}
```

## 7. Tween缓动库封装

### Cocos Creator Tween最佳实践

```typescript
// 封装Promise化的tween工具
function tweenPromise<T>(target: T, tw: Tween<T>): Promise<void> {
    return new Promise(resolve => {
        tw.call(() => resolve()).start();
    });
}

// 常用动画预设
const AnimPresets = {
    // 弹入
    popIn: (node: Node, duration = 0.3) =>
        tween(node).set({ scale: Vec3.ZERO })
            .to(duration, { scale: Vec3.ONE }, { easing: 'backOut' }),

    // 飞入
    flyTo: (node: Node, target: Vec3, duration = 0.4) =>
        tween(node).to(duration, { position: target }, { easing: 'cubicOut' }),

    // 震动
    shake: (node: Node, intensity = 5, times = 5) => {
        const original = node.position.clone();
        let tw = tween(node);
        for (let i = 0; i < times; i++) {
            tw = tw.to(0.03, { position: v3(original.x + intensity, original.y, 0) })
                    .to(0.03, { position: v3(original.x - intensity, original.y, 0) });
        }
        return tw.to(0.03, { position: original });
    },

    // 数字滚动
    countTo: (label: Label, from: number, to: number, duration = 0.5) => {
        const obj = { value: from };
        return tween(obj).to(duration, { value: to }, {
            onUpdate: () => { label.string = Math.floor(obj.value).toString(); }
        });
    }
};
```

来源: [Cocos Creator Tween System](https://docs.cocos.com/creator/3.8/manual/en/tween/)
来源: [Cocos Creator Tween Examples](https://docs.cocos.com/creator/3.8/manual/en/tween/tween-example.html)

## 8. 粒子特效标准化

### 特效分类与配置
```typescript
const PARTICLE_PRESETS = {
    WIN_CELEBRATION: {
        texture: 'particles/star',
        emissionRate: 50, lifeTime: 1.5,
        startSize: 20, endSize: 0,
        startColor: Color.YELLOW, endColor: new Color(255, 200, 0, 0),
        gravity: { x: 0, y: -200 },
        emitterMode: 'GRAVITY'
    },
    CHIP_TRAIL: {
        texture: 'particles/circle',
        emissionRate: 30, lifeTime: 0.5,
        startSize: 8, endSize: 2,
        startColor: Color.WHITE, endColor: new Color(255, 215, 0, 0),
        emitterMode: 'GRAVITY'
    },
    CARD_GLOW: {
        texture: 'particles/glow',
        emissionRate: 10, lifeTime: 1.0,
        startSize: 40, endSize: 60,
        startColor: new Color(255, 255, 200, 150), endColor: new Color(255, 255, 200, 0),
        emitterMode: 'RADIUS'
    }
};
```
