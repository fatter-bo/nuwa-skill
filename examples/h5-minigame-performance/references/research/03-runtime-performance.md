# 维度三：运行时性能

## 1. DrawCall 合批

### DrawCall 的性能影响
DrawCall 是 CPU 调用图形 API（OpenGL/WebGL）命令 GPU 绘制图形的操作。每次 DrawCall 伴随渲染状态切换开销，是 H5/小游戏的首要性能瓶颈。

### Cocos Creator 2D 合批条件（3.8+）
要实现同一 DrawCall 渲染多个 2D 组件，需满足以下全部条件：
1. **相同摄像机可见性**：兄弟节点在同一 Camera 中可见
2. **相同材质**：Material、Effect、Defines、Uniform 值完全一致
3. **相同混合/深度状态**：BlendState 和 DepthStencilState 一致
4. **共享顶点缓冲**：顶点数据通过相同 MeshBuffer 传输（v3.4.1+）
5. **相同纹理和采样器**：纹理来源和采样器类型一致

不可参与标准合批的组件：Mask、Graphics、UIMeshRenderer

来源: [Cocos Creator 3.8 - 2D Batching Guidelines](https://docs.cocos.com/creator/3.8/manual/en/ui-system/components/engine/ui-batch.html)

### 合批优化技巧

**图集整合（Atlas）**
- 使用 Auto Atlas 将同一界面的 Sprite 合并到一张图集
- Dynamic Atlas（动态图集）：引擎运行时自动将小纹理合并到大纹理
- 确保同一界面元素使用同一图集，避免跨图集节点交叉排列

**节点树排列**
- 将使用相同图集的节点相邻排列
- 避免不同图集的节点交叉（如：图集A的Sprite → Label → 图集A的Sprite 会打断合批）
- Label 节点尽量集中，或使用 BMFont/位图缓存使 Label 能与 Sprite 合批

**3D 合批方式**
| 方式 | 适用场景 | 优缺点 |
|------|---------|--------|
| 静态合批 | 不移动的物体 | 预合并网格，零运行时开销，增加内存 |
| 动态合批 | 移动的低面数物体 | 每帧重算顶点，增加 CPU 负担 |
| GPU Instancing | 大量相同网格 | 一次提交多实例 Transform，需 Shader 支持 |

来源: [Cocos Forum - DrawCall Optimization](https://forum.cocosengine.org/t/an-introduction-to-draw-call-performance-optimization/55852)

## 2. 节点树优化

### 节点数量控制
- 场景中活跃节点数直接影响引擎遍历和更新开销
- 不可见的 UI 面板使用 `node.active = false` 而非设置透明度为 0
- 避免深层嵌套节点树，扁平化结构减少遍历深度

### 按需激活
- 屏幕外的节点及时 deactivate
- 滚动列表只实例化可见项（虚拟滚动/回收列表）
- 远处或不可见的游戏对象暂停更新逻辑

## 3. 对象池（Object Pool）

### 为什么需要对象池
运行时频繁创建和销毁节点/组件实例效率极低，会引起 GC 抖动和帧率下降。对象池的 get/put 操作开销远低于 instantiate/destroy。

### Cocos Creator NodePool 最佳实践
1. **场景初始化时创建池**：在 onLoad 中预创建节点实例
2. **一种 Prefab 一个池**：不同 Prefab 使用独立 NodePool
3. **利用 reuse/unuse 回调**：重置节点状态（位置、缩放、组件数据）
4. **总是归还节点**：put() 自动从父节点移除，无需手动 removeFromParent
5. **合理池大小**：根据同屏最大数量设定池容量，而非总需求量

### 通用对象池实现
```typescript
class ObjectPool<T> {
    private pool: T[] = [];
    private factory: () => T;
    private reset: (obj: T) => void;

    constructor(factory: () => T, reset: (obj: T) => void, initSize: number) {
        this.factory = factory;
        this.reset = reset;
        for (let i = 0; i < initSize; i++) {
            this.pool.push(factory());
        }
    }

    get(): T {
        return this.pool.length > 0 ? this.pool.pop()! : this.factory();
    }

    put(obj: T): void {
        this.reset(obj);
        this.pool.push(obj);
    }
}
```

来源: [Cocos Creator - Pooling](https://docs.cocos.com/creator/2.4/manual/en/scripting/pooling.html)

## 4. 脏矩形渲染

### 原理
仅重绘画面中发生变化的区域（脏矩形），而非每帧全屏重绘。适用于静态内容较多的 2D 游戏（如棋牌、卡牌、解谜）。

### 实现策略
- 标记发生变化的节点，计算其包围盒作为脏区域
- 合并重叠的脏矩形以减少绘制次数
- 仅对脏区域内的节点执行渲染
- Cocos Creator 中可通过自定义渲染管线实现局部更新

### 适用场景
- 棋盘类游戏（只有被操作的棋子区域需要重绘）
- UI 密集型应用（大部分 UI 静止不动）
- 不适用于全屏动画、粒子密集的场景

## 5. Shader 优化

### 移动端 Fragment Shader 优化
- **避免 discard 和 alpha test**：会破坏 GPU 的 HSR（Hidden Surface Removal）机制
- **减少纹理采样次数**：合并多次采样为一次，使用纹理图集
- **避免复杂数学运算**：用查找表（LUT）纹理替代实时计算
- **使用低精度**：`lowp` / `mediump` 替代 `highp`（除 position 外）

### Overdraw 控制
- 半透明对象是 Overdraw 的主要来源
- 减少全屏后处理效果（Bloom、模糊）的使用
- 粒子特效控制发射数量和生命周期
- 利用 GPU 的 Early-Z 测试：先渲染不透明物体（前到后排序），再渲染半透明物体（后到前排序）

### 材质合并
- 自定义 Shader 会打断渲染合批
- 尽量使用引擎内置 Shader，共享材质实例
- 如需自定义效果，确保相同效果的节点使用同一 Material 实例

来源: [Cocos2d-x Optimizing Graphics](https://docs.cocos2d-x.org/cocos2d-x/v3/en/advanced_topics/optimizing.html)
来源: [Cocos Creator Shader](https://docs.cocos.com/creator/3.8/manual/en/shader/)
