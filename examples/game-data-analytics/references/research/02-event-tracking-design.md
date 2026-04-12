# 研究维度二：数据埋点设计

## 1. 事件埋点命名规范

### 命名格式

业界推荐使用结构化命名格式：**category_object_action**（类别_对象_动作）

示例：
- `combat_boss_defeated` — 战斗系统中Boss被击败
- `shop_gem_pack_purchased` — 商店中宝石礼包被购买
- `tutorial_step_completed` — 教程步骤完成
- `match_round_finished` — 对局回合结束

**核心原则**：
- 使用snake_case统一命名，避免大小写混用导致同一事件被拆分为多条记录
- 避免缩写（除非团队内有明确共识），`purchase_amount_usd`优于`prc_amt`
- 单一事件+属性，而非创建变体事件（如用`boss_defeated`+`boss_id`属性，而非`boss_1_defeated`、`boss_2_defeated`）
- 大多数游戏追踪50-200种事件类型，但实际驱动决策的仅15-20个核心事件

**来源**：Countly "How to Design a Game Event Taxonomy"; Amplitude "Analytics Tracking Practices"; Avo Docs "Naming Conventions"
**可信度**：高 — 行业头部分析平台官方最佳实践

### Object-Action命名法

另一种简洁有力的命名框架：**[名词] + [过去式动词]**

示例：
- `Song Played`
- `Level Completed`
- `Item Purchased`
- `Match Started`

确保所有事件命名风格一致，选定一种后坚持使用。

## 2. 事件分层体系

### 三层埋点架构

| 层级 | 说明 | 示例 |
|------|------|------|
| **页面级（Screen/Page）** | 用户看到了什么 | `screen_viewed(screen_name=main_menu)` |
| **操作级（Action/Interaction）** | 用户做了什么 | `button_clicked(button_name=play_again)` |
| **业务级（Business/Outcome）** | 产生了什么业务结果 | `purchase_completed(amount=6.99, currency=USD)` |

### 按游戏系统分类

事件应按游戏系统分组，形成清晰的分析边界：

| 系统类别 | 核心事件 |
|---------|---------|
| **注册/登录** | `app_opened`, `user_registered`, `user_logged_in` |
| **新手引导** | `tutorial_started`, `tutorial_step_completed`, `tutorial_skipped`, `tutorial_completed` |
| **核心玩法** | `match_started`, `match_completed`, `level_started`, `level_completed`, `level_failed` |
| **经济系统** | `currency_earned`, `currency_spent`, `item_acquired`, `item_consumed` |
| **变现** | `purchase_initiated`, `purchase_completed`, `purchase_failed`, `ad_requested`, `ad_shown`, `ad_clicked`, `ad_rewarded` |
| **社交** | `friend_invited`, `friend_added`, `share_completed`, `chat_message_sent` |
| **留存机制** | `daily_login_reward_claimed`, `push_notification_received`, `push_notification_opened` |

**来源**：Countly Game Event Taxonomy Guide; AppsFlyer Gaming Measurement Framework
**可信度**：高 — 结合多个分析平台的最佳实践

## 3. 核心路径埋点清单

### 用户生命周期关键路径

```
注册 → 新手引导 → 首局 → 首充 → 日常留存 → 流失预警 → 回流
```

每个阶段的必备埋点：

**阶段一：注册**
- `app_first_open` — 首次打开（含属性：渠道来源、设备信息、OS版本）
- `registration_started` — 开始注册
- `registration_completed` — 注册完成（含属性：注册方式 guest/email/social）

**阶段二：新手引导**
- `tutorial_started` — 教程开始
- `tutorial_step_completed(step_id, step_name)` — 每步完成
- `tutorial_skipped(at_step)` — 教程跳过（记录跳过位置）
- `tutorial_completed(duration_sec)` — 教程完成（含耗时）

**阶段三：首局**
- `first_match_started` — 首局开始
- `first_match_completed(result, duration, score)` — 首局完成
- `core_loop_reached` — 到达核心循环

**阶段四：首充**
- `store_opened` — 商店页面打开
- `product_viewed(product_id, price)` — 商品浏览
- `purchase_initiated(product_id, price)` — 发起购买
- `purchase_completed(product_id, revenue, currency)` — 购买成功
- `purchase_failed(product_id, error_code)` — 购买失败

**阶段五：日常留存**
- `session_started(session_count, days_since_install)` — 会话开始
- `daily_reward_claimed(day_streak)` — 签到领奖
- `feature_used(feature_name)` — 功能使用

**阶段六：流失预警与回流**
- `churn_risk_triggered(risk_score, last_active_days)` — 流失预警触发
- `reengagement_push_sent(campaign_id)` — 召回推送发出
- `user_returned(days_inactive)` — 用户回归

### 棋牌游戏专属埋点

- `room_entered(room_type, room_level, stake_amount)` — 进入房间
- `match_dealt(hand_id, player_count, bot_count)` — 发牌
- `match_action(action_type, action_value)` — 对局操作（加注/弃牌/吃碰杠等）
- `match_result(result, win_amount, duration_sec, rounds)` — 对局结果
- `rank_changed(old_rank, new_rank, room_level)` — 等级变动
- `currency_balance_snapshot(currency_type, balance)` — 货币余额快照

## 4. 属性维度设计

### 通用属性维度

| 维度 | 属性名 | 说明 |
|------|--------|------|
| 用户维度 | `user_id`, `install_date`, `days_since_install`, `user_segment` | 用户身份与生命周期 |
| 设备维度 | `platform`, `os_version`, `device_model`, `screen_resolution` | 设备环境 |
| 地理维度 | `country`, `language`, `timezone` | 地域信息 |
| 渠道维度 | `utm_source`, `utm_medium`, `utm_campaign`, `media_source` | 获客渠道 |
| 会话维度 | `session_id`, `session_count`, `session_duration` | 会话上下文 |
| 经济维度 | `currency_type`, `amount`, `balance_after` | 经济系统 |
| 版本维度 | `app_version`, `ab_test_group` | 版本与实验 |

### 属性数据类型规范

- String：有限集合（如`room_type: "beginner|intermediate|advanced"`）
- Number：连续数值（如`score: 1500`，`duration_sec: 45.2`）
- Boolean：是否标记（如`is_first_purchase: true`）
- Timestamp：时间戳（ISO 8601格式）

## 5. 服务端 vs 客户端埋点选型

| 维度 | 客户端埋点 | 服务端埋点 |
|------|-----------|-----------|
| **适用场景** | UI交互、页面浏览、用户行为 | 交易、经济系统、匹配、排行 |
| **数据可靠性** | 可能丢失（网络断开/App崩溃） | 高可靠（服务器控制） |
| **防作弊** | 弱（可被篡改） | 强（服务器验证） |
| **实时性** | 即时采集，延迟上报 | 即时记录 |
| **成本** | 低（SDK集成） | 高（需后端开发） |
| **典型事件** | `screen_viewed`, `button_clicked`, `ad_shown` | `purchase_completed`, `match_result`, `currency_earned` |

**最佳实践**：涉及货币/积分/排名等核心经济数据必须用服务端埋点；UI交互和行为路径可用客户端埋点；关键转化事件（如首充）双端埋点做交叉验证。

**来源**：Firebase Analytics Documentation; Amplitude Data Planning Playbook; AppsFlyer Gaming Measurement Guide
**可信度**：高 — 多平台一手文档综合

## 6. 埋点文档模板

### 事件定义表模板

| 字段 | 说明 |
|------|------|
| Event Name | 事件名称（遵循命名规范） |
| Category | 所属系统类别 |
| Description | 事件触发的业务含义 |
| Trigger Condition | 具体触发时机 |
| Platform | Client/Server/Both |
| Parameters | 属性列表（名称、类型、必填/选填、取值范围） |
| Owner | 负责开发者 |
| Priority | P0/P1/P2 |
| Status | Draft/Review/Implemented/Verified |

### 埋点文档管理原则

1. 埋点文档是分析的"宪法"，新事件实现前必须先在文档中定义
2. 定期审计：每季度清理不再使用的事件，避免数据冗余
3. 版本管理：每次埋点变更记录changelog，含变更原因和影响范围
4. 验证机制：每次发版前用debug模式验证核心埋点数据正确性

**来源**：Amplitude "Plan Your Taxonomy"; Krista Seiden "Events: Best Practices for Hierarchies and Naming Conventions"
**可信度**：高 — 行业最佳实践
