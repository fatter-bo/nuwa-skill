# 02-other-sports: 篮球/网球/电竞盘口与结算规则

## 篮球

### 核心规则：加时赛算入全场盘口
- Moneyline/Spread/O/U：含OT
- 各节盘口（Q1-Q4）：不含OT
- 2nd Half：含OT
- Player Props：通常含OT

### 最低比赛时长
| 联赛 | 最低 | 规则 |
|------|------|------|
| NBA | 43分钟 | 不满=No Action全退 |
| NCAA | 35分钟 | 不满=No Action全退 |
| WNBA | 35分钟 | 不满=No Action全退 |
| FIBA | 各异 | 部分赛制无OT（平局=No Action） |

### 数据需求
```
score_q1_home/away
score_q2_home/away
score_q3_home/away
score_q4_home/away
score_ot_home/away (per OT period)
final_score (including all OT)
total_minutes_played
player_stats[] {player_id, points, rebounds, assists...}
```

## 网球

### 退赛处理——最复杂的结算问题

**Category 1（打完1盘即有效）**：
Bet365, Betfred, BetMGM, Coral, Ladbrokes, Paddy Power, Sky Bet, Betfair Exchange
- Match Winner：打完1盘→有效
- Set Betting/Handicap/O/U：退赛→全部作废

**Category 2a（必须打完）**：
888sport, William Hill, NetBet
- 所有盘口：没打完→全部作废

**Category 2b（打完必须，取消资格例外）**：
Bet365, Betway, BetVictor
- 退赛→全部作废
- 但取消资格（Disqualification）→按晋级者结算

**Walkover（弃权）**：
- 比赛未开始就退出→**所有博彩公司都全部作废**

### 数据需求
```
match_status (completed/retired/walkover/disqualified)
sets_completed
set_scores[] {set_number, games_a, games_b, tiebreak_score}
total_games_a, total_games_b
match_winner
retirement_point {set, game} (if applicable)
```

## 电竞

### 盘口类型
| 盘口 | 数据需求 |
|------|---------|
| Series Winner | seriesWinner, mapsWon |
| Map Winner | mapResults[] {mapNumber, winner} |
| Map Handicap | mapsWon per team |
| Kill O/U | kills per team per map |
| First Blood | firstBlood team per map |
| First Tower | firstTower team per map |

### 数据源挑战
- 无统一权威机构
- 依赖游戏API（Riot/Valve），各自Schema不同
- 独家数据协议限制访问
- 低级别赛事缺乏监控
- 延迟：游戏API可能滞后实际状态
- 假赛：低级别赛事因低薪资和弱监管更普遍

### 数据需求
```
match_id, team_a, team_b, tournament, game_title
series_format (BO1/BO3/BO5)
series_winner
map_results[] {map_number, map_name, winner, score}
kills_per_map[] {map, kills_a, kills_b}
first_blood_per_map[] {map, team}
first_tower_per_map[] {map, team}
match_status (completed/technical_forfeit/remake)
data_source (Riot/Valve/organizer)
```

## Live Betting术语

### 关键概念
- **In-Play/In-Running**：比赛进行中投注
- **Suspend**：临时关闭（进球/红牌/VAR时）
- **Resume**：重新开放（赔率调整后）
- **Ball in Play**：球在场上（vs 死球时间）
- **Courtsiding**：现场观赛利用1-5秒信息优势下注

### 延迟层级
| 来源 | 延迟 |
|------|------|
| 现场观众 | 0秒（courtsiding风险） |
| 专业数据源（Sportradar） | 1-3秒 |
| 电视转播 | 5-15秒 |
| 博彩公司接受延迟 | +3-8秒 |

### 为什么1秒很重要
1秒数据优势=可以投注已发生的事件。博彩公司用接受延迟+Sharp检测对抗。
FastEvent的<500ms延迟意味着可以成为in-play数据的一级提供者。

## 赔率格式与庄家利润

### 典型Vig/Overround
| 市场 | 典型Vig |
|------|---------|
| NFL/NBA Spread | 4-5% |
| 足球1X2 | 5-8% |
| 网球 | 4-6% |
| Props | 5-10% |
| Futures | 15-40% |
| Pinnacle（最低） | 2-3% |
