# 维度3：Egret → Cocos Creator迁移

## 调研来源
- [Cocos Creator 3.8 Node and Component API](https://docs.cocos.com/creator/3.8/manual/en/scripting/access-node-component.html)
- [Cocos Creator Event System](https://docs.cocos.com/creator/3.8/manual/en/engine/event/event-node.html)
- [Cocos Creator DragonBones Support](https://docs.cocos.com/creator/3.8/manual/en/asset/dragonbones.html)
- [DragonBones ArmatureDisplay Component](https://docs.cocos.com/creator/3.8/manual/en/editor/components/dragonbones.html)

## 核心发现

### 总体对比
| 维度 | Egret | Cocos Creator 3.x |
|------|-------|-------------------|
| 语言 | TypeScript | TypeScript |
| 架构 | 显示列表（DisplayList） | 节点组件（Entity-Component） |
| UI系统 | EUI（EXML皮肤） | UI组件（场景编辑器） |
| 资源管理 | RES模块（JSON配置） | Asset Manager（编辑器管理） |
| 动画 | DragonBones / Tween | DragonBones / Spine / Animation |
| 渲染 | Canvas/WebGL | WebGL 2.0 / WebGPU |
| 编辑器 | EgretWing（已停更） | Cocos Creator编辑器 |

### 核心API映射表

#### 显示对象
| Egret | Cocos Creator 3.x | 说明 |
|-------|-------------------|------|
| `egret.DisplayObject` | `cc.Node` | 基础显示节点 |
| `egret.DisplayObjectContainer` | `cc.Node`（含children） | 容器节点 |
| `egret.Sprite` | `cc.Node + cc.Sprite` | 精灵（Cocos中拆分为节点+组件） |
| `egret.Bitmap` | `cc.Node + cc.Sprite` | 位图显示 |
| `egret.TextField` | `cc.Node + cc.Label` | 文本 |
| `egret.BitmapText` | `cc.Node + cc.Label(BMFont)` | 位图字体 |
| `egret.Shape` | `cc.Node + cc.Graphics` | 矢量图形 |
| `egret.Stage` | `cc.director.getScene()` | 舞台/场景根节点 |

#### 事件系统
| Egret | Cocos Creator 3.x | 说明 |
|-------|-------------------|------|
| `obj.addEventListener(type, handler, this)` | `node.on(type, handler, this)` | 添加事件监听 |
| `obj.removeEventListener(type, handler, this)` | `node.off(type, handler, this)` | 移除事件监听 |
| `egret.TouchEvent.TOUCH_TAP` | `cc.Node.EventType.TOUCH_END` | 点击事件 |
| `egret.TouchEvent.TOUCH_BEGIN` | `cc.Node.EventType.TOUCH_START` | 触摸开始 |
| `egret.TouchEvent.TOUCH_MOVE` | `cc.Node.EventType.TOUCH_MOVE` | 触摸移动 |
| `egret.Event.ADDED_TO_STAGE` | `onEnable() 生命周期` | 添加到舞台 |
| `egret.Event.REMOVED_FROM_STAGE` | `onDisable() 生命周期` | 从舞台移除 |
| `event.stageX / stageY` | `event.getUILocation()` | 全局坐标 |
| `event.localX / localY` | `event.getLocation()` | 本地坐标 |

#### EUI组件映射
| EUI组件 | Cocos Creator | 迁移注意事项 |
|---------|--------------|------------|
| `eui.Button` | `cc.Button + cc.Label` | Cocos的Button是组件不是节点，需挂载到Node上 |
| `eui.Image` | `cc.Sprite` | 直接替换 |
| `eui.Label` | `cc.Label` | 字体配置方式不同 |
| `eui.TextInput` | `cc.EditBox` | API差异较大 |
| `eui.ProgressBar` | `cc.ProgressBar` | 属性名不同（value → progress） |
| `eui.Scroller` | `cc.ScrollView` | 滚动方向配置方式不同 |
| `eui.List` | `cc.ScrollView + 自定义` | Cocos无内置List，需自行实现虚拟列表 |
| `eui.DataGroup` | 自定义实现 | 需用ScrollView+动态节点池模拟 |
| `eui.TabBar` | `cc.ToggleContainer` | 概念相近但API不同 |
| `eui.Group` | `cc.Node + cc.Layout` | Layout组件替代Group布局功能 |
| `eui.Rect` | `cc.Node + cc.Sprite(纯色)` | 无直接对应 |
| `eui.CheckBox` | `cc.Toggle` | API和事件不同 |

#### 资源管理
| Egret RES | Cocos Creator | 说明 |
|-----------|--------------|------|
| `RES.loadConfig()` | 编辑器自动管理 | Cocos在编辑器中管理资源 |
| `RES.loadGroup("preload")` | `resources.preload()` / Bundle | 资源预加载 |
| `RES.getRes("name")` | `resources.load()` | 获取资源（Cocos为异步） |
| `default.res.json` | 无需手动配置 | Cocos编辑器自动生成meta |
| 精灵图集(sheet) | SpriteAtlas | 格式不同需重新打包 |

### DragonBones兼容方案
Cocos Creator 3.x原生支持DragonBones v5.6.3及以下版本：
1. 将Egret项目中的DragonBones数据文件（_ske.json + _tex.json + _tex.png）直接复制到Cocos项目
2. 在Cocos场景中创建节点，添加dragonBones.ArmatureDisplay组件
3. 将骨骼数据拖入Dragon Asset属性，图集数据拖入Dragon Atlas Asset属性
4. 选择Armature和Animation即可播放

**注意**：如果Egret项目使用了.dbbin二进制格式，Cocos Creator同样支持。

### 关键差异与陷阱
1. **坐标系** — Egret原点左上角(Y轴向下)，Cocos Creator原点左下角(Y轴向上)，需要翻转Y坐标
2. **锚点** — Egret默认锚点(0,0)即左上角，Cocos默认(0.5,0.5)即中心
3. **显示列表 vs 组件系统** — Egret用继承（class extends egret.Sprite），Cocos用组合（Component挂到Node上）
4. **EXML → 场景编辑器** — EXML无法自动转换，需手动在Cocos编辑器中重建UI
5. **RES资源名 → 路径引用** — Egret用资源名字符串，Cocos用文件路径或UUID
6. **Tween差异** — Egret的egret.Tween与Cocos的cc.tween语法不同，但概念一致

### 迁移策略建议
1. **先迁移资源**：图片直接复制，精灵图集需重新用TexturePacker打包，DragonBones数据直接复用
2. **重建UI**：在Cocos编辑器中重建界面布局，无法从EXML自动导入
3. **改写逻辑**：TypeScript逻辑可大量复用，主要修改API调用部分
4. **适配坐标系**：全局查找坐标赋值代码，翻转Y值
