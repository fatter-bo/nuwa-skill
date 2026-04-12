# 维度2：资源提取方法论

## 资源类型与定位策略

### 图片资源
- **直接图片**：解包后在 `res/`、`assets/`、`resources/` 等目录下，格式为 PNG/JPG/WebP
- **Base64内嵌**：部分小图直接编码在JS/CSS中，需正则提取 `data:image/png;base64,...`
- **远程资源**：通过CDN加载的资源需抓包获取URL，批量下载

### 图集（Sprite Sheet）还原
小游戏常用 TexturePacker 将多张小图合并为一张大图+描述文件，需逆向还原。

**描述文件格式**：
- `.plist`（Cocos系常用）
- `.json`（通用格式，含frame坐标、旋转、裁切信息）
- `.atlas`（Spine动画专用）

**还原工具**：
| 工具 | 支持格式 | 链接 |
|------|---------|------|
| texture-unpacker (Node.js) | plist, json | [GitHub](https://github.com/pavle-goloskokovic/texture-unpacker) |
| TextureUnpacker (Python) | plist, atlas | [GitHub](https://github.com/aa13058219642/TextureUnpacker) |
| pangliang/texture-unpacker | plist, json | [GitHub](https://github.com/pangliang/texture-unpacker) |
| TexturePacker内置Splitter | 所有TP格式 | [官方](https://www.codeandweb.com/texturepacker) |

**还原流程**：
1. 定位大图PNG + 对应描述文件（plist/json）
2. 解析描述文件中每个sprite的坐标、尺寸、旋转标记、裁切偏移
3. 从大图中按坐标切割，反转旋转，补回透明边距
4. 输出独立sprite文件

来源：[TexturePacker官方文档](https://www.codeandweb.com/texturepacker/documentation/sprite-sheet-cutter)，GitHub开源项目 — 可信度：高

### Spine动画资源
Spine骨骼动画由三部分组成：
- `.json` 或 `.skel`（骨骼数据，二进制或JSON）
- `.atlas`（图集描述）
- `.png`（图集图片）

**提取方法**：
1. 在解包目录中搜索 `.skel`、`.atlas` 文件或含 `skeleton`、`bones`、`slots` 关键词的JSON
2. 使用 [Spine Viewer](http://esotericsoftware.com/) 或开源的 [spine-web-player](https://github.com/niceguydave/spine-web-player) 预览验证
3. 注意版本兼容性：Spine 3.x 和 4.x 的 skel 二进制格式不同

### 粒子配置
- Cocos系使用 `.plist` 格式的粒子配置文件
- 搜索关键词：`emitterType`、`maxParticles`、`particleLifespan`
- 可直接导入 Cocos Creator 粒子编辑器

### 音频资源
- 常见格式：`.mp3`、`.ogg`、`.wav`
- 微信小游戏对音频格式有限制，主要支持 MP3
- 音频文件通常在 `res/audio/` 或 `res/sound/` 目录

### 字体资源
- `.ttf`、`.woff` 文件直接可用
- 位图字体（BitmapFont）：`.fnt` + `.png` 组合
- 搜索关键词：`font`、`BMFont`、`labelAtlas`

## 资源加密与混淆方案

### 常见加密方式

| 加密方式 | 特征 | 应对策略 |
|---------|------|---------|
| TexturePacker内容保护 | 图片打开全黑/花屏，有密钥 | IDA调试提取密钥，或从代码中搜索密钥 |
| 自定义XOR加密 | 文件头不符合标准格式 | 分析JS代码中的解密函数，提取密钥 |
| Base64编码 | 长字符串以 = 结尾 | 直接Base64解码 |
| 资源名混淆 | 文件名为hash值 | 从配置文件（如settings.js）中查找映射关系 |
| 远程动态加载 | 本地包中无资源 | 抓包获取CDN地址，批量下载 |

来源：[IDA远程调试破解TexturePacker加密素材](https://blog.jtwo.me/ida-remote-debug-texturepacker-material/)，CSDN技术博客 — 可信度：中-高

### 加密资源识别方法
1. 检查文件头Magic Number是否符合标准格式（PNG: `89 50 4E 47`，JPG: `FF D8 FF`）
2. 若不符合，尝试在JS代码中搜索 `decrypt`、`decipher`、`xor`、`aes` 等关键词
3. 定位解密函数，提取密钥和算法
4. 编写对应解密脚本

## 资源清单生成

提取完成后应生成资源清单：
```
资源清单模板：
├── 图片资源 (X张)
│   ├── UI图标 (Xpx × Xpx) × N
│   ├── 角色素材 (含Spine动画) × N
│   ├── 场景背景 × N
│   └── 特效粒子 × N
├── 音频资源 (X个)
│   ├── BGM × N
│   └── 音效 × N
├── 字体资源 × N
└── 配置文件 × N
```
