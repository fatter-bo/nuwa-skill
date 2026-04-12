# 维度一：包体大小控制

## 1. 代码压缩与 Tree Shaking

### JavaScript 压缩
- Terser/UglifyJS 对构建产物进行代码压缩：变量名混淆、Dead Code Elimination、常量折叠
- Cocos Creator 构建时默认开启代码压缩，可在 Build 面板勾选 "Compress JavaScript"
- 生产构建务必关闭 SourceMap 以减小产物体积

### Tree Shaking
- Cocos Creator 3.x 基于模块化引擎，支持按功能裁剪未使用的引擎模块（Physics、Particle、Terrain 等）
- 在 Project Settings > Feature Crop 中取消勾选不需要的模块，可显著减小引擎代码体积
- 业务代码方面，确保使用 ES Module 的 import/export 以利于 Rollup/Webpack 的 Tree Shaking

来源: [Cocos Creator 3.8 Project Settings](https://docs.cocos.com/creator/3.8/manual/en/editor/project/)

### Unity WebGL 代码裁剪
- 设置 Managed Stripping Level 为 Medium 或 High，移除未使用的 .NET 库
- High 级别裁剪需充分测试，可能裁掉反射调用的类
- 通过 link.xml 保护需要保留的类型

来源: [10 Critical Unity WebGL Performance Tips](https://friendzy.xyz/2025/09/17/unity-webgl-performance-tips/)

## 2. 纹理压缩格式选型

### ASTC（Adaptive Scalable Texture Compression）
- 推荐首选格式，压缩率高、质量好，支持 HDR 纹理
- 块大小选择：
  - 高质量需求（2D UI）：ASTC 4x4（8bpp）或 5x5（5.12bpp）
  - 中等质量（模型纹理）：ASTC 6x6（3.56bpp）
  - 低质量容忍（背景/特效）：ASTC 8x8（2bpp）或 10x10（1.28bpp）
- 兼容性：iOS 全系支持，Android OpenGL ES 3.1+ / 大部分 Mali/Adreno GPU 支持

### ETC2
- ETC1 的扩展，支持 Alpha 通道（RGBA），压缩比与 ETC1 相同但质量更高
- Android OpenGL ES 3.0+ 全面支持
- 对于需要兼容老旧 Android 设备的项目仍是安全选择

### WebP
- 有损压缩质量优于 JPEG，支持 Alpha 通道
- 适合非 GPU 直接消费的场景（如 Loading 界面图片、预览图）
- 注意：WebP 是 CPU 解码格式，不可直接上传 GPU，运行时需转为 RGBA

### 选型决策树
```
需要 GPU 直接使用？
├── 是 → 目标平台支持 ASTC？
│   ├── 是 → 使用 ASTC（按质量需求选块大小）
│   └── 否 → 使用 ETC2（Android）/ PVRTC（旧 iOS，已不推荐）
└── 否 → 使用 WebP（有 Alpha）或 JPEG（无 Alpha）
```

来源: [Cocos Creator Compressed Textures](https://docs.cocos.com/creator/3.8/manual/en/asset/compress-texture.html)

## 3. 音频压缩

### 格式选择
- 背景音乐：Ogg Vorbis（.ogg），码率 96-128kbps，使用流式加载（Streaming）
- 音效：MP3 或 Ogg，短音效可选择预加载到内存
- 避免使用 WAV 等无损格式

### 压缩策略
- 背景音乐采样率降至 22050Hz（人耳对音乐频率敏感度有限）
- 短音效采样率可降至 11025Hz
- 使用单声道（Mono）而非立体声（Stereo），体积减半
- Unity WebGL：音频设置 Load Type 为 "Streaming"，Format 为 "Vorbis"

来源: [Unity WebGL Performance Tips](https://friendzy.xyz/2025/09/17/unity-webgl-performance-tips/)

## 4. 字体子集化

### CJK 字体问题
- 完整中文字体（如 NotoSansSC）可达 8-16MB，包含数万字形
- 游戏中实际使用的汉字通常不超过 3000-5000 个

### 子集化工具
- **pyftsubset**（fonttools 库）：`pyftsubset input.ttf --text-file=chars.txt --output-file=subset.woff2 --flavor=woff2`
- **glyphhanger**：自动扫描网页使用的字符并生成子集字体
- **fontmin**：针对中文字体的子集化工具

### 效果
- 完整字体 190KB → 子集化后 WOFF2 仅 7.4KB（节省 96%）
- 实际中文游戏场景：16MB 字体 → 按使用字符子集化后可降至 200-500KB

来源: [fonttools subset documentation](https://fonttools.readthedocs.io/en/stable/subset/)
来源: [glyphhanger](https://github.com/filamentgroup/glyphhanger)

## 5. 代码分包

### Cocos Creator 分包
- 将非首屏资源划分到不同 Asset Bundle
- 主包只包含启动场景和核心框架代码
- 游戏内容按关卡/模块拆分为独立 Bundle，按需加载

### Unity Addressable Assets
- 将资源与初始构建解耦，实现按需加载
- 创建极小的启动包，后续资源延迟加载
- 通过 Addressable Groups 管理资源分组和加载策略

来源: [Unity WebGL Performance Tips - Addressable Assets](https://friendzy.xyz/2025/09/17/unity-webgl-performance-tips/)

## 6. 微信小游戏分包限制

### 包体限制规范
| 项目 | 限制 |
|------|------|
| 主包大小 | ≤ 4MB |
| 单个普通分包 | 不限制 |
| 单个独立分包 | ≤ 4MB |
| 所有包总大小 | ≤ 20MB（开通虚拟支付后 ≤ 30MB）|

### 分包策略
- 主包仅放启动场景 + 核心框架 + Loading 资源
- 游戏资源按模块拆分为普通分包
- 首屏完成后在后台静默下载其他分包
- 利用"分包预下载"配置，在进入某页面时预下载下一模块的分包

### 突破限制的方案
- 大资源（纹理/音频/配置）放 CDN，运行时动态下载
- 利用微信小游戏文件系统缓存已下载资源
- 配置表使用 JSON 压缩 + 懒加载

来源: [微信小游戏分包加载文档](https://developers.weixin.qq.com/minigame/dev/guide/base-ability/subPackage/useSubPackage.html)
