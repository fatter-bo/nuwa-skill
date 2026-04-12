---
name: h5-minigame-performance
description: |
  H5/小游戏性能优化专家。专注于Cocos Creator和Unity WebGL在H5和小游戏平台上的性能优化，
  覆盖包体大小、加载速度、运行帧率、内存管理的全链路优化。
  触发词：「性能优化」「包体太大」「加载慢」「卡顿」「帧率低」「内存泄漏」「DrawCall」
---

# H5/小游戏性能优化

> 核心目标：让 H5/小游戏在包体、加载、帧率、内存四个维度都达到生产级标准——包体小于限制、首屏秒开、60FPS 流畅、内存可控不崩溃。

## Skill 规则

- 优化建议必须同时考虑 Cocos Creator 和 Unity WebGL 两套引擎
- 所有纹理方案必须附带内存估算（宽×高×bpp），不接受"差不多"
- 性能指标必须量化：给出具体数值目标（FPS、DrawCall、内存、加载时间）
- 微信小游戏优化必须考虑分包限制（主包 4MB / 总包 20-30MB）
- 每条优化建议标注"收益等级"（高/中/低）和"实施难度"（易/中/难）

---

## 回答工作流

| 触发指令 | 行动 |
|---------|------|
| 「包体太大」 | 诊断包体组成 → 输出纹理压缩/代码裁剪/音频压缩/字体子集化方案 |
| 「加载慢」 | 分析加载链路 → 输出分包/预加载/CDN缓存/Loading界面方案 |
| 「卡顿/帧率低」 | 定位瓶颈（CPU/GPU）→ 输出DrawCall合批/对象池/Shader优化方案 |
| 「内存泄漏」 | 输出检测流程 + 工具使用 + 常见泄漏场景排查清单 |
| 「DrawCall太高」 | 分析合批条件 → 输出图集整合/节点树调整/材质合并方案 |
| 「Unity WebGL优化」 | 输出WASM编译/IL2CPP/压缩选型/内存管理全套方案 |
| 「性能监控」 | 输出埋点体系 + Chrome DevTools用法 + 告警策略 |

---

## 一、包体大小控制

### 1.1 纹理压缩选型矩阵

纹理通常占包体和内存的 60-80%，是优化的第一优先级。

| 格式 | bpp | 质量 | 适用场景 | 平台兼容性 |
|------|-----|------|---------|-----------|
| ASTC 4x4 | 8 | 高 | UI、角色立绘 | iOS全系, Android ES3.1+ |
| ASTC 6x6 | 3.56 | 中 | 模型纹理 | 同上 |
| ASTC 8x8 | 2 | 中低 | 背景、地图 | 同上 |
| ETC2 RGBA | 8 | 中 | 需兼容老Android | Android ES3.0+ |
| WebP | 可变 | 高 | 非GPU纹理(Loading图) | 全平台 |

**选型决策树**：
```
需要GPU直接使用？
├── 是 → 目标平台支持ASTC？
│   ├── 是 → ASTC（按质量需求选块大小）
│   └── 否 → ETC2 (Android) / 降级 PNG
└── 否 → WebP（有Alpha）或 JPEG（无Alpha）
```

**内存估算公式**：`纹理内存 = 宽 × 高 × bpp ÷ 8`
- 1024×1024 RGBA8888 = **4MB** → ASTC 4x4 = **1MB** → ASTC 8x8 = **256KB**

### 1.2 代码体积优化

**Cocos Creator**：
- 在 Project Settings > Feature Crop 中裁剪未使用的引擎模块（Physics、Particle、Terrain）
- 构建时开启代码压缩（Terser）
- 使用 ES Module 的 import/export 以利于 Tree Shaking

**Unity WebGL**：
- Managed Stripping Level 设为 Medium/High（移除未使用的 .NET 库）
- 通过 `link.xml` 保护反射调用的类型
- 避免 LINQ、System.Reflection（阻止代码裁剪）

### 1.3 音频与字体

**音频**：背景音乐用 Ogg Vorbis 96-128kbps + 流式加载；音效用 MP3/Ogg 单声道；采样率降至 22050Hz。

**字体子集化**：完整中文字体 8-16MB → pyftsubset 按使用字符子集化 → 200-500KB（节省 95%+）。
```bash
pyftsubset input.ttf --text-file=chars.txt --output-file=subset.woff2 --flavor=woff2
```

### 1.4 微信小游戏分包策略

| 项目 | 限制 |
|------|------|
| 主包 | ≤ 4MB |
| 单个独立分包 | ≤ 4MB |
| 总包 | ≤ 20MB（虚拟支付 ≤ 30MB）|

**策略**：主包仅放启动场景 + 核心框架 + Loading 资源；游戏资源按模块拆分普通分包；大资源放 CDN 动态下载 + 本地文件系统缓存。

---

## 二、首屏加载优化

### 2.1 加载链路分析

```
用户点击 → HTML下载 → 引擎JS/WASM下载 → 引擎初始化 → 首屏资源加载 → 首帧渲染
 ①              ②              ③               ④              ⑤
```

每一段都是优化点：
- **①** HTML 极简化（< 5KB）
- **②** 引擎文件 Brotli 压缩 + CDN 分发 + 强缓存（Hash 文件名）
- **③** 并行初始化 + 分帧避免阻塞
- **④** 只加载首屏必要资源（P0/P1），其余懒加载
- **⑤** 纯 HTML/CSS Loading 界面在引擎初始化前即可见

### 2.2 资源优先级分层

```
P0 - 立即需要：Loading界面资源、核心引擎代码
P1 - 首屏需要：启动场景纹理、UI图集、必要音效
P2 - 即将需要：下一关卡资源、常用特效
P3 - 延迟加载：高级关卡、可选音乐、设置界面
```

### 2.3 缓存策略

| 资源类型 | 缓存策略 | Cache-Control |
|----------|---------|---------------|
| 引擎 JS/WASM | 强缓存 + 文件名Hash | max-age=31536000, immutable |
| 纹理图集 | 强缓存 + Hash | max-age=31536000, immutable |
| 配置文件 | 协商缓存 | no-cache（校验ETag）|
| HTML 入口 | 不缓存 | no-store |

### 2.4 Loading 界面最佳实践

- 使用纯 HTML/CSS 实现，不依赖引擎，即刻可见
- 显示真实加载进度（前90%真实进度 + 后10%引擎初始化）
- 加入游戏提示/小贴士转移等待焦虑
- Unity WebGL：自定义 HTML/CSS Loading 替代引擎内置方案

---

## 三、运行时性能

### 3.1 DrawCall 合批

**Cocos Creator 2D 合批五大条件**（全部满足方可合批）：
1. 相同摄像机可见性
2. 相同材质（Material + Effect + Defines + Uniform）
3. 相同混合/深度状态
4. 共享顶点缓冲（MeshBuffer）
5. 相同纹理和采样器

**合批优化实操**：
- 使用 Auto Atlas / Dynamic Atlas 合并纹理
- 节点树中同图集节点相邻排列，避免交叉打断合批
- Label 使用 BMFont/位图缓存使其能与 Sprite 合批
- 不可合批组件（Mask、Graphics）单独分层

**3D 合批方式**：
| 方式 | 适用场景 | 开销 |
|------|---------|------|
| 静态合批 | 不动物体 | 内存增加，零运行时开销 |
| 动态合批 | 低面数运动物体 | CPU 每帧重算顶点 |
| GPU Instancing | 大量相同Mesh | 最优，需Shader支持 |

### 3.2 对象池

运行时频繁 instantiate/destroy 是帧率杀手。对象池的 get/put 开销接近零。

**最佳实践**：
- 场景 onLoad 中预创建池
- 一种 Prefab 一个池
- 利用 reuse/unuse 回调重置状态
- 池大小 = 同屏最大数量（非总需求量）

```typescript
// 通用对象池
class ObjectPool<T> {
    private pool: T[] = [];
    constructor(private factory: () => T, private reset: (obj: T) => void, initSize: number) {
        for (let i = 0; i < initSize; i++) this.pool.push(factory());
    }
    get(): T { return this.pool.pop() ?? this.factory(); }
    put(obj: T): void { this.reset(obj); this.pool.push(obj); }
}
```

### 3.3 Shader 与渲染优化

- **避免 discard / alpha test**：破坏 GPU HSR 机制
- **减少纹理采样**：合并采样，使用图集
- **使用低精度**：`lowp`/`mediump` 替代 `highp`（position 除外）
- **控制 Overdraw**：半透明对象是主要来源，限制全屏后处理效果
- **渲染排序**：不透明前到后，半透明后到前

### 3.4 节点树与更新优化

- 不可见节点 `node.active = false`（非设置透明度 0）
- 滚动列表使用虚拟列表（仅实例化可见项）
- 扁平化节点树，减少遍历深度
- 屏幕外对象暂停更新逻辑

---

## 四、内存管理

### 4.1 内存预算规划

```
移动端安全线 ≤ 256MB：
├── 引擎运行时       20-50MB
├── 纹理内存         50-150MB（最大消耗者）
├── 顶点/索引缓冲    10-30MB
├── 音频缓冲         10-30MB
└── 脚本堆内存       20-50MB

桌面端安全线 ≤ 512MB
```

### 4.2 资源释放策略

- 场景切换时释放上一场景专属资源
- 常驻资源（全局UI、通用音效）标记常驻不随场景释放
- 定期调用 `assetManager.releaseUnusedAssets()` 清理引用计数为 0 的资源
- 动态创建的纹理（RenderTexture）必须手动 destroy

### 4.3 内存泄漏检测清单

| 泄漏类型 | 表现 | 排查方法 |
|---------|------|---------|
| 事件监听器未移除 | 节点销毁后内存不释放 | Heap Snapshot 查 Detached DOM |
| 定时器未清除 | 闭包持有节点引用 | 检查 setInterval/setTimeout |
| 纹理未释放 | WebGL 内存持续增长 | WebGL-Memory 追踪 |
| 闭包引用 | 大对象无法 GC | Allocation Timeline 分析 |
| 全局变量缓存 | 堆内存只增不减 | Heap Snapshot 对比 |

**检测流程**：初始快照 M0 → 执行操作 N 次 → 强制 GC → 快照 M1 → 若 M1-M0 线性增长则存在泄漏。

### 4.4 WebGL 内存特殊注意

- iOS Safari 限制最严，超限直接杀 Tab 无警告
- Unity WebGL WASM 堆自动增长可能失败导致崩溃，建议固定内存预算
- `texImage2D` 可能需要双倍内存（相比 `texStorage2D`）
- GPU 压缩纹理（ASTC/ETC2）内存消耗比 RGBA8888 低 4-8 倍

---

## 五、Unity WebGL 特有优化

### 5.1 WASM 编译与 IL2CPP

**Build 模式**：开发用 "Shorter build time"，发布用 "Faster (smaller) builds"。

**Enable Exceptions**：发布设为 None（最小产物、最佳性能），需要异常处理则 "Full Without Stacktrace"。

**代码优化**：
- 缓存 GetComponent 结果
- 用 StringBuilder 替代 string 拼接
- 用 struct 处理简单数据容器
- 避免 LINQ、反射、动态代码生成

### 5.2 Brotli vs Gzip

| 特性 | Brotli | Gzip |
|------|--------|------|
| 压缩率 | 优（比Gzip小15-25%）| 良 |
| HTTPS 要求 | Chrome/Firefox 需要 | 无 |
| 服务器支持 | 需确认 | 几乎全部 |

**决策**：HTTPS + 服务器支持 → Brotli 级别 11；否则 → Gzip。效果：构建数据缩小 60-80%，加载从 10s+ 降到 3s 以内。

### 5.3 发布检查清单

- [x] Compression: Brotli
- [x] Enable Exceptions: None
- [x] Managed Stripping Level: Medium/High
- [x] 纹理 Crunch Compression
- [x] 音频 Vorbis + Streaming
- [x] Addressable Assets 按需加载
- [x] 自定义 HTML/CSS Loading
- [x] 内存预算 256-512MB

---

## 六、性能监控与调试

### 6.1 工具矩阵

| 工具 | 用途 | 适用阶段 |
|------|------|---------|
| Chrome Performance | CPU/GPU 帧分析 | 开发+测试 |
| Chrome Memory | 内存泄漏检测 | 开发+测试 |
| WebGL-Memory | WebGL 资源追踪 | 开发 |
| Cocos Profiler | DrawCall/FPS 实时统计 | 开发 |
| 自定义埋点 | 线上性能监控 | 生产 |

### 6.2 关键性能指标

```typescript
interface PerfMetrics {
    // 加载
    engineInitTime: number;      // 引擎初始化 ≤ 2s
    firstSceneLoadTime: number;  // 首场景加载 ≤ 3s
    totalStartupTime: number;    // 总启动时间 ≤ 5s
    // 运行
    avgFPS: number;              // 平均帧率 ≥ 55
    minFPS: number;              // 最低帧率 ≥ 30
    drawCallCount: number;       // 2D游戏 ≤ 50, 3D游戏 ≤ 150
    // 内存
    jsHeapUsed: number;          // ≤ 150MB
    webglMemory: number;         // ≤ 200MB
}
```

### 6.3 告警策略

| 指标 | 阈值 | 告警级别 |
|------|------|---------|
| 崩溃率 | > 1% | P0 |
| 平均 FPS | < 25 | P1 |
| 启动时间 | > 8s | P2 |
| 内存使用 | > 300MB | P2 |
| DrawCall (2D) | > 100 | P3 |

---

## 框架：性能优化决策框架

### 框架一：性能优化优先级矩阵

按"收益×实施难度"排序，先做高收益低难度项：

```
         高收益
           │
   ② 优先做 │ ① 立刻做
     中难度  │  低难度
           │
  ─────────┼─────────
           │
   ④ 评估后做│ ③ 有空做
     高难度  │  低难度
           │
         低收益
```

### 框架二：瓶颈定位流程

```
帧率低？
├── CPU 瓶颈 → Chrome Performance 火焰图
│   ├── Scripting 过高 → 优化游戏逻辑/对象池
│   ├── GC 频繁 → 减少临时对象分配
│   └── Rendering 过高 → 减少节点数/简化布局
└── GPU 瓶颈 → DrawCall/Overdraw 分析
    ├── DrawCall 过高 → 图集合并/合批优化
    ├── Overdraw 过高 → 减少半透明/后处理
    └── Shader 过重 → 简化Fragment Shader
```

### 框架三：包体预算分配

```
总预算 = 平台限制（如微信小游戏 20MB）

推荐分配：
├── 引擎代码      15-25%
├── 纹理资源      40-50%
├── 音频资源      10-15%
├── 配置/数据     5-10%
├── 字体          3-5%
└── 其他（Shader等）2-5%
```

### 框架四：纹理格式决策树

见 1.1 节选型矩阵。核心原则：GPU 消费用压缩纹理，CPU 消费用 WebP/JPEG。

### 框架五：内存生命周期管理

```
资源生命周期：
加载 → 使用 → 标记不需要 → 等待释放窗口 → 释放

释放时机：
├── 即时释放：一次性资源（过场动画纹理）
├── 场景释放：场景专属资源（关卡纹理）
├── LRU 释放：缓存资源（历史关卡纹理池）
└── 永不释放：常驻资源（全局UI、通用音效）
```

### 框架六：加载策略决策

```
资源加载策略选择：
├── 启动必须 → 打入主包/首屏加载
├── 即将使用 → 预加载（预测式/触发式）
├── 可能使用 → 懒加载（首次使用时加载）
└── 低频使用 → CDN 远程加载 + 本地缓存
```

### 框架七：设备分级适配

```
设备等级判定：
├── 高端：WebGL2 + maxTextureSize ≥ 8192 → 全特效
├── 中端：WebGL2 + maxTextureSize ≥ 4096 → 减少后处理
└── 低端：WebGL1 / maxTextureSize < 4096 → 最低画质
```

---

## 启发式规则

1. **"纹理优先"原则**：性能优化的第一刀永远切在纹理上——压缩格式 + 尺寸控制 + 图集合并，这三项能解决 60% 以上的包体和内存问题。

2. **"DrawCall 50 法则"**：2D 游戏 DrawCall 控制在 50 以内，3D 简单场景 150 以内。超过这个数字先查合批条件是否满足，而非优化单个 DrawCall 的内容。

3. **"分帧不分线程"**：H5/小游戏是单线程环境，大任务不能开线程，只能分帧执行。每帧预算 16ms（60FPS），单帧耗时超过 50ms 就是 Long Task。

4. **"先量化再优化"**：不 Profile 不优化。先用 Chrome DevTools 和 Cocos Profiler 量化瓶颈，再针对性解决，拒绝凭感觉优化。

5. **"内存 256 红线"**：移动端 WebGL 内存目标控制在 256MB 以内。iOS Safari 超限直接杀进程无警告，没有第二次机会。

6. **"Loading 不能白屏"**：用户可以等 5 秒但不能看 5 秒白屏。引擎初始化前必须有纯 HTML/CSS 的 Loading 界面，给出进度反馈。

7. **"缓存是最好的加载优化"**：通过文件名 Hash + 强缓存 + CDN，让二次访问的用户几乎零等待。首次加载优化 CDN + Brotli 压缩。

8. **"GC 是帧率的隐形杀手"**：频繁创建临时对象触发 GC Pause，导致间歇性卡顿。用对象池复用对象、用 StringBuilder 替代字符串拼接、避免每帧 new 对象。

9. **"合批的敌人是交叉"**：DrawCall 合批失败的第一原因是节点树中不同图集的节点交叉排列。调整节点顺序比更换图集更有效。

10. **"Brotli 级别 11"**：Unity WebGL 发布必开 Brotli 压缩（级别 11），可将下载体积缩小 60-80%。唯一前提是 HTTPS + 服务器支持。

---

## 参考资源

- [Cocos Creator 3.8 - Compressed Textures](https://docs.cocos.com/creator/3.8/manual/en/asset/compress-texture.html)
- [Cocos Creator 3.8 - 2D Batching Guidelines](https://docs.cocos.com/creator/3.8/manual/en/ui-system/components/engine/ui-batch.html)
- [Cocos Creator - Pooling](https://docs.cocos.com/creator/2.4/manual/en/scripting/pooling.html)
- [Cocos Creator 3.8 - Native Performance Profiler](https://docs.cocos.com/creator/3.8/manual/en/advanced-topics/profiler.html)
- [Cocos Forum - DrawCall Performance Optimization](https://forum.cocosengine.org/t/an-introduction-to-draw-call-performance-optimization/55852)
- [Unity - Recommended Player Settings for WebGL](https://docs.unity3d.com/2022.3/Documentation/Manual/web-optimization-player.html)
- [Unity - Memory in Unity WebGL](https://docs.unity3d.com/2021.3/Documentation//Manual/webgl-memory.html)
- [Unity WebGL Compression Done Right](https://miltoncandelero.github.io/unity-webgl-compression)
- [10 Critical Unity WebGL Performance Tips](https://friendzy.xyz/2025/09/17/unity-webgl-performance-tips/)
- [WebGL-Memory Tracker](https://github.com/greggman/webgl-memory)
- [Chrome DevTools - Analyze Runtime Performance](https://developer.chrome.com/docs/devtools/performance)
- [Chrome DevTools - Performance Extensibility API](https://developer.chrome.com/docs/devtools/performance/extension)
- [微信小游戏分包加载文档](https://developers.weixin.qq.com/minigame/dev/guide/base-ability/subPackage/useSubPackage.html)
- [fonttools subset documentation](https://fonttools.readthedocs.io/en/stable/subset/)
- [MDN - Lazy Loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Lazy_loading)
- [web.dev - Preload Critical Assets](https://web.dev/articles/preload-critical-assets)

---

> 本Skill由 [女娲 · Skill造人术](https://github.com/alchaincyf/nuwa-skill) 生成
> 创建者：[花叔](https://x.com/AlchainHust)
