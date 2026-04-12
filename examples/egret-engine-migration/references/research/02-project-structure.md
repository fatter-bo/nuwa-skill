# 维度2：Egret项目结构分析

## 调研来源
- [Egret Engine Project Structure](https://topic.alibabacloud.com/a/egret-engine-development-h5-game-project-structure_8_8_31101436.html)
- [egret-labs/egret-core Wiki - Using Resource System](https://github.com/egret-labs/egret-core/wiki/Using-Resource-System)
- [egret-labs/egret-examples](https://github.com/egret-labs/egret-examples)
- [egretProperties.json示例](https://github.com/egret-labs/Nest/blob/master/egretProperties.json)

## 核心发现

### 标准目录结构
```
project-root/
├── .wing/                    # EgretWing IDE配置
├── bin-debug/                # 调试编译输出
├── bin-release/              # 发布编译输出
├── libs/                     # 引擎库文件
│   └── modules/
│       ├── egret/            # 核心模块
│       ├── eui/              # UI组件模块
│       ├── res/              # 资源管理模块
│       ├── tween/            # 缓动模块
│       ├── dragonBones/      # 骨骼动画模块
│       └── game/             # 游戏模块
├── resource/                 # 资源目录
│   ├── assets/               # 图片、音频等静态资源
│   ├── config/               # 配置文件
│   ├── eui_skins/            # EUI皮肤EXML文件
│   ├── default.res.json      # 资源配置清单
│   └── default.thm.json      # 主题配置
├── src/                      # TypeScript源码
│   ├── Main.ts               # 入口类
│   ├── LoadingUI.ts          # 加载界面
│   └── ...                   # 业务逻辑
├── template/                 # 模板文件
├── egretProperties.json      # 项目配置（引擎版本、模块列表）
├── index.html                # 入口HTML
├── tsconfig.json             # TypeScript配置
└── wingProperties.json       # Wing IDE配置
```

### 关键配置文件

#### egretProperties.json
```json
{
    "engineVersion": "5.4.1",
    "compilerVersion": "5.4.1",
    "template": {},
    "target": { "current": "web" },
    "modules": [
        { "name": "egret" },
        { "name": "eui" },
        { "name": "res" },
        { "name": "tween" },
        { "name": "dragonBones" },
        { "name": "game" }
    ],
    "eui": {
        "exmlRoot": "resource/eui_skins",
        "themes": "resource/default.thm.json",
        "exmlPublishPolicy": "path"
    }
}
```
- 决定引擎版本和启用的模块
- eui字段配置EXML皮肤路径和主题文件

#### default.res.json（资源清单）
```json
{
    "resources": [
        { "name": "bg_jpg", "type": "image", "url": "assets/bg.jpg" },
        { "name": "ui_json", "type": "sheet", "url": "assets/ui.json" },
        { "name": "dragon_ske_json", "type": "json", "url": "assets/dragon_ske.json" }
    ],
    "groups": [
        { "name": "preload", "keys": "bg_jpg,ui_json" },
        { "name": "game", "keys": "dragon_ske_json" }
    ]
}
```
- resources：定义每个资源的name（唯一标识）、type、url
- groups：定义资源组，按组加载

#### default.thm.json（主题配置）
```json
{
    "skins": {
        "eui.Button": "resource/eui_skins/ButtonSkin.exml",
        "eui.CheckBox": "resource/eui_skins/CheckBoxSkin.exml",
        "eui.ProgressBar": "resource/eui_skins/ProgressBarSkin.exml"
    },
    "exmls": [
        "resource/eui_skins/ButtonSkin.exml",
        "resource/eui_skins/PanelSkin.exml"
    ]
}
```
- skins：组件类型到默认皮肤文件的映射
- exmls：需要预加载的exml文件列表

### EXML界面描述文件
```xml
<?xml version="1.0" encoding="utf-8"?>
<e:Skin class="MainSkin" states="normal,disabled"
    xmlns:e="http://ns.egret.com/eui"
    xmlns:w="http://ns.egret.com/wing">
    <e:Image source="bg_jpg" width="100%" height="100%"/>
    <e:Button id="startBtn" label="开始游戏"
        horizontalCenter="0" verticalCenter="0"/>
    <e:List id="itemList" dataProvider="{dataList}">
        <e:itemRendererSkinName>ItemSkin</e:itemRendererSkinName>
        <e:layout>
            <e:VerticalLayout gap="10"/>
        </e:layout>
    </e:List>
</e:Skin>
```
- 基于XML的声明式UI描述，类似Android Layout XML或MXML
- 支持数据绑定（花括号语法）、状态管理、布局系统
- 组件与皮肤分离，一个组件可切换不同皮肤

### 资源类型体系
| type值 | 说明 | 文件格式 |
|--------|------|---------|
| image | 普通图片 | .jpg, .png |
| sheet | 精灵图集 | .json + .png |
| json | JSON数据 | .json |
| text | 文本 | .txt, .xml |
| font | 位图字体 | .fnt + .png |
| sound | 音频 | .mp3, .ogg |
| bin | 二进制 | 任意 |

### 资源加载流程
1. `RES.loadConfig("default.res.json", "resource/")` — 加载资源配置
2. `RES.loadGroup("preload")` — 按组加载资源
3. 监听 `RES.ResourceEvent.GROUP_COMPLETE` — 加载完成回调
4. `RES.getRes("bg_jpg")` — 获取已加载资源
5. `RES.getResAsync("item", callback)` — 异步获取单个资源
