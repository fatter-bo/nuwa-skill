# 维度五：Unity WebGL 特有优化

## 1. WASM 编译优化

### IL2CPP 转换流程
Unity WebGL 的编译链路：C# → IL2CPP → C++ → Emscripten → WebAssembly (WASM)

每一步都有优化空间：
- C# 层：减少泛型使用（每个泛型实例化会生成独立 C++ 代码）
- C++ 层：IL2CPP 设置影响生成代码质量
- WASM 层：Emscripten 编译选项影响最终产物大小和执行效率

### Build 模式选择
| 选项 | 编译速度 | 产物大小 | 运行性能 |
|------|---------|---------|---------|
| Faster (smaller) builds | 快 | 小 | 略低 |
| Shorter build time | 最快 | 较大 | 低 |
| Release (LTO) | 慢 | 最小 | 最高 |

推荐：开发期使用 "Shorter build time"，发布使用 "Faster (smaller) builds" 或 "Release"。

来源: [Unity - Recommended Player settings for WebGL](https://docs.unity3d.com/2022.3/Documentation/Manual/web-optimization-player.html)

### WASM 体积优化
- 启用 Code Stripping（Managed Stripping Level: Medium/High）
- 移除未使用的引擎模块（Physics、Audio、Animation 等）
- 使用 `link.xml` 保护反射调用的类型
- 避免使用 LINQ（会生成大量 IL 代码）
- 避免使用 `System.Reflection`（会阻止代码裁剪）

## 2. IL2CPP 设置

### 关键设置项

**Enable Exceptions**
| 设置 | 说明 | 建议 |
|------|------|------|
| None | 不支持异常处理 | 发布版使用，性能最佳，产物最小 |
| Explicitly Thrown Only | 仅支持显式 throw | 开发期使用 |
| Full Without Stacktrace | 完整异常但无堆栈 | 需要异常处理的生产版本 |
| Full With Stacktrace | 完整异常 + 堆栈 | 仅调试使用，产物大幅增加 |

**C++ Compiler Configuration**
- Release 模式：启用 LTO（Link Time Optimization），减小产物并提升性能
- Debug 模式：禁用优化，加快编译速度

### 代码层面优化
```csharp
// 避免：频繁 GetComponent 调用
void Update() {
    GetComponent<Renderer>().material.color = Color.red; // 每帧反射查找
}

// 推荐：缓存组件引用
private Renderer _renderer;
void Awake() {
    _renderer = GetComponent<Renderer>();
}
void Update() {
    _renderer.material.color = Color.red;
}

// 避免：string 拼接（产生大量 GC）
string info = "HP: " + hp + " / " + maxHp;

// 推荐：StringBuilder
StringBuilder sb = new StringBuilder();
sb.Append("HP: ").Append(hp).Append(" / ").Append(maxHp);
string info = sb.ToString();
```

来源: [10 Critical Unity WebGL Performance Tips](https://friendzy.xyz/2025/09/17/unity-webgl-performance-tips/)

## 3. Brotli/Gzip 压缩选型

### 压缩格式对比
| 特性 | Brotli | Gzip |
|------|--------|------|
| 压缩率 | 更优（约比 Gzip 小 15-25%） | 良好 |
| 解压速度 | 略慢 | 快 |
| 浏览器支持 | 主流浏览器均支持 | 全面支持 |
| 服务器支持 | 需确认服务器配置 | 几乎所有服务器 |
| HTTPS 要求 | Chrome/Firefox 仅 HTTPS 下支持 | 无要求 |
| 压缩级别 | 0-11（推荐 11 用于静态资源） | 1-9 |

### 选型决策
```
服务器支持 Brotli 且使用 HTTPS？
├── 是 → 使用 Brotli（压缩级别 11）
│   ├── JS/WASM/Data 文件使用 .br 后缀
│   └── 需配置正确的 Content-Encoding: br
└── 否 → 使用 Gzip
    ├── 配置 Content-Encoding: gzip
    └── 确保服务器正确发送 .gz 文件
```

### 实际效果
- Brotli 压缩可将构建数据缩小 60-80%
- 初始下载从 10+ 秒缩短到 3 秒以内（典型宽带）
- JS + WASM + Data 三大文件都应压缩

来源: [Unity WebGL compression done right](https://miltoncandelero.github.io/unity-webgl-compression)
来源: [The Impact of Compression on Unity WebGL Game Performance](https://moldstud.com/articles/p-the-impact-of-compression-on-unity-webgl-game-performance-boosting-efficiency-and-load-times)

### 服务器配置示例（Nginx）
```nginx
# Brotli 预压缩
location ~ \.(js|wasm|data)$ {
    gzip off;
    # 尝试提供 .br 预压缩文件
    location ~ \.js$ {
        add_header Content-Type application/javascript;
        add_header Content-Encoding br;
        try_files $uri.br $uri =404;
    }
    location ~ \.wasm$ {
        add_header Content-Type application/wasm;
        add_header Content-Encoding br;
        try_files $uri.br $uri =404;
    }
    location ~ \.data$ {
        add_header Content-Type application/octet-stream;
        add_header Content-Encoding br;
        try_files $uri.br $uri =404;
    }
}
```

## 4. Unity WebGL 构建优化清单

### Player Settings
- [x] Compression Format: Brotli
- [x] Enable Exceptions: None（发布版）
- [x] Managed Stripping Level: Medium 或 High
- [x] Color Space: Gamma（比 Linear 性能更好）
- [x] 禁用 Auto Graphics API，手动选择 WebGL 2.0

### 资源优化
- [x] 纹理启用 Crunch Compression
- [x] 音频格式 Vorbis + Streaming 加载
- [x] 模型 LOD 设置
- [x] 使用 Texture Atlas 减少材质数量

### 代码优化
- [x] 缓存组件引用
- [x] 使用对象池
- [x] 避免频繁 GC（减少临时对象分配）
- [x] 使用 struct 替代 class 处理简单数据
- [x] 避免 LINQ、反射、动态代码生成

### 运行时
- [x] 设置合理的内存预算（256-512MB）
- [x] 使用 Addressable Assets 按需加载
- [x] 实现自定义 HTML/CSS Loading 界面
- [x] 监控内存使用，防止 Tab 崩溃

来源: [Unity - Optimize WebGL for mobile](https://docs.unity3d.com/2022.3/Documentation/Manual/web-optimization-mobile.html)
来源: [Eliot's WebGL Notes for 2026](https://discussions.unity.com/t/eliots-webgl-notes-for-2026/1701530)
