# 维度三：手牌管理系统

## 1. 牌数据结构设计

### 扑克牌数据结构
```typescript
// 位编码表示（参考Cactus Kev方案）
// 高16位: 花色+点数位掩码  低16位: 素数乘积值
enum Suit { SPADE = 0, HEART = 1, CLUB = 2, DIAMOND = 3 }
enum Rank { TWO = 2, THREE = 3, ..., ACE = 14 }

interface PokerCard {
    id: number;        // 唯一标识 0-53 (含大小王)
    suit: Suit;        // 花色
    rank: Rank;        // 点数
    bitmask: number;   // 位掩码（用于快速牌型判断）
}

// ID编码: id = suit * 13 + (rank - 2)
// 大王=52, 小王=53
```

### 麻将牌数据结构
```typescript
// 1×34数组编码（每种牌的数量）
// [0-8]: 万子(1-9万)  [9-17]: 条子  [18-26]: 筒子
// [27-30]: 风牌(东南西北)  [31-33]: 箭牌(中发白)

interface MahjongTile {
    type: TileType;     // WAN | TIAO | TONG | FENG | JIAN
    value: number;      // 1-9(数牌) 或 1-4(风牌) 或 1-3(箭牌)
    index: number;      // 全局编码 0-33
    instanceId: number; // 实例ID 0-135/143（区分同种牌的4张）
}

// 手牌用数组表示: handCount[34], handCount[i]表示第i种牌的数量
```

来源: [Cactus Kev Poker Evaluator](http://suffe.cool/poker/evaluator.html)
来源: [qipai Algorithm](https://github.com/hufans/qipai)

## 2. 手牌排序算法

### 扑克牌排序
```typescript
// 斗地主排序规则: 大王>小王>2>A>K>...>3
const DDZ_RANK_ORDER = [15, 14, 13, 1, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3];

function sortPokerHand(cards: PokerCard[], rule: 'standard' | 'doudizhu'): PokerCard[] {
    return cards.sort((a, b) => {
        // 先按点数降序
        const rankDiff = getRankWeight(b.rank, rule) - getRankWeight(a.rank, rule);
        if (rankDiff !== 0) return rankDiff;
        // 同点数按花色排序
        return a.suit - b.suit;
    });
}
```

### 麻将牌排序
```typescript
// 排序规则: 万子 → 条子 → 筒子 → 风牌 → 箭牌，各类内部按数值升序
function sortMahjongHand(tiles: MahjongTile[]): MahjongTile[] {
    return tiles.sort((a, b) => {
        if (a.type !== b.type) return a.type - b.type;
        return a.value - b.value;
    });
}
```

来源: [Poker Sorting Algorithm](http://alistersoft-indie-dev.blogspot.com/2015/06/simple-sorting-algorithm-for-poker-hand.html)
来源: [101 Computing Sorting](https://www.101computing.net/playing-cards-sorting-algorithm/)

## 3. 拖拽交互系统

### 扑克牌扇形展开
```typescript
// 手牌扇形布局算法
function fanLayout(cards: Node[], options: {
    maxWidth: number,    // 最大展开宽度
    cardWidth: number,   // 单牌宽度
    overlap: number,     // 最小重叠间距
    arcHeight: number,   // 弧度高度
    arcAngle: number     // 最大旋转角度
}) {
    const count = cards.length;
    const spacing = Math.min(
        options.cardWidth - options.overlap,
        options.maxWidth / count
    );
    const startX = -(count - 1) * spacing / 2;

    cards.forEach((card, i) => {
        const x = startX + i * spacing;
        const t = count > 1 ? i / (count - 1) - 0.5 : 0;
        const y = -options.arcHeight * t * t * 4; // 抛物线弧度
        const angle = options.arcAngle * t;
        card.setPosition(x, y);
        card.angle = angle;
    });
}
```

### 拖拽选牌（扑克）
```typescript
// 滑动多选模式（斗地主常用）
onTouchMove(event: Touch) {
    const touchPos = event.getLocation();
    for (const card of this.handCards) {
        if (card.getBoundingBox().contains(touchPos)) {
            card.toggleSelected();
        }
    }
}

// 选中状态：牌上移20像素
card.setPositionY(isSelected ? originalY + 20 : originalY);
```

### 拖拽出牌（麻将）
```typescript
// 麻将拖拽出牌：向上滑动超过阈值触发出牌
onTouchEnd(event: Touch) {
    const delta = event.getStartLocation().y - event.getLocation().y;
    if (delta > DRAG_THRESHOLD) {
        this.playCard(this.selectedTile);
    } else {
        this.cancelDrag();
    }
}
```

## 4. 出牌合法性校验

### 客户端预校验
客户端进行快速预校验以提供即时反馈，但最终以服务器判定为准。

```typescript
class PlayValidator {
    // 斗地主出牌校验
    validateDDZ(selected: PokerCard[], lastPlay: CardPattern | null): boolean {
        const pattern = this.recognizePattern(selected);
        if (!pattern) return false; // 不构成合法牌型
        if (!lastPlay) return true; // 首出任意合法牌型
        return pattern.type === lastPlay.type
            && pattern.length === lastPlay.length
            && pattern.rank > lastPlay.rank;
    }
}
```

### 牌型识别算法

#### 扑克牌型识别（斗地主为例）
```typescript
enum CardPatternType {
    SINGLE, PAIR, TRIPLE, TRIPLE_WITH_ONE, TRIPLE_WITH_PAIR,
    STRAIGHT, DOUBLE_STRAIGHT, TRIPLE_STRAIGHT,
    BOMB, ROCKET, AIRPLANE, AIRPLANE_WITH_WINGS
}

function recognizePattern(cards: PokerCard[]): CardPattern | null {
    const rankCount = new Map<number, number>(); // 统计每个点数的数量
    cards.forEach(c => rankCount.set(c.rank, (rankCount.get(c.rank) || 0) + 1));

    const counts = [...rankCount.values()].sort((a, b) => b - a);
    const n = cards.length;

    // 火箭（双王）
    if (n === 2 && cards.every(c => c.rank >= 14)) return { type: ROCKET };
    // 炸弹
    if (n === 4 && counts[0] === 4) return { type: BOMB, rank: ... };
    // 单张/对子/三条
    if (n === 1) return { type: SINGLE };
    if (n === 2 && counts[0] === 2) return { type: PAIR };
    // ... 依次判断更复杂牌型
}
```

#### 扑克手牌评估（德州扑克）
三种主流算法：
1. **直方图法**：统计点数和花色频率，识别对子/三条/同花等模式
2. **位运算法**：用位掩码表示牌，快速判断顺子和同花
3. **查表法（Cactus Kev）**：为每种点数分配素数，乘积作为哈希键查表，7462种不同手牌等级

来源: [Cactus Kev Evaluator](http://suffe.cool/poker/evaluator.html)
来源: [Poker Hand Evaluator](https://github.com/HenryRLee/PokerHandEvaluator)
来源: [Innosoft - Poker Algorithms](https://innosoft-group.com/essential-algorithms-for-poker-hand-ranking-game-logic/)

#### 麻将胡牌检测
```typescript
// 标准胡牌: 4组面子(顺子/刻子) + 1对将
function canWin(handCount: number[]): boolean {
    // 遍历每种牌作为将牌
    for (let i = 0; i < 34; i++) {
        if (handCount[i] >= 2) {
            handCount[i] -= 2;
            if (canFormMelds(handCount, 0)) {
                handCount[i] += 2;
                return true;
            }
            handCount[i] += 2;
        }
    }
    return false;
}

function canFormMelds(hand: number[], startIdx: number): boolean {
    // 找到第一张牌
    let idx = startIdx;
    while (idx < 34 && hand[idx] === 0) idx++;
    if (idx >= 34) return true; // 所有牌消耗完毕，胡牌

    // 尝试刻子
    if (hand[idx] >= 3) {
        hand[idx] -= 3;
        if (canFormMelds(hand, idx)) { hand[idx] += 3; return true; }
        hand[idx] += 3;
    }
    // 尝试顺子（仅数牌，且不跨类型）
    if (idx < 27 && idx % 9 <= 6 && hand[idx+1] > 0 && hand[idx+2] > 0) {
        hand[idx]--; hand[idx+1]--; hand[idx+2]--;
        if (canFormMelds(hand, idx)) {
            hand[idx]++; hand[idx+1]++; hand[idx+2]++;
            return true;
        }
        hand[idx]++; hand[idx+1]++; hand[idx+2]++;
    }
    return false;
}
```

**优化方案**：
- 查表法：按花色切分后查表，性能极高（100万次/秒@四癞子）
- 缓存向听数（shanten）计算结果
- 特殊牌型独立判断（七对、十三幺、清一色等）

来源: [qipai Mahjong Algorithm](https://github.com/hufans/qipai)
来源: [Mahjong Winning Algorithm](https://github.com/dingmingxin/mahjong-1)
