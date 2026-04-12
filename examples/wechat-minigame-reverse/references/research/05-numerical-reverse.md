# 维度5：数值体系逆向

## 数值数据定位策略

### 本地配置文件
小游戏的数值配置通常存放在以下位置：

| 位置 | 格式 | 特征 |
|------|------|------|
| `src/settings.js` | JS对象 | Cocos Creator的全局配置 |
| `assets/*.json` | JSON | 场景/预制体中的组件参数 |
| `res/config/*.json` | JSON | 独立配置文件 |
| `game.js` 或 `main.js` 内联 | JS | 直接硬编码在逻辑中 |
| `fileconfig.json` | JSON | Laya引擎资源配置 |

### 代码内嵌数值
当配置未独立成文件时，需从代码中提取：

搜索关键词：
- 概率类：`probability`、`rate`、`chance`、`random`、`Math.random`、`weight`
- 掉落类：`drop`、`loot`、`reward`、`item`
- 合成类：`recipe`、`merge`、`combine`、`craft`、`formula`、`material`
- 经济类：`price`、`cost`、`gold`、`coin`、`diamond`、`gem`
- 属性类：`attack`、`defense`、`hp`、`speed`、`damage`

## 概率表逆向

### 常见概率系统实现

**权重随机**（最常见）：
```javascript
// 典型实现：按权重随机选择
var items = [
    {id: 1, weight: 50},  // 50%
    {id: 2, weight: 30},  // 30%
    {id: 3, weight: 15},  // 15%
    {id: 4, weight: 5}    // 5%
];
```

**保底机制（Pity System）**：
```javascript
// 搜索关键词：pity、guarantee、counter、累计
if (pullCount >= 90) {
    // 保底触发
    return guaranteedSSR();
}
```

**伪随机（PRD）**：
搜索 `seed`、`PRNG`、`mersenne`，部分游戏使用确定性随机数序列

### 提取方法
1. 搜索 `Math.random()` 调用点，向上追溯概率参数
2. 搜索权重数组/对象，提取所有权重配置
3. 搜索 `gacha`、`draw`、`pull`、`lottery` 等抽卡相关函数
4. 注意区分客户端概率展示（可能是假的）和实际服务端逻辑

## 掉落表逆向

### 掉落表结构
```javascript
// 典型掉落表结构
var dropTable = {
    "monster_001": [
        {itemId: "gold", min: 10, max: 50, rate: 1.0},
        {itemId: "gem", min: 1, max: 3, rate: 0.1},
        {itemId: "equip_rare", min: 1, max: 1, rate: 0.01}
    ]
};
```

### 提取步骤
1. 搜索 `drop`、`loot` 关键词定位掉落函数
2. 追溯掉落配置数据源（本地JSON / 代码内联 / 服务端下发）
3. 整理为结构化表格：怪物ID → 掉落物 → 数量范围 → 概率

## 合成配方逆向

### 常见数据结构
```javascript
// 合成配方
var recipes = [
    {inputs: [{id: "wood", count: 3}, {id: "stone", count: 2}], output: {id: "axe", count: 1}},
    {inputs: [{id: "iron", count: 5}], output: {id: "sword", count: 1}}
];

// 合并/升级（三消/合并类游戏）
var mergeRules = {
    "item_lv1": {mergeCount: 3, result: "item_lv2"},
    "item_lv2": {mergeCount: 3, result: "item_lv3"}
};
```

### 分析方法
1. 搜索合成相关函数入口
2. 提取输入-输出映射关系
3. 构建合成树/依赖图
4. 计算最终产物所需的基础材料总量

## 经济参数逆向

### 关键经济参数

| 参数类型 | 搜索关键词 | 分析要点 |
|---------|-----------|---------|
| 货币产出率 | `earn`、`income`、`produce` | 单位时间/单次操作的货币获取量 |
| 货币消耗点 | `cost`、`spend`、`consume` | 升级/购买/解锁的货币消耗 |
| 通货膨胀 | 等级成长曲线 | 每级所需经验/金币的增长倍率 |
| 付费货币汇率 | `exchange`、`convert` | 钻石→金币的兑换比例 |
| 体力系统 | `energy`、`stamina`、`heart` | 恢复速度、上限、消耗量 |

### 数值平衡性验证

1. **收支平衡计算**：
   - 统计所有货币产出点及产出速率
   - 统计所有货币消耗点及消耗量
   - 计算每日净收支（免费玩家 vs 付费玩家）

2. **进度曲线分析**：
   - 提取升级经验表，检查增长曲线（线性/指数/S型）
   - 计算到达各阶段所需时间
   - 识别"付费墙"（进度显著放缓的拐点）

3. **概率期望计算**：
   - 根据概率表计算获得特定物品的期望抽数
   - 结合货币产出率，计算免费玩家获得的预期时间
   - 与付费路径对比，评估"Pay-to-Win"程度

## 输出格式

数值逆向结果应整理为标准化表格：

```markdown
## 概率表
| 品质 | 权重 | 实际概率 | 保底机制 |
|------|------|---------|---------|

## 掉落表
| 关卡/怪物 | 掉落物 | 数量范围 | 概率 |
|-----------|--------|---------|------|

## 合成配方
| 产物 | 材料1 | 材料2 | ... | 基础材料总计 |
|------|-------|-------|-----|-------------|

## 经济参数
| 参数 | 值 | 备注 |
|------|-----|------|
```

来源：[看雪论坛-游戏逆向](https://bbs.kanxue.com/thread-11623.htm)，[网易游戏学堂-逆向分析工程师](https://api.academy.163.com/course/careerArticle?course=550)，[CSDN-网络游戏逆向分析](https://blog.csdn.net/qq_36301061/article/details/139536118) — 可信度：中-高
