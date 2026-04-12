# 维度4：玩法机制拆解

## 核心循环提取

### 微信小游戏典型核心循环

**超休闲类**（如三消、跑酷、益智）：
```
操作 → 即时反馈 → 关卡完成/失败 → 激励广告/分享续命 → 下一关
```

**放置/养成类**：
```
自动战斗获得金币 → 消耗金币提升属性 → 抽取装备/技能 → 强化数值 → 继续战斗
```

**社交竞技类**（如棋牌、对战）：
```
匹配 → 对局 → 结算（排名/段位变化）→ 分享炫耀 → 再来一局
```

来源：[36kr - 羊了个羊们怎样日入百万](https://36kr.com/p/1927160383998343)，[GameRes游资网](https://www.gameres.com/908245.html) — 可信度：高

### 核心循环提取方法
1. **体验法**：亲自玩3-5个完整Session，记录操作序列
2. **代码验证**：在反编译代码中搜索 `gameLoop`、`update`、`tick`、`onFrame`
3. **状态图绘制**：从状态机代码中提取所有状态和转换条件

## 关卡/难度曲线分析

### 数据来源
- **本地关卡配置**：搜索 `level`、`stage`、`difficulty` 相关JSON
- **服务端关卡**：抓包获取关卡接口返回的配置数据
- **动态难度**：搜索 `difficulty`、`adaptive`、`DDA` 相关逻辑

### 分析维度
| 维度 | 关注点 |
|------|--------|
| 难度梯度 | 前N关教学期→平稳期→难度跳跃点 |
| 新元素引入节奏 | 每隔几关引入新机制/新障碍 |
| 失败率设计 | 关键关卡的预期失败率（通常50-70%） |
| 复活/续命机制 | 广告复活的触发条件和次数限制 |
| 体力/次数限制 | 每日免费次数、体力恢复速度 |

## 付费点与广告位识别

### 广告类型（微信小游戏IAA模式）

| 广告类型 | 触发场景 | 用户价值 |
|---------|---------|---------|
| 资源广告 | 主动点击获取金币/钻石 | 直接奖励 |
| 玩法广告 | 看广告解锁关卡/续命/获得额外机会 | 延长体验 |
| 加成广告 | 看广告获得双倍收益/加速buff | 临时增益 |
| 插屏广告 | 关卡间/结算页自动弹出 | 无（强制曝光）|
| Banner广告 | 游戏底部常驻 | 无（被动展示）|

来源：[微信开放文档-激励政策](https://developers.weixin.qq.com/minigame/introduction/commercialization/)，[2024武汉微信小游戏公开课](https://blog.zengrong.net/post/2024-wuhan-wechat-microgame/) — 可信度：高（官方来源）

### 代码中广告位识别
搜索关键词：
- `wx.createRewardedVideoAd` — 激励视频广告
- `wx.createBannerAd` — Banner广告
- `wx.createInterstitialAd` — 插屏广告
- `adUnitId` — 广告位ID
- `onClose`（广告关闭回调中通常有奖励发放逻辑）

### 内购设计（IAP模式）
搜索关键词：`wx.requestMidasPayment`、`payment`、`purchase`、`buy`、`shop`

典型内购点：
- 去广告特权
- 皮肤/角色
- 战斗通行证
- 礼包（限时/首充）
- 宝石/钻石商店

## 社交裂变机制分析

### 常见裂变手段

| 机制 | 实现方式 | 代码特征 |
|------|---------|---------|
| 分享续命 | 分享到群/好友可获得额外机会 | `wx.shareAppMessage` |
| 排行榜 | 好友排名刺激攀比 | `wx.getOpenDataContext`、`wx.getUserCloudStorage` |
| 组队/助力 | 邀请好友帮忙通关 | 自定义分享参数+后端验证 |
| 群排名 | 分享到群后显示群内排名 | `shareTicket` 参数 |
| 复活求助 | 失败后邀请好友帮忙复活 | 分享+回调验证 |

### 关键API
```javascript
// 分享
wx.shareAppMessage({ title, imageUrl, query })

// 开放数据域（排行榜）
wx.getOpenDataContext()

// 订阅消息（召回）
wx.requestSubscribeMessage()
```

## Session结构分析

### 用户Session生命周期
```
冷启动 → wx.login() → 获取code → 换取session_key → 游戏初始化
热启动 → onShow() → 检查session有效性 → 恢复游戏状态
```

### 关键数据点
- `wx.login()` 返回的 `code`（一次性，用于换取openid）
- `session_key`（敏感，不应下发到客户端）
- 自定义登录态（游戏自己的token）
- 用户数据存储：`wx.setStorageSync` / 服务端存储

### 分析方法
1. 抓包观察登录流程的接口调用顺序
2. 在代码中搜索 `wx.login`、`session`、`token`、`auth`
3. 分析token生成和刷新机制
4. 识别用户数据的存储位置（本地 vs 服务端）
