# 维度二：牌桌渲染系统

## 1. 牌桌布局算法

### 圆桌布局（德州扑克/麻将）
适用于3-10人的多人棋牌游戏。

```typescript
function getCircularSeatPositions(
    playerCount: number,
    radius: number,
    center: Vec2,
    startAngle: number = -Math.PI / 2 // 自己在底部
): Vec2[] {
    const positions: Vec2[] = [];
    const angleStep = (2 * Math.PI) / playerCount;
    for (let i = 0; i < playerCount; i++) {
        const angle = startAngle + i * angleStep;
        positions.push(new Vec2(
            center.x + radius * Math.cos(angle),
            center.y + radius * Math.sin(angle)
        ));
    }
    return positions;
}
```

### 半圆布局（斗地主/三人场）
自己固定在底部中央，其他玩家分布在上半圆。

```typescript
function getSemiCircleSeatPositions(
    playerCount: number,
    radius: number,
    center: Vec2
): Vec2[] {
    const positions: Vec2[] = [];
    // 自己在底部中央
    positions.push(new Vec2(center.x, center.y - radius));
    // 其他玩家均匀分布在上半圆
    const otherCount = playerCount - 1;
    const angleRange = Math.PI; // 180度
    const angleStep = angleRange / (otherCount + 1);
    for (let i = 1; i <= otherCount; i++) {
        const angle = -Math.PI / 2 + i * angleStep;
        positions.push(new Vec2(
            center.x + radius * Math.cos(angle),
            center.y + radius * Math.sin(angle)
        ));
    }
    return positions;
}
```

### 方桌布局（麻将四人桌）
四个固定位置：下/右/上/左（对应东南西北或自己/下家/对家/上家）。

```typescript
const MAHJONG_SEATS = {
    SELF:     { pos: Vec2(0, -H/2), rotation: 0 },
    RIGHT:    { pos: Vec2(W/2, 0),  rotation: 90 },
    OPPOSITE: { pos: Vec2(0, H/2),  rotation: 180 },
    LEFT:     { pos: Vec2(-W/2, 0), rotation: 270 }
};
```

## 2. 座位系统设计

### 座位数据模型
```typescript
interface SeatInfo {
    seatIndex: number;         // 服务器座位号
    localIndex: number;        // 本地显示序号（自己=0）
    position: Vec2;            // 屏幕坐标
    rotation: number;          // 朝向角度
    player: PlayerInfo | null; // 玩家信息
    isLocal: boolean;          // 是否是本地玩家
}
```

### 座位映射算法
将服务器的绝对座位号映射为本地玩家视角的相对座位号：

```typescript
function mapToLocalSeat(serverSeatIndex: number, mySeatIndex: number, totalSeats: number): number {
    return (serverSeatIndex - mySeatIndex + totalSeats) % totalSeats;
}
```

## 3. 视角管理

### 核心原则
- 本地玩家始终在屏幕底部
- 其他玩家根据座位关系顺时针排列
- 手牌只展示本地玩家的正面，其他玩家显示牌背
- 公共区域（出牌区、公共牌）居中显示

### 视角旋转
对于麻将等需要旋转视角的游戏：
- 对家的牌倒置显示（180度旋转）
- 左右家的牌侧放（90度/270度旋转）
- 出牌区域分区显示，每个方向有独立出牌区

## 4. 响应式适配

### 多分辨率适配策略
```typescript
// 基于设计分辨率的缩放策略
const DESIGN_WIDTH = 1920;
const DESIGN_HEIGHT = 1080;

function adaptLayout(screenWidth: number, screenHeight: number) {
    const scaleX = screenWidth / DESIGN_WIDTH;
    const scaleY = screenHeight / DESIGN_HEIGHT;

    // 牌桌背景: SHOW_ALL（完整显示）
    // 手牌区域: 固定在底部，宽度自适应
    // 座位布局: 按比例缩放radius参数
    return {
        tableScale: Math.min(scaleX, scaleY),
        handAreaWidth: screenWidth,
        seatRadius: BASE_RADIUS * Math.min(scaleX, scaleY)
    };
}
```

### 安全区域处理
- 刘海屏/异形屏适配：手牌区域避开底部安全区
- 横屏锁定：棋牌游戏强制横屏
- iPad等宽屏设备：牌桌居中，两侧留白

### Cocos Creator适配方案
- 使用Widget组件锚定UI元素到安全区边界
- Canvas的适配模式选择SHOW_ALL或FIXED_WIDTH
- 不同设备上动态调整座位radius和牌的缩放比

来源: [Cocos Creator 2D Game Development](https://unityunreal.com/tutorials/2194-2d-game-development-with-cocos-creator-the-ultimate-guide.html)
来源: [Tabletopia Layout](https://help.tabletopia.com/knowledge-base/arranging-the-space-and-objects/)
