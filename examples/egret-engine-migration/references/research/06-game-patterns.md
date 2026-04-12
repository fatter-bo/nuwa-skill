# 维度6：常见Egret游戏类型分析

## 调研来源
- [Egretia and HTML5 Games for Dummies - Medium](https://medium.com/@JChensays/egretia-and-html-5-games-for-dummies-3653ead3b5d2)
- [China's mobile gamers have sparked an HTML5 renaissance - TechNode](https://technode.com/2017/08/15/chinas-mobile-gamers-have-sparked-an-html5-renaissance/)
- [Egret EUI H5 event entry - Programmer All](https://www.programmerall.com/article/49731600180/)
- [Egret Slot Game - GitHub](https://github.com/t1123425/Slot-Game)
- [Egret Technology - LinkedIn](https://www.linkedin.com/company/egret-technology)

## 核心发现

### Egret市场定位
Egret Engine曾占据中国H5游戏市场70%以上份额。其核心优势是"即点即玩"的H5特性，天然适合社交平台传播（微信、微博、QQ、头条等）。游戏类型集中在轻量级、快速传播的品类。

### 三大典型应用领域

#### 1. 棋牌/博彩类游戏
**典型特征**：
- 多房间/多桌系统，WebSocket实时通信
- 丰富的UI界面（筹码动画、发牌动画、计时器）
- DragonBones用于角色表情和胜负动画
- 音效密集（发牌声、下注声、胜利音乐）

**技术栈模式**：
```
前端：Egret + EUI(界面) + DragonBones(动画) + WebSocket(通信)
后端：Node.js / Java / Go (游戏服务器)
协议：Protobuf / JSON over WebSocket
```

**迁移要点**：
- 通信层(WebSocket + Protobuf)可独立迁移，与引擎无关
- UI层需完全重建（大量定制EUI组件）
- 动画资源(DragonBones)可直接复用
- 核心游戏逻辑（牌局规则、下注逻辑）通常在后端，前端只做展示

#### 2. 休闲H5游戏
**典型类型**：消除、跑酷、射击、益智
**典型特征**：
- 单场景或少量场景
- 简单物理碰撞检测
- 排行榜/社交分享功能
- 广告SDK集成（激励视频/插屏）
- 关卡配置通过JSON数据驱动

**技术栈模式**：
```
核心：Egret + Tween(动画) + P2.js(物理，可选)
数据：JSON关卡配置 + 本地存储(egret.localStorage)
分发：微信小游戏 / H5页面
```

**迁移要点**：
- 逻辑通常较简单，TypeScript代码量小
- 关卡配置JSON可直接复用
- 核心玩法逻辑可几乎原样迁移（改API调用即可）
- 需替换平台SDK（微信小游戏API → 目标平台API）

#### 3. 营销互动小游戏
**典型场景**：电商大促、品牌推广、抽奖活动、签到打卡
**典型特征**：
- 生命周期短（通常1-4周活动期）
- 重UI设计、轻游戏逻辑
- 大量EUI组件和动画效果
- 与业务后端深度集成（奖品发放、用户积分）
- 分享裂变机制

**技术栈模式**：
```
前端：Egret + EUI(界面为主) + Tween(转场动画)
后端：RESTful API(活动逻辑、奖品管理)
集成：微信JS-SDK(分享) + 埋点SDK(数据统计)
```

**迁移要点**：
- 如果只是"仿做"，无需迁移——直接用新引擎参考设计重做
- 核心价值在视觉设计和交互方案，不在代码
- 图片和动画资源提取后可作为新项目的设计参考
- 活动逻辑通常绑定后端API，前端只做展示和调用

### 各游戏类型迁移难度评估
| 游戏类型 | 代码复杂度 | UI复杂度 | 通信复杂度 | 总迁移难度 |
|---------|-----------|---------|-----------|-----------|
| 棋牌类 | 中（前端逻辑不多，核心在后端） | 高（大量定制UI） | 高（实时通信） | 高 |
| 休闲H5 | 低-中 | 低-中 | 低（单机或简单排行） | 低-中 |
| 营销互动 | 低 | 中-高（重设计） | 低（REST API） | 低（建议重做而非迁移） |
| 中重度RPG | 高 | 高 | 高 | 很高（罕见，但存在） |

### 常见代码模式

#### MVC/MVVM架构
许多Egret项目使用PureMVC或自定义MVC框架：
```typescript
// 典型的Egret MVC项目结构
src/
├── controller/     // 命令(Command)
├── model/          // 数据模型(Proxy)
├── view/           // 视图(Mediator + Component)
│   ├── components/ // EUI组件
│   └── mediators/  // 视图中介
├── service/        // 网络服务
└── Main.ts
```

#### 场景管理模式
```typescript
// Egret常见的场景切换模式
class SceneManager {
    private static container: egret.DisplayObjectContainer;
    public static changeScene(scene: BaseScene): void {
        container.removeChildren();
        container.addChild(scene);
    }
}
```

#### 网络通信模式
```typescript
// WebSocket + Protobuf 典型模式
class NetworkManager {
    private socket: egret.WebSocket;
    public connect(url: string): void { ... }
    public send(cmd: number, data: any): void { ... }
    public onMessage(event: egret.ProgressEvent): void { ... }
}
```

### Egret项目识别特征清单
如何快速判断一个项目是否基于Egret引擎：
1. HTML入口文件引用了`egret.min.js`或`egret.web.min.js`
2. 存在`egretProperties.json`配置文件
3. 存在`resource/default.res.json`资源清单
4. 源码中大量使用`egret.`命名空间
5. 存在`.exml`文件（使用了EUI）
6. `libs/modules/`下有egret/eui/res等目录
