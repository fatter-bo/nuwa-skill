# 维度五：社交与社区

## 1. 好友系统

### 核心功能
- 好友添加/搜索/删除
- 在线状态显示（在线/游戏中/离线）
- 好友正在玩的牌桌/赛事信息
- 好友邀请入桌快捷功能
- 好友间私信

### PokerStars Home Games
PokerStars的Home Games功能是社交体验标杆：
- 创建/加入俱乐部（Club），设置名称和邀请码
- 俱乐部内公告板
- 俱乐部成员可组织私人牌局
- 支持视频聊天（WebRTC集成），模拟线下牌局社交感

来源: [PokerStars - Home Games Video Chat](https://www.pokerstars.com/poker/learn/news/video-chat-with-your-friends-while-playing-online-poker-with-latest-pokerstars-home-games-feature/)

## 2. 俱乐部系统（Club-Based Model）

### PPPoker/PokerBros/ClubGG模式
俱乐部模式是近年来最重要的产品创新之一：

**核心架构**：
- 玩家 → 加入俱乐部 → 在俱乐部内打牌
- 俱乐部主 → 创建/管理牌桌 → 设置规则和盲注 → 管理成员
- 代理系统 → 俱乐部主可指定代理，代理可邀请玩家、管理子团队
- 联盟机制 → 多个俱乐部联合，共享玩家池

来源: [Poker Game Developers - Club-Based Poker App](https://pokergamedevelopers.com/developing-club-based-poker-app-like-pppoker/)

**俱乐部管理后台功能**：
- 成员管理：加入/移除/角色分配（管理员/代理/普通成员）
- 牌桌管理：创建/关闭桌、设置盲注/买入/人数
- 筹码管理：给成员充值/回收筹码、交易记录
- 数据统计：俱乐部整体数据、成员排行、盈亏报表
- 反作弊：IP/设备限制、手牌审计

来源: [SOMUCHPOKER - How to Create Online Poker Club](https://somuchpoker.com/news/how-to-create-your-own-online-poker-club-on-pppoker-pokerbros-or-upoker)

### ClubGG（GGPoker旗下）
- 2021年由GGPoker母公司NSUS推出
- 继承GGPoker的游戏引擎和UI
- 支持Hold'em、Omaha、5-Card Omaha等多种玩法
- 俱乐部间可组成联盟扩大玩家池

来源: [BluffingMonkeys - ClubGG Poker App Guide](https://bluffingmonkeys.com/clubgg-poker-app-guide/)

## 3. 观战系统（Rail / Spectating）

### 常见实现方式
- **牌桌观战**：旁观者视角，看到公共牌和玩家动作，但看不到底牌
- **延迟直播**：为防止信息泄露，观战画面延迟15-60秒
- **Final Table观战**：大型赛事最终桌开放观战，增加赛事观赏性
- **好友观战**：一键跳转到好友所在牌桌观战

### 产品设计注意
- **底牌保护**：观战者绝对不能看到活跃玩家的底牌
- **延迟机制**：防止观战者向玩家传递信息（反作弊）
- **观战人数限制**：高流量牌桌需限制观战人数
- **互动功能**：观战者可发弹幕/表情，但不能影响牌局

## 4. 聊天系统

### 牌桌内聊天
- **文字聊天**：牌桌内的文字消息（通常有过滤器屏蔽不当言论）
- **表情/贴纸**：预设表情包（Nice Hand / Well Played / Bluff等）
- **互动道具**：向对手扔虚拟道具（啤酒/烟/番茄等）——社交货币化手段
- **语音聊天**：实时语音通话（Poker Face等App的核心卖点）
- **视频聊天**：最接近线下牌局体验（Poker Face、PokerStars Home Games）

来源: [Apple App Store - Poker Face](https://apps.apple.com/us/app/poker-face-texas-holdem-live/id1364570884)

### Poker Face 的视频社交创新
- 支持最多5人同时视频通话
- 视频画面直接嵌入牌桌界面
- 可以看到对手的面部表情——线下牌局的"读牌"体验线上化
- 核心价值：将社交/娱乐从"附加功能"提升为"核心体验"

来源: [Comunix - Poker Face](https://www.getcomunix.com/)

## 5. 直播与录像分享

### 手牌分享
- **文本格式分享**：复制手牌记录文字，粘贴到论坛/群聊
- **图片分享**：手牌回放截图，带关键节点标注
- **短视频/GIF分享**：自动生成手牌回放短视频
- **链接分享**：生成可在线查看的手牌回放链接

### 直播功能
- **平台内直播**：在App内直播打牌过程（需延迟处理）
- **流媒体集成**：OBS/Twitch/YouTube等外部直播工具支持
- **底牌显示切换**：直播者可选择向观众展示自己底牌
- **弹幕互动**：观众实时评论

### 社区功能
- **排行榜**：周/月/赛季排行
- **成就系统**：完成特定成就解锁徽章（如皇家同花顺、连胜记录）
- **论坛/讨论区**：平台内的策略讨论社区
- **教练功能**：教练可以观看学员的牌局并标注复盘

## 6. 社交功能货币化

### 虚拟道具与表情
- 付费表情包/贴纸
- 互动道具（扔道具给对手）
- 牌桌皮肤/背景
- 卡牌背面皮肤
- 头像框/称号

### VIP/会员体系
- 专属表情/道具
- 优先观战位
- 专属俱乐部功能
- 数据分析增强功能

来源: [Poker Game Developers - Club Monetization](https://pokergamedevelopers.com/developing-club-based-poker-app-like-pppoker/)
