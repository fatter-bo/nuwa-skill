# 04 各玩法的规则数据化——可编程结构化定义

## 核心思想

将棋牌规则从自然语言转化为可编程的形式化定义，使得：
1. 规则可以被代码精确执行
2. 规则变体可以通过参数调整生成
3. AI可以基于形式化规则进行博弈和学习

---

## 通用游戏状态模型（参考OpenSpiel）

### 游戏定义六要素

```
GameDefinition {
  state_space:      状态空间定义（所有合法游戏状态的集合）
  action_space:     动作空间定义（所有合法操作的集合）
  transition_fn:    状态转移函数 (state, action) → new_state
  observation_fn:   观察函数 (state, player) → observable_state
  reward_fn:        奖励/计分函数 (state) → scores[]
  terminal_fn:      终局判定函数 (state) → bool
}
```

### 游戏分类维度

| 维度 | 选项 | 代表游戏 |
|------|------|---------|
| 信息 | 完全信息 / 不完全信息 | 象棋 / 扑克 |
| 随机性 | 确定性 / 随机性 | 围棋 / 麻将 |
| 玩家数 | 2人 / 多人 | 象棋 / 斗地主 |
| 合作性 | 零和 / 合作 / 混合 | 象棋 / 升级(组队) |
| 动作类型 | 顺序 / 同时 | 扑克 / 石头剪刀布 |

---

## 德州扑克的结构化定义

```yaml
game:
  name: "Texas Hold'em No-Limit"
  type: sequential, stochastic, imperfect_information
  players: [2, 10]  # 2到10人

state:
  deck: Card[52]           # 牌堆
  community: Card[0..5]    # 公共牌
  hands: Map<Player, Card[2]>  # 底牌
  pot: int                 # 底池
  bets: Map<Player, int>   # 当前轮下注
  stacks: Map<Player, int> # 筹码
  phase: enum { PREFLOP, FLOP, TURN, RIVER, SHOWDOWN }
  active_players: Set<Player>
  dealer_pos: int          # 庄家位置

action_space:
  FOLD: {}                 # 弃牌
  CHECK: {}                # 过牌（当前无需跟注时）
  CALL: {}                 # 跟注
  RAISE: { amount: int }   # 加注（NL: min_raise..stack）
  ALL_IN: {}               # 全下

transition_rules:
  dealing:
    PREFLOP: deal 2 cards to each player
    FLOP:    reveal 3 community cards
    TURN:    reveal 1 community card
    RIVER:   reveal 1 community card
  
  betting_round:
    - starts left of dealer (preflop: left of BB)
    - each player: FOLD | CHECK | CALL | RAISE | ALL_IN
    - round ends when all active players have equal bets
    - minimum raise = previous raise size
  
  phase_transition:
    betting_complete AND active_players > 1 → next_phase
    betting_complete AND active_players == 1 → winner (last standing)
    RIVER betting complete → SHOWDOWN

hand_ranking: # 从高到低
  - ROYAL_FLUSH:     [A, K, Q, J, 10] same suit
  - STRAIGHT_FLUSH:  5 consecutive same suit
  - FOUR_OF_A_KIND:  4 same rank + 1 kicker
  - FULL_HOUSE:      3 same rank + 2 same rank
  - FLUSH:           5 same suit
  - STRAIGHT:        5 consecutive
  - THREE_OF_A_KIND: 3 same rank + 2 kickers
  - TWO_PAIR:        2×2 same rank + 1 kicker
  - ONE_PAIR:        2 same rank + 3 kickers
  - HIGH_CARD:       5 highest cards

terminal_condition:
  - one player remaining (all others folded)
  - showdown after river betting
  - all players all-in (run out remaining community cards)

scoring:
  winner takes pot (split if tied)
  side_pots when players are all-in with different stacks
```

---

## 斗地主的结构化定义

```yaml
game:
  name: "Dou Di Zhu (Fight the Landlord)"
  type: sequential, stochastic, imperfect_information
  players: 3

state:
  deck: Card[54]               # 含大小王
  hands: Map<Player, Card[]>   # 手牌（地主20张，农民各17张）
  kitty: Card[3]               # 底牌
  landlord: Player             # 地主
  current_play: Combination    # 当前台面上的出牌
  pass_count: int              # 连续过牌数
  bomb_count: int              # 已出炸弹/火箭数
  bid_value: int               # 叫分值（1/2/3）

card_ranking:  # 单张排名
  [RED_JOKER, BLACK_JOKER, 2, A, K, Q, J, 10, 9, 8, 7, 6, 5, 4, 3]

combination_types:
  SINGLE:          { cards: 1 }
  PAIR:            { cards: 2, constraint: same_rank }
  TRIPLE:          { cards: 3, constraint: same_rank }
  TRIPLE_PLUS_1:   { cards: 4, constraint: triple + any_single }
  TRIPLE_PLUS_2:   { cards: 5, constraint: triple + any_pair }
  SEQUENCE:        { cards: [5..12], constraint: consecutive_ranks, exclude: [2, JOKER] }
  PAIR_SEQUENCE:   { cards: [6..], constraint: consecutive_pairs, min_pairs: 3 }
  AIRPLANE:        { cards: [6..], constraint: consecutive_triples, min_triples: 2 }
  AIRPLANE_WINGS:  { constraint: airplane + singles/pairs per triple }
  BOMB:            { cards: 4, constraint: same_rank, special: beats_all_non_bomb }
  ROCKET:          { cards: 2, constraint: [RED_JOKER, BLACK_JOKER], special: beats_all }

transition_rules:
  bidding_phase:
    - rotate: each player bids 0/1/2/3
    - highest bidder becomes landlord
    - landlord receives kitty (3 cards)
    - bid_value = winning bid
  
  play_phase:
    - landlord leads first
    - next player must: PASS or play same_type with higher_rank
    - exception: BOMB beats any non-bomb; ROCKET beats all
    - when 2 consecutive passes: leader starts new round
  
  bomb_multiplier:
    each BOMB or ROCKET played: total_multiplier *= 2

terminal_condition:
  any player empties hand

scoring:
  base = bid_value
  multiplier = 2^bomb_count
  spring_bonus: if opponents played 0 cards → multiplier *= 2
  landlord_wins: each opponent pays base × multiplier
  opponent_wins: landlord pays each opponent base × multiplier
```

---

## 麻将（四川血战）的结构化定义

```yaml
game:
  name: "Sichuan Mahjong (Bloody)"
  type: sequential, stochastic, imperfect_information
  players: 4

state:
  wall: Tile[108]                    # 牌墙（万/条/筒各36）
  hands: Map<Player, Tile[]>         # 手牌
  discards: Map<Player, Tile[]>      # 弃牌区
  melds: Map<Player, Meld[]>         # 明牌组合
  missing_suit: Map<Player, Suit>    # 定缺花色
  finished: Set<Player>              # 已胡玩家
  current_turn: Player

tile_types:
  suits: [WAN, TIAO, TONG]  # 万、条、筒
  ranks: [1..9]
  # 无字牌、无花牌

action_space:
  DRAW:    从牌墙摸牌
  DISCARD: 打出一张牌
  CHI:     不可（四川麻将禁止吃）
  PENG:    碰（需2张相同+他人打出1张）
  GANG:    杠（明杠/暗杠/加杠）
  HU:      胡牌（自摸或点炮）
  PASS:    放弃操作

winning_condition:
  # 标准胡牌形式
  hand = N×{顺子或刻子} + 1×{雀头}  # N=4(标准)
  # 额外条件
  - 必须已完成"缺一门"（missing_suit的牌全部打出）
  - 七对也可胡

special_rules:
  blood_battle:
    - 一人胡牌后游戏继续
    - 已胡者不再参与
    - 直到剩余<=1人未胡或牌墙摸完
  
  gang_settlement:  # 即时结算
    MING_GANG: 放杠者付 bottom×1 给杠者
    AN_GANG:   其余每人付 bottom×2 给杠者
    JIA_GANG:  其余每人付 bottom×1 给杠者
  
  flow_penalty:  # 流局惩罚
    未听牌者赔付听牌者
    未缺一门者("花猪")赔付所有人

scoring:
  base = 底注
  fan_multiplier = 2^(番数)
  self_drawn: 其余未胡者各付 base × fan_multiplier
  discard_win: 放炮者单独付 base × fan_multiplier
```

---

## 国际象棋的结构化定义

```yaml
game:
  name: "Chess"
  type: sequential, deterministic, perfect_information
  players: 2

state:
  board: Piece[8][8]
  turn: enum { WHITE, BLACK }
  castling_rights: { WK: bool, WQ: bool, BK: bool, BQ: bool }
  en_passant_target: Square | null
  halfmove_clock: int    # 50步规则计数器
  fullmove_number: int
  # FEN notation完整编码一个状态

action_space:
  MOVE: { from: Square, to: Square }
  CASTLE: { side: KINGSIDE | QUEENSIDE }
  PROMOTE: { from: Square, to: Square, piece: QUEEN|ROOK|BISHOP|KNIGHT }
  EN_PASSANT: { from: Square, to: Square }

legal_move_generation:
  for each piece of current player:
    generate pseudo-legal moves by piece type
    filter: move must not leave own king in check

terminal_conditions:
  CHECKMATE:    current player's king in check AND no legal moves → opponent wins
  STALEMATE:    current player NOT in check AND no legal moves → draw
  FIFTY_MOVE:   halfmove_clock >= 100 → draw (by claim)
  THREEFOLD:    same position occurred 3 times → draw (by claim)
  INSUFFICIENT: K vs K, K+B vs K, K+N vs K → draw
  AGREEMENT:    both players agree → draw
  TIMEOUT:      clock expires → opponent wins (if sufficient material)
```

---

## 通用参数化框架

通过参数化，同一引擎可支持多种变体：

```yaml
# 麻将参数化模板
mahjong_variant:
  tiles:
    suits: [WAN, TIAO, TONG]          # 可选去掉某些花色
    honors: [WIND, DRAGON] | []        # 四川无字牌
    flowers: [FLOWER, SEASON] | []     # 可选花牌
  
  rules:
    chi_allowed: true | false          # 四川禁吃
    min_fan: 0 | 1 | 8                 # 番缚
    blood_battle: true | false         # 血战到底
    missing_suit: true | false         # 定缺
    wildcard: null | RED_ZHONG | ...   # 赖子
    scoring_system: NATIONAL | CANTON | RIICHI | SICHUAN
  
  scoring:
    method: FAN_TABLE | HAN_FU | MULTIPLIER
    base_payment: int
    self_draw_bonus: DOUBLE | EXTRA_FAN | NONE
```

---

## 来源
- OpenSpiel (Google DeepMind): https://openspiel.readthedocs.io/
- Board Game Arena State Machine: https://en.boardgamearena.com/doc/Your_game_state_machine
- Game Programming Patterns (State Pattern): https://gameprogrammingpatterns.com/state.html
- Game Architecture for Card Game Model: https://bennycheung.github.io/game-architecture-card-ai-1
