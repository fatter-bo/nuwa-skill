# 维度5：资源提取与复用

## 调研来源
- [DragonBones Export Animation](https://docs.egret.com/dragonbones/en/docs/dbPro/export/exportAnimation)
- [DragonBones Create Animation](https://docs.egret.com/dragonbones/en/docs/dbLibs/createAnimation)
- [Egret Resource System Wiki](https://github.com/egret-labs/egret-core/wiki/Using-Resource-System)
- [Cocos Creator DragonBones Assets](https://docs.cocos.com/creator/3.8/manual/en/asset/dragonbones.html)

## 核心发现

### 可提取的资源类型总览
| 资源类型 | 存放位置 | 格式 | 复用难度 |
|---------|---------|------|---------|
| 普通图片 | resource/assets/ | .png, .jpg | 低 — 直接复制 |
| 精灵图集 | resource/assets/ | .json + .png | 中 — 需拆分或重新打包 |
| DragonBones动画 | resource/assets/ | _ske.json + _tex.json + _tex.png | 低 — 跨引擎通用格式 |
| 位图字体 | resource/assets/ | .fnt + .png | 中 — 格式兼容但需验证 |
| 音频文件 | resource/assets/ | .mp3, .ogg | 低 — 直接复制 |
| 粒子配置 | resource/assets/ | .json | 高 — 格式不通用，需手动重建 |
| EXML皮肤 | resource/eui_skins/ | .exml | 高 — 仅Egret可用，需手动重建UI |
| JSON配置 | resource/config/ | .json | 低 — 通用格式，逻辑需适配 |

### 图片资源提取

#### 独立图片
直接从`resource/assets/`目录复制，无需任何转换。

#### 精灵图集(SpriteSheet)拆分
Egret的精灵图集由两个文件组成：
- **图集描述** (.json)：记录每个子图的名称、位置、尺寸
- **合并图片** (.png)：包含所有子图的大图

```json
{
    "file": "spritesheet.png",
    "frames": {
        "btn_normal": { "x": 0, "y": 0, "w": 120, "h": 40, "offX": 0, "offY": 0, "sourceW": 120, "sourceH": 40 },
        "btn_press": { "x": 120, "y": 0, "w": 120, "h": 40, "offX": 0, "offY": 0, "sourceW": 120, "sourceH": 40 }
    }
}
```

**提取方法**：
1. 用Python脚本（PIL/Pillow）根据JSON中的坐标裁剪出单独图片
2. 或使用TexturePacker的Import功能直接导入，再导出为目标引擎格式
3. Cocos Creator可直接使用Egret的图集格式（需验证兼容性）

### DragonBones动画数据导出

DragonBones动画数据由三个文件组成：
1. **骨骼数据** (`xxx_ske.json` 或 `xxx_ske.dbbin`)：骨骼层级、动画关键帧
2. **图集描述** (`xxx_tex.json`)：纹理图集中每个部件的位置
3. **图集纹理** (`xxx_tex.png`)：所有骨骼部件的合并图片

**跨引擎复用**：
- **Cocos Creator**：直接复制三个文件到Cocos项目，原生支持DragonBones v5.6.3及以下
- **Unity**：使用DragonBones Unity Runtime库，导入相同的数据文件
- **重新编辑**：用DragonBones Pro软件打开原始工程文件(.dbproj)可重新编辑并导出

**注意**：如果只有导出后的JSON文件没有.dbproj工程文件，仍可在DragonBones Pro中导入JSON重新编辑，但部分编辑元数据会丢失。

### 位图字体提取

Egret使用标准BMFont格式：
- `.fnt`文件：字符映射数据（文本格式，记录每个字符的位置、大小、偏移）
- `.png`文件：字符纹理图片

**复用方案**：
- BMFont是业界通用格式，Cocos Creator和Unity都原生支持
- Cocos Creator：将.fnt和.png放入assets，创建Label组件选择BMFont类型
- Unity：使用TextMeshPro的BMFont导入功能

### 粒子系统配置

Egret的粒子效果通常使用`egret-particle`扩展库，配置存储为JSON：
```json
{
    "emitterType": 0,
    "maxParticles": 100,
    "speed": 200,
    "lifespan": 1.0,
    "startSize": 30,
    "endSize": 0,
    "startColor": {"r": 255, "g": 255, "b": 255, "a": 255}
}
```

**迁移方案**：
- 粒子配置格式不通用，无法直接导入其他引擎
- 需要在目标引擎中根据原始参数手动重建粒子效果
- 参数含义（发射类型、速度、生命周期、大小变化、颜色变化）在各引擎中概念一致，可对照调整

### 音频资源提取
- 直接复制.mp3/.ogg文件到目标项目
- Egret通常同时提供mp3和ogg两种格式以兼容不同浏览器
- Unity/Cocos Creator都支持这两种格式，无需转换

### 批量提取工具脚本思路

```python
# 伪代码：从Egret项目提取所有资源
import json, os, shutil
from PIL import Image

def extract_resources(egret_project_path, output_path):
    # 1. 读取default.res.json获取资源清单
    res_config = json.load(open(f"{egret_project_path}/resource/default.res.json"))

    for res in res_config["resources"]:
        src = f"{egret_project_path}/resource/{res['url']}"
        if res["type"] == "image":
            shutil.copy(src, f"{output_path}/images/")
        elif res["type"] == "sheet":
            # 解析JSON拆分精灵图集
            split_spritesheet(src, output_path)
        elif res["type"] == "sound":
            shutil.copy(src, f"{output_path}/audio/")
        elif res["type"] == "font":
            shutil.copy(src, f"{output_path}/fonts/")

    # 2. 复制DragonBones文件（匹配_ske.json/_tex.json/_tex.png三元组）
    copy_dragonbones_assets(egret_project_path, output_path)
```

### 资源复用决策矩阵
| 资源 | 直接复用 | 需转换 | 需重建 |
|------|---------|--------|--------|
| PNG/JPG图片 | ✅ | | |
| MP3/OGG音频 | ✅ | | |
| DragonBones动画 | ✅ | | |
| BMFont字体 | ✅ | | |
| JSON配置数据 | ✅ | | |
| 精灵图集 | | ✅ 需重新打包 | |
| 粒子效果 | | | ✅ 手动重建 |
| EXML皮肤 | | | ✅ 手动重建UI |
