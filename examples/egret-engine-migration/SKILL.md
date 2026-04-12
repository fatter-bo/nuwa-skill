---
name: egret-engine-migration
description: |
  Egret引擎迁移专家。系统掌握白鹭引擎（Egret）项目的分析和迁移方法论，
  能将Egret项目迁移到Cocos Creator或Unity，同时也能从Egret项目中提取设计思路和资源用于高仿重做。
  基于Egret技术栈全景、项目结构分析、跨引擎API映射、资源提取复用、游戏类型模式等
  6个维度深度调研的系统性迁移工具。
  当用户说「egret迁移」「白鹭引擎」「egret分析」「egret转cocos」「egret项目」「eui迁移」时使用。
---

# Egret引擎迁移专家

## 使用说明

**这个Skill不扮演角色，是一个工具。** 用户提供Egret项目或提出迁移需求时，按照下面的工作流执行。

---

## 迁移路线决策树

**在开始迁移之前，先确定迁移路线。**

```
用户需求是什么？
├─ 完整迁移到另一个引擎
│   ├─ 目标是Web/H5/小游戏平台 → Egret → Cocos Creator（推荐3.x）
│   ├─ 目标是原生App/高性能 → Egret → Unity
│   └─ 不确定 → 评估项目复杂度后建议
├─ 提取资源用于新项目
│   └─ 走资源提取流程
└─ 分析现有Egret项目（理解结构/评估工作量）
    └─ 走项目分析流程
```

---

## 执行工作流

```
Phase 0: 项目识别与评估（先确认是Egret项目）
  → 检查egretProperties.json确认引擎版本
  → 扫描模块依赖（EUI? DragonBones? Socket?）
  → 评估项目规模（源文件数、资源量）
  → 输出：项目概况报告

Phase 1: 资源盘点与提取
  → 解析default.res.json获取完整资源清单
  → 分类标记：可直接复用 / 需转换 / 需重建
  → 提取可复用资源

Phase 2: 代码分析与映射
  → 识别使用的Egret API和模式
  → 映射到目标引擎API
  → 标记无法直接映射的部分

Phase 3: 逐模块迁移/重建
  → 按优先级执行迁移
  → 验证功能一致性
```

---

## 框架一：Egret项目快速识别清单

**确认一个项目是否基于Egret引擎，检查以下标志：**

| # | 识别标志 | 位置 |
|---|---------|------|
| 1 | 存在`egretProperties.json` | 项目根目录 |
| 2 | HTML入口引用`egret.min.js`或`egret.web.min.js` | index.html |
| 3 | 存在`resource/default.res.json` | resource/ |
| 4 | 源码使用`egret.`命名空间 | src/*.ts |
| 5 | 存在`.exml`皮肤文件 | resource/eui_skins/ |
| 6 | `libs/modules/`下有egret/eui/res等目录 | libs/ |

**确认后从`egretProperties.json`提取关键信息：**
- `engineVersion` — 引擎版本（决定API兼容性）
- `modules` — 启用的模块列表（决定迁移范围）
- `eui.themes` — 主题配置路径
- `target.current` — 目标平台（web/wxgame等）

---

## 框架二：Egret项目标准结构地图

```
project-root/
├── egretProperties.json      ← 【必读】引擎版本+模块配置
├── tsconfig.json             ← TypeScript编译配置
├── index.html                ← 入口页面
├── src/                      ← TypeScript源码
│   ├── Main.ts               ← 入口类（加载流程起点）
│   └── ...                   ← 业务逻辑
├── resource/                 ← 【核心】所有资源
│   ├── default.res.json      ← 资源清单（name→url映射+分组）
│   ├── default.thm.json      ← 主题配置（组件→皮肤映射）
│   ├── assets/               ← 图片、音频、动画数据
│   ├── config/               ← 游戏配置JSON
│   └── eui_skins/            ← EXML皮肤文件
├── libs/modules/             ← 引擎库文件
└── bin-debug/                ← 编译输出
```

---

## 框架三：Egret → Cocos Creator API映射表

### 显示对象映射

| Egret | Cocos Creator 3.x | 说明 |
|-------|-------------------|------|
| `egret.DisplayObject` | `cc.Node` | 基础节点 |
| `egret.Sprite` | `cc.Node + cc.Sprite` | Cocos拆分为节点+组件 |
| `egret.Bitmap` | `cc.Node + cc.Sprite` | 位图显示 |
| `egret.TextField` | `cc.Node + cc.Label` | 文本 |
| `egret.BitmapText` | `cc.Node + cc.Label(BMFont)` | 位图字体 |
| `egret.Shape` | `cc.Node + cc.Graphics` | 矢量绘制 |
| `egret.Stage` | `cc.director.getScene()` | 舞台/场景根 |

### 事件系统映射

| Egret | Cocos Creator 3.x |
|-------|-------------------|
| `addEventListener(type, handler, this)` | `node.on(type, handler, this)` |
| `removeEventListener(type, handler, this)` | `node.off(type, handler, this)` |
| `TouchEvent.TOUCH_TAP` | `Node.EventType.TOUCH_END` |
| `TouchEvent.TOUCH_BEGIN` | `Node.EventType.TOUCH_START` |
| `Event.ADDED_TO_STAGE` | `onEnable()` 生命周期回调 |
| `Event.ENTER_FRAME` | `update(dt)` 生命周期回调 |
| `event.stageX/stageY` | `event.getUILocation()` |

### EUI → Cocos UI组件映射

| EUI组件 | Cocos Creator | 迁移注意 |
|---------|--------------|---------|
| `eui.Button` | `cc.Button + cc.Label` | Button是组件非节点 |
| `eui.Image` | `cc.Sprite` | 直接替换 |
| `eui.Label` | `cc.Label` | 字体配置方式不同 |
| `eui.TextInput` | `cc.EditBox` | API差异大 |
| `eui.ProgressBar` | `cc.ProgressBar` | value → progress |
| `eui.Scroller` | `cc.ScrollView` | 配置方式不同 |
| `eui.List` | `ScrollView + 自定义` | **无内置List，需实现虚拟列表** |
| `eui.DataGroup` | 自定义实现 | 需用节点池模拟 |
| `eui.Group + Layout` | `cc.Node + cc.Layout` | Layout组件替代 |
| `eui.CheckBox` | `cc.Toggle` | API和事件不同 |
| `eui.TabBar` | `cc.ToggleContainer` | 概念相近 |

### 资源管理映射

| Egret RES | Cocos Creator |
|-----------|--------------|
| `RES.loadConfig()` | 编辑器自动管理 |
| `RES.loadGroup("preload")` | `resources.preload()` / Bundle |
| `RES.getRes("name")` | `resources.load(path, callback)` |
| `default.res.json` | 无需手动配置 |
| 精灵图集(sheet) | SpriteAtlas（需重新打包） |

---

## 框架四：Egret → Unity迁移对照

### TypeScript → C#类型映射

| TypeScript | C# | 注意 |
|-----------|-----|------|
| `number` | `int`/`float`/`double` | 根据用途选精度 |
| `string` | `string` | 一致 |
| `boolean` | `bool` | 关键字不同 |
| `Array<T>` | `List<T>` | API方法名不同 |
| `Map<K,V>` | `Dictionary<K,V>` | API方法名不同 |
| `Promise<T>` | `Task<T>` / async-await | 异步模式类似 |
| `() => void` | `Action` / `Func<T>` | 委托类型替代 |

### EUI → UGUI映射

| EUI | UGUI | 说明 |
|-----|------|------|
| `eui.Button` | `Button + Text/TMP` | 需配合Image和Text子对象 |
| `eui.Label` | `TextMeshPro` | 建议直接用TMP |
| `eui.Image` | `Image` / `RawImage` | UI用Image |
| `eui.ProgressBar` | `Slider`或`Image.fillAmount` | 无原生ProgressBar |
| `eui.Scroller` | `ScrollRect` | 需配合Mask+Content |
| `eui.List` | `ScrollRect + 对象池` | 需自行实现虚拟化 |
| EXML皮肤 | Prefab | Prefab替代EXML |
| 数据绑定 | 无内置 | 需自实现或用MVVM框架 |

### 资源管理映射（RES → Addressables）

| Egret RES | Unity Addressables |
|-----------|-------------------|
| `default.res.json` | Addressables Groups |
| `RES.loadGroup()` | `Addressables.LoadAssetsAsync<T>(label)` |
| `RES.getRes("name")` | `Addressables.LoadAssetAsync<T>(key)` |
| 精灵图集 | SpriteAtlas |

---

## 框架五：资源提取与复用矩阵

| 资源类型 | 文件格式 | 可直接复用 | 需转换 | 需重建 |
|---------|---------|-----------|--------|--------|
| 普通图片 | .png/.jpg | **直接复制** | | |
| 音频文件 | .mp3/.ogg | **直接复制** | | |
| DragonBones动画 | _ske.json + _tex.json + _tex.png | **直接复制** | | |
| 位图字体 | .fnt + .png | **直接复制** | | |
| JSON配置 | .json | **直接复制** | | |
| 精灵图集 | .json + .png | | **重新打包** | |
| 粒子效果 | .json | | | **手动重建** |
| EXML皮肤 | .exml | | | **手动重建UI** |

### DragonBones跨引擎复用步骤

1. 从Egret项目的`resource/assets/`找到三元组文件：`xxx_ske.json` + `xxx_tex.json` + `xxx_tex.png`
2. **Cocos Creator**：直接复制到Cocos项目assets目录，添加ArmatureDisplay组件，拖入Dragon Asset和Dragon Atlas Asset
3. **Unity**：导入DragonBones Unity Runtime包，同样使用三元组文件

**注意**：Cocos Creator支持DragonBones v5.6.3及以下版本。如果Egret项目使用了更高版本的DragonBones，需确认兼容性。

### 精灵图集拆分方法

Egret精灵图集由描述JSON和合并PNG组成。拆分思路：
1. 解析JSON中的frames字段（包含每个子图的x/y/w/h坐标）
2. 用图像处理库（如Python PIL）按坐标裁剪出独立图片
3. 或用TexturePacker导入后重新导出为目标引擎格式

---

## 框架六：迁移难度评估模型

### 按游戏类型评估

| 游戏类型 | 代码复杂度 | UI复杂度 | 网络复杂度 | 推荐策略 |
|---------|-----------|---------|-----------|---------|
| 棋牌类 | 中 | 高 | 高(WebSocket) | 逐模块迁移，通信层独立处理 |
| 休闲H5 | 低-中 | 低-中 | 低 | 直接迁移，工作量可控 |
| 营销互动 | 低 | 中-高 | 低 | **建议重做而非迁移** |
| 中重度RPG | 高 | 高 | 高 | 分阶段迁移，预期工期长 |

### 按模块评估工作量

| 模块 | 可自动化程度 | 人工工作量 | 优先级 |
|------|------------|-----------|--------|
| 图片/音频资源 | 高（直接复制） | 低 | 先做 |
| DragonBones动画 | 高（直接复用） | 低 | 先做 |
| 游戏配置数据 | 高（JSON通用） | 低 | 先做 |
| 游戏核心逻辑 | 中（改API调用） | 中 | 次做 |
| UI界面 | 低（需手动重建） | 高 | 后做 |
| 网络通信层 | 中（协议可复用） | 中 | 独立处理 |
| 平台SDK集成 | 低（完全不同） | 中-高 | 最后做 |

---

## 框架七：关键迁移陷阱清单

### 坐标系差异（最常见的Bug来源）

| 引擎 | 原点位置 | Y轴方向 | 默认锚点 |
|------|---------|---------|---------|
| Egret | 左上角 | 向下 | (0, 0) 左上 |
| Cocos Creator | 左下角 | 向上 | (0.5, 0.5) 中心 |
| Unity | 中心（世界坐标） | 向上 | (0.5, 0.5) 中心 |

**处理方法**：全局搜索坐标赋值代码（`.x =` / `.y =`），Y值取反或重新计算。

### 架构模式差异

| 概念 | Egret | Cocos Creator / Unity |
|------|-------|---------------------|
| 代码组织 | 类继承（extends egret.Sprite） | 组件组合（Component挂到Node上） |
| 帧更新 | `addEventListener(ENTER_FRAME, fn)` | `update(dt)` 生命周期 |
| 显示管理 | `addChild()` / `removeChild()` | 节点树 + `active`属性控制 |
| 资源引用 | 字符串名称（"bg_jpg"） | 文件路径或UUID引用 |

### 常见遗漏项
1. **EUI数据绑定** — Egret的EXML支持`{dataField}`绑定语法，目标引擎均无等价物，需改为手动赋值
2. **皮肤状态管理** — EXML的`states="normal,disabled"`在目标引擎中需用代码实现状态切换
3. **RES资源名引用** — Egret用`RES.getRes("name")`的字符串名，迁移后需改为路径引用
4. **egret.setTimeout/setInterval** — 需替换为目标引擎的定时器方案
5. **egret.localStorage** — 需替换为目标引擎的本地存储API

---

## 决策启发式

1. **营销活动类项目不值得迁移** — 生命周期短、价值在设计不在代码，直接用新引擎重做，提取资源作参考即可
2. **先迁资源再迁逻辑** — 资源提取成本低且可验证，先做完资源层再动代码
3. **DragonBones是最大的迁移加速器** — 骨骼动画数据跨引擎通用，提取后可直接用于Cocos Creator和Unity
4. **EUI是最大的迁移成本** — EXML无法自动转换，复杂界面需在目标引擎编辑器中完全重建
5. **List/DataGroup迁移要特别注意** — Egret的eui.List功能完善，Cocos Creator和Unity都没有等价的内置组件，需自行实现虚拟列表
6. **坐标系差异是Bug温床** — Egret的Y轴向下，Cocos/Unity的Y轴向上，全局搜索替换是必要步骤
7. **TypeScript到TypeScript优先选Cocos** — 如果目标平台允许，Egret→Cocos Creator保持同一语言，迁移成本最低
8. **通信层可独立迁移** — WebSocket+Protobuf的网络层与渲染引擎无关，可以最小改动复用
9. **5.0+的WebAssembly项目注意版本锁定** — Egret 5.0引入的wasm渲染核心在迁移时可忽略，因为目标引擎有自己的渲染方案
10. **项目分析先读三个文件** — `egretProperties.json`（版本和模块）、`default.res.json`（资源清单）、`Main.ts`（入口逻辑），可快速理解项目全貌

---

## 输出模板

### 项目分析报告模板

```
== Egret项目分析报告 ==

引擎版本: [从egretProperties.json读取]
启用模块: [egret, eui, res, tween, dragonBones, ...]
目标平台: [web / wxgame / ...]

资源统计:
  - 图片资源: X 个
  - 精灵图集: X 个
  - DragonBones动画: X 组
  - 音频文件: X 个
  - EXML皮肤: X 个
  - 其他配置: X 个

代码规模:
  - TypeScript文件: X 个
  - 总代码行数: ~X 行
  - 使用EUI: 是/否
  - 使用DragonBones: 是/否
  - 使用WebSocket: 是/否

项目类型判断: [棋牌 / 休闲 / 营销互动 / RPG / 其他]
推荐迁移路线: [Cocos Creator / Unity / 建议重做]
预估工作量: [X 人天]
```

### 迁移方案模板

```
== 迁移方案 ==

源项目: [Egret版本]
目标引擎: [Cocos Creator 3.x / Unity 202x]

Phase 1 - 资源迁移 (预计X天)
  可直接复用: [列表]
  需转换: [列表]
  需重建: [列表]

Phase 2 - 核心逻辑迁移 (预计X天)
  [按模块列出迁移计划]

Phase 3 - UI重建 (预计X天)
  [按界面列出重建计划]

Phase 4 - 联调测试 (预计X天)
  [测试要点]

风险项:
  - [识别到的风险]

总预估: X 人天
```

---

## 执行注意事项

### 需要读取文件的场景
- 用户提供了Egret项目代码，需要分析`egretProperties.json`、`default.res.json`、`Main.ts`等
- 用户提供了特定的EXML文件需要分析UI结构
- 用户提供了TypeScript源码需要给出迁移方案

### 不需要读取文件的场景
- 用户问Egret和Cocos/Unity的API对应关系（直接用映射表回答）
- 用户问迁移策略建议（用决策启发式回答）
- 用户问资源格式兼容性（用资源复用矩阵回答）

### 输出风格
- **实用导向** — 给出可直接执行的具体步骤，不做泛泛而谈
- **表格优先** — API映射、资源分类等用表格呈现
- **标注风险** — 每个迁移步骤标注可能踩坑的地方
- **给出估时** — 在可能的情况下给出工作量预估

---

## 诚实边界

此工具的局限：

- **无法自动迁移EXML** — EUI的EXML皮肤文件没有自动转换到Cocos/Unity的工具，需要手动重建
- **API映射非100%覆盖** — 映射表覆盖了最常用的API，但Egret的一些冷门API可能需要单独查找对应方案
- **版本兼容性风险** — DragonBones跨引擎复用在v5.6.3及以下版本验证过，更高版本需实测
- **工作量预估为粗略值** — 实际工作量取决于项目复杂度、团队对目标引擎的熟悉程度等因素
- **不覆盖Egret 3D** — 本工具聚焦于Egret 2D项目迁移，Egret3D（实验性项目）不在范围内
- **营销类项目建议重做** — 对于短生命周期的营销活动项目，迁移不如重做经济，本工具会建议重做而非硬迁

---

## 附录：调研来源

调研过程详见 `references/research/` 目录（6个文件）。

### 核心参考来源
- [egret-labs/egret-core (GitHub)](https://github.com/egret-labs/egret-core) — Egret引擎源码和文档
- [Egret Engine官方文档](https://docs.egret.com/engine) — API参考和开发指南
- [DragonBones官方文档](https://docs.egret.com/dragonbones/en) — 骨骼动画系统
- [Cocos Creator 3.8 DragonBones支持](https://docs.cocos.com/creator/3.8/manual/en/asset/dragonbones.html) — DragonBones跨引擎兼容
- [Cocos Creator Node/Component API](https://docs.cocos.com/creator/3.8/manual/en/scripting/access-node-component.html) — 目标引擎API参考
- [Cocos Creator Event System](https://docs.cocos.com/creator/3.8/manual/en/engine/event/event-node.html) — 事件系统对照
- [egret-labs/egret-examples (GitHub)](https://github.com/egret-labs/egret-examples) — 官方示例项目
- [Egret Resource System Wiki](https://github.com/egret-labs/egret-core/wiki/Using-Resource-System) — RES资源管理系统

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
