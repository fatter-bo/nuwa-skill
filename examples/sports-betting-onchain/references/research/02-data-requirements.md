# 02-data-requirements: 体育数据需求详解

## 双维度比分

### 概念
同一场比赛有两个有效的"最终比分"：
- **正规时间比分**：90分钟（足球）、48分钟（NBA常规赛）
- **完整比分**：含加时、点球、overtime

### 为什么重要
- "90分钟赢家"市场 → 正规时间比分结算
- "晋级者"市场 → 完整比分结算
- UMA只接受单一"结果"——无法区分
- FastEvent原生支持双维度

## 分节比分

| 运动 | 分节 | 投注市场 |
|------|------|---------|
| 足球 | 上半场/下半场 | 半场比分、半场/全场 |
| 篮球 | Q1/Q2/Q3/Q4+半场 | 各节比分、半场线 |
| 网球 | 各盘/盘内局 | 盘比分、总盘数 |
| 棒球 | 各局 | 首5局、各局得分 |
| 冰球 | 各节（3节） | 各节比分 |

## 状态机

```
Scheduled → Live → HalfTime → SecondHalf → Finished → Settled → OnChain
                                                         ↓
                                                    Cancelled → Voided
```

每次状态转换触发的协议行为：
| 状态 | 触发行为 |
|------|---------|
| Scheduled | 创建市场、开放投注 |
| Live | 启动现场投注、赔率动态调整 |
| HalfTime | 暂停现场投注（部分）、可结算半场市场 |
| Finished | 关闭所有投注 |
| Settled | 结算所有市场 |
| OnChain | 链上确认、触发支付 |
| Cancelled | 退款所有投注 |

## matchId生成
`keccak256(abi.encodePacked(address(this), matchId, finalPrice))`
- 确定性哈希 → 跨协议唯一标识
- 签名者对每个子市场分别签名

## 数据字段清单

### 通用（所有运动）
```
matchId, sport, league, season, round
homeTeam { id, name, logo }
awayTeam { id, name, logo }
venue { name, city, country }
startTime (UTC timestamp)
status (enum)
scores {
  regularTime { home, away }
  fullTime { home, away }  // 含加时
  periods[] { period, home, away }
}
```

### 足球专属
```
goals[], cards[] { type, team, player, minute }
corners { home, away }
possession { home, away }  // 百分比
shotsOnTarget { home, away }
varDecisions[]
```

### 篮球专属
```
points { home, away }
rebounds { home, away }
assists { home, away }
fouls { home, away }
timeouts { home, away }
shotClock
```

### 网球专属
```
sets[] { home, away, tiebreak }
currentServer
aces { home, away }
doubleFaults { home, away }
```

## 上游数据源对比

| 供应商 | 覆盖 | 延迟 | 定价 |
|--------|------|------|------|
| Sportradar | 60+运动，官方授权 | 秒级 | $500-1000+/月起 |
| SportsDataIO | 北美主要联赛 | 秒级 | 企业谈判 |
| API-Football | 1200+联赛 | 15秒更新 | $19/月起 |
| Sportmonks | 2200+足球联赛 | 实时 | $29/月起 |
