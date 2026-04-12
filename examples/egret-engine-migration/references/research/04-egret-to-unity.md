# 维度4：Egret → Unity迁移

## 调研来源
- [egret-labs/egret3d-unityplugin](https://github.com/egret-labs/egret3d-unityplugin)
- [Unity UGUI Documentation](https://docs.unity3d.com/Manual/UISystem.html)
- [Unity Addressables](https://docs.unity3d.com/Manual/com.unity.addressables.html)
- [DragonBones Unity Runtime](https://github.com/DragonBones/DragonBonesJS)

## 核心发现

### 总体对比
| 维度 | Egret | Unity |
|------|-------|-------|
| 语言 | TypeScript | C# |
| 运行环境 | 浏览器/WebView | 原生运行时 |
| 架构 | 显示列表 | Entity-Component |
| UI系统 | EUI(EXML) | UGUI / UI Toolkit |
| 资源管理 | RES模块 | AssetBundle / Addressables |
| 渲染 | WebGL/Canvas | 原生GPU渲染 |
| 2D方案 | 内置2D引擎 | 2D Package(SpriteRenderer等) |

### TypeScript → C#转写策略

#### 类型系统映射
| TypeScript | C# | 注意事项 |
|-----------|-----|---------|
| `number` | `int` / `float` / `double` | 根据用途选择精度 |
| `string` | `string` | 基本一致 |
| `boolean` | `bool` | 关键字不同 |
| `any` | `object` | 尽量避免，用泛型替代 |
| `interface` | `interface` | C#接口需显式实现 |
| `enum` | `enum` | 基本一致 |
| `Array<T>` | `List<T>` | C#的List更接近TS的Array行为 |
| `Map<K,V>` | `Dictionary<K,V>` | API方法名不同 |
| `Promise<T>` | `Task<T>` / `async/await` | C#的异步模式类似但语法有差异 |
| `() => void` | `Action` / `Func<T>` | 委托类型替代箭头函数类型 |
| `可选参数?` | `参数 = default` | C#用默认值实现可选 |

#### 常见模式转换
```
// Egret TypeScript
class GameSprite extends egret.Sprite {
    private hp: number = 100;
    public constructor() {
        super();
        this.addEventListener(egret.TouchEvent.TOUCH_TAP, this.onClick, this);
    }
    private onClick(e: egret.TouchEvent): void {
        this.hp -= 10;
    }
}

// Unity C#
public class GameSprite : MonoBehaviour {
    private int hp = 100;
    void OnMouseDown() {  // 或用UI事件系统
        hp -= 10;
    }
}
```

### UI系统迁移（EUI → UGUI）

| EUI | UGUI | 迁移说明 |
|-----|------|---------|
| `eui.Button` | `Button + Text` | UGUI的Button需配合Image和Text子对象 |
| `eui.Label` | `Text` / `TextMeshPro` | 建议直接用TextMeshPro |
| `eui.Image` | `Image` / `RawImage` | UI用Image，非UI用SpriteRenderer |
| `eui.TextInput` | `InputField` | API和事件完全不同 |
| `eui.ProgressBar` | `Slider` / 自定义 | UGUI无原生ProgressBar，用Slider或Image.fillAmount |
| `eui.Scroller` | `ScrollRect` | 需配合Mask和Content使用 |
| `eui.List` | `ScrollRect + 自定义` | 需自行实现对象池和虚拟化 |
| `eui.Group + Layout` | `LayoutGroup` | HorizontalLayoutGroup / VerticalLayoutGroup / GridLayoutGroup |
| `eui.CheckBox` | `Toggle` | 概念一致 |
| `eui.TabBar` | `ToggleGroup` | 需手动组装 |
| EXML皮肤 | Prefab | Unity用Prefab替代EXML描述 |
| 数据绑定 | 无内置，需自实现或用MVVM框架 | EUI的数据绑定在Unity中无直接等价物 |

### 资源管理迁移（RES → Addressables）

| Egret RES | Unity Addressables | 说明 |
|-----------|-------------------|------|
| `default.res.json`(资源清单) | Addressables Groups(编辑器配置) | Unity在编辑器中管理资源分组 |
| `RES.loadGroup("preload")` | `Addressables.LoadAssetsAsync<T>(label)` | 按标签加载资源组 |
| `RES.getRes("name")` | `Addressables.LoadAssetAsync<T>(key)` | 按key加载单个资源 |
| `RES.getResAsync()` | `Addressables.LoadAssetAsync<T>()` | 全部异步 |
| 精灵图集(sheet) | SpriteAtlas | Unity有原生SpriteAtlas支持 |
| 资源名字符串引用 | Addressable Key / AssetReference | 可用字符串key或强类型引用 |

### 2D渲染体系差异
| 概念 | Egret | Unity 2D |
|------|-------|----------|
| 精灵显示 | `egret.Bitmap` | `SpriteRenderer`组件 |
| 渲染排序 | 显示列表顺序(addChild先后) | Sorting Layer + Order in Layer |
| 图形绘制 | `egret.Shape` + `graphics` | `LineRenderer` / 自定义Mesh |
| 粒子效果 | 第三方粒子库 | Particle System(强大得多) |
| 摄像机 | 无独立概念 | Camera组件（可多摄像机） |
| 物理 | P2.js等第三方 | Box2D(内置Physics2D) |

### 关键差异与陷阱
1. **继承 vs 组合** — Egret大量使用类继承(extends Sprite)，Unity用组件组合(AddComponent)，需重构代码结构
2. **坐标系** — Egret左上角原点Y向下，Unity中心原点Y向上（与Cocos迁移类似的问题）
3. **生命周期** — Egret用事件(ADDED_TO_STAGE等)，Unity用MonoBehaviour生命周期(Awake/Start/Update/OnEnable/OnDisable)
4. **帧循环** — Egret的`addEventListener(ENTER_FRAME)`对应Unity的`Update()`
5. **异步模型** — Egret用回调/Promise，Unity用Coroutine/async-await/UniTask
6. **资源同步vs异步** — Egret的RES.getRes()是同步的(预加载后)，Unity的Addressables全异步
7. **性能差异** — Unity原生运行远优于WebGL，可做更复杂的效果

### 迁移策略建议
1. **先做架构重设计** — 不要1:1翻译，按Unity的Component模式重新设计类结构
2. **UI重建** — 在Unity编辑器中用UGUI重建界面，EXML无法自动转换
3. **资源转换** — 图片资源直接导入Unity，精灵图集需重新配置SpriteAtlas
4. **DragonBones运行时** — Unity有DragonBones C#运行时，动画数据可直接复用
5. **逻辑分层迁移** — 将业务逻辑与引擎API调用分离，先迁移纯逻辑部分
