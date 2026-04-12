# TG 小游戏开发实践

> 来源: https://core.telegram.org/bots/api (Games), https://core.telegram.org/bots/webapps, 行业案例研究

## 1. 游戏引擎选型

### Phaser.js (推荐 2D)
- 轻量级 HTML5 游戏框架，天然适配 WebView
- 内置物理引擎（Arcade / Matter.js）
- 丰富的 Loader / Sprite / Animation / Tween 系统
- 社区活跃，Telegram 游戏案例最多
- 包体小（压缩后约 200-300KB），首屏加载快

### Cocos Creator (推荐高性能)
- 支持 2D/3D，WebGL 渲染性能优异
- 可视化编辑器，适合团队协作
- 导出 Web 构建直接部署为 Mini App
- 适合重度游戏（SLG、卡牌、棋牌）
- 包体较大，需做资源分包和懒加载

### 纯 HTML5 Canvas
- 零依赖，包体最小
- 适合极简玩法（点击、合成、Idle）
- 完全可控，无框架学习成本
- 参考 Notcoin 的点击挖矿玩法

### PixiJS
- 高性能 2D 渲染库
- 不含游戏逻辑框架，需自行搭建
- 适合需要精细渲染控制的场景

## 2. 游戏与 Bot 通信

### initData 验证（核心安全）

服务端验证流程:
```python
import hmac, hashlib

# 1. 从 initData 提取 hash 和其余参数
# 2. 将参数按 key 字母排序，拼成 key=value\n 格式
data_check_string = "\n".join(sorted(params))
# 3. 计算 secret_key
secret_key = hmac.new(b"WebAppData", bot_token.encode(), hashlib.sha256).digest()
# 4. 计算并比对 hash
calculated_hash = hmac.new(secret_key, data_check_string.encode(), hashlib.sha256).hexdigest()
assert calculated_hash == received_hash
```

### sendData（简单模式）
- Keyboard Button 启动时可用
- 将数据直接发回 Bot（不超过 4096 字节）
- 无需自建服务器
- 适合轻量交互（如表单提交、简单游戏结果）

### 通过自建后端通信（主流方案）
- Mini App 通过 HTTP/WebSocket 与自建后端通信
- 后端通过 Bot API 回推消息给用户
- 后端验证 initData 确认用户身份
- 适合实时游戏和复杂业务逻辑

## 3. 用户身份识别

- initData.user.id: Telegram 用户 ID（全局唯一）
- initData.user.username: 用户名（可选）
- initData.user.first_name / last_name: 显示名称
- initData.user.language_code: 语言偏好
- initData.user.is_premium: 是否 Premium 用户
- initData.user.photo_url: 头像 URL（仅部分启动模式可用）
- initData.start_param: 启动参数（用于追踪来源、邀请码等）

## 4. 排行榜系统

### Bot API 原生排行榜
- `setGameScore(user_id, score, chat_id, message_id)`: 设置分数
- `getGameHighScores(user_id, chat_id, message_id)`: 获取排行榜
- 分数只能递增（除非 force=true）
- 自动在游戏消息中显示排名

### 自建排行榜（推荐）
- 灵活定义排名维度（积分/等级/成就等）
- 支持好友排行、全服排行、赛季排行
- 结合 CloudStorage 做本地缓存
- Redis Sorted Set 是后端排行榜标准方案

## 5. 分享邀请机制

### 直接链接分享
```
https://t.me/botname/appname?startapp=invite_CODE
```
- 通过 start_param 传递邀请码
- 后端记录邀请关系，发放奖励

### switchInlineQuery
- 切换到内联模式，用户可向好友发送游戏卡片
- 支持选择目标聊天类型

### shareMessage
- 分享预制的 inline message
- 可包含游戏截图、分数、邀请链接

### shareToStory
- 分享到 Telegram Stories
- 支持自定义媒体和参数

## 6. 成功案例分析

### Notcoin (3000万+ 用户)
- 玩法: Tap-to-Earn（点击挖矿）
- 技术: 纯 HTML5 + WebSocket
- 增长: 极致的邀请奖励 + Squad 机制
- 最终实现代币上线交易所

### Hamster Kombat (亿级用户)
- 玩法: Idle + 经营模拟
- 技术: React + 自建后端
- 增长: 社交媒体病毒传播 + 每日签到 + 任务系统

### Catizen (Cat-themed)
- 玩法: 合成 + 养成
- 技术: Cocos Creator
- 增长: TON 生态深度整合 + NFT

## 7. 性能优化要点

- 首屏加载控制在 2s 以内（使用 loading 占位 + ready()）
- 资源按需加载，使用 WebP/AVIF 图片格式
- 合图 (Sprite Sheet) 减少 HTTP 请求
- 音效使用 Web Audio API，避免 autoplay 限制
- requestAnimationFrame 保持 60fps
- 低端设备降级策略（减少粒子/阴影/动画复杂度）
- WebSocket 心跳保活，断线自动重连
