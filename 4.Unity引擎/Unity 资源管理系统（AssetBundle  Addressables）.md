# Unity 资源管理系统（AssetBundle / Addressables）

> **面向人群**：有 Unity 基础、准备面试或想理解资源管理底层原理的开发者。
>
> **阅读方式**：前六章讲 Resources → AssetBundle → ResourceManager 的演进链路；第七章讲底层序列化原理；第八章讲 Addressables 自动化方案；第九章做方案对比与面试总结。

------

# 第一章 为什么需要资源管理

## 1.1 什么是游戏资源（Asset）

在 Unity 中，除了代码之外，游戏运行所需要的一切内容都称为**资源（Asset）**：模型、纹理、材质、动画、音频、视频、Prefab、Scene、Shader、ScriptableObject 等。这些资源最终都会被 Unity 序列化并打包到游戏中。

一个角色 Prefab 并不是独立文件，它通常依赖很多其它资源：

```
Player.prefab
├── Player.fbx
├── Body.mat ── Body.png
├── Hair.mat ── Hair.png
├── AnimatorController ── Animation Clips
└── Audio
```

所以一个 Prefab 本质上是一棵**资源依赖树（Dependency Graph）**。

## 1.2 游戏为什么需要资源管理？

假设一个小 Demo，总共只有 Player、Enemy、Map 三类资源，加起来 30MB。最简方案是：全部打进安装包，启动时全部加载 —— 完全没有问题。

但商业游戏的规模完全不同：120 个英雄、500 个皮肤、40 张地图、20000 张 UI 图片、5000 个特效、30000 个动画、10GB 音频。如果仍然全部打进安装包：

- **安装包 35GB**：玩家首次下载一小时，每次更新再下载 35GB。
- **内存爆炸**：全部加载可能从 8GB 飙到 12GB，触发 OOM 崩溃。
- **大量浪费**：100 个英雄中玩家可能只玩鲁班七号，为什么要加载所有英雄？

> **核心结论：资源应该按需加载，而不是全部加载。** 这就是资源管理系统存在的第一个原因。

## 1.3 一个现代游戏真正需要什么？

商业游戏的目标经常是矛盾的：
- **全部提前加载** → 进入游戏不卡，但启动特别慢、内存爆炸
- **需要时再加载** → 内存小，但第一次打开角色会卡顿（磁盘读取）

> **资源管理系统的本质：在启动速度、内存、磁盘 IO、网络下载之间寻找平衡。**

具体来说需要做到：启动快、安装包小、内存占用低、更新快、资源按需下载、资源按需释放。

## 1.4 Unity 最早是怎么解决的？

Unity 最初提供了 `Resources` 文件夹 —— 把资源放在 `Assets/Resources` 下，用 `Resources.Load<GameObject>("Hero")` 加载。很简单，但它解决了**加载问题**，却没有解决**管理问题**。

## 1.5 Resources 最大的问题

很多人说 "Resources 不能热更新"，这是结果，不是根因。真正的问题是：**Unity 根本不知道哪些资源该放进 Resources**。

Unity 会把整个 Resources 文件夹所有内容都打进安装包。哪怕 `Boss.prefab` 玩家一辈子打不到，也得一起下载。三个天然缺陷：
1. 无法按需下载
2. 无法按需更新
3. 所有资源强制进入安装包

随着项目变大，这三个问题越来越严重。

## 1.6 我们真正需要什么？

合理的方式是按阶段加载：安装游戏 → 下载核心资源 → 进入登录界面 → 下载大厅 UI → 选择英雄 → 下载英雄资源 → 进入地图 → 下载地图资源 → 退出地图 → 释放地图资源。

我们需要一种新的资源组织形式，能做到：不强制进安装包、可以单独下载/更新、按需加载/释放。为此，Unity 设计了 **AssetBundle**。

> **本章总结**：资源管理系统不是为了"方便加载"，而是为了让大型游戏在安装包大小、内存占用、加载速度和热更新之间取得平衡。

------

# 第二章 Resources 系统

## 2.1 Resources 到底是什么？

很多初学者以为 Resources 是 Unity 的资源管理系统。更准确的说法是：**Unity 提供的一种"资源索引 + 运行时加载"机制**。

Unity Build 时扫描 `Assets/Resources` 下的所有资源并建立索引。运行时 `Resources.Load<GameObject>("Hero")` 本质上就是：查 Resources 数据库 → 找到资源位置 → 读取数据 → 反序列化 → 生成 Unity Object。

> **Resources 本质：Build 时建立索引，运行时根据字符串查找资源。**

## 2.2 Resources.Load() 到底发生了什么？

很多人以为 `Resources.Load("Hero")` 就是打开文件然后读取。实际上 Unity 内部经历了完整的流程：

1. **查找索引**：根据字符串 "Hero" 在 Resources Database 中定位到 `Assets/Resources/Hero.prefab`
2. **判断缓存**：如果资源已在内存中，直接返回；否则继续
3. **读取数据**：从磁盘读取 Prefab 中保存的 Transform、MeshRenderer、Animator、Collider 等各种引用
4. **反序列化**：将 Serialized Data（Position/Rotation/Scale、Mesh GUID、Material GUID）恢复为真正的 Unity Object。这一步非常耗 CPU
5. **递归加载依赖**：Hero 依赖 Material，Material 依赖 Texture，Unity 一路递归直到依赖全部恢复

```csharp
// 同步加载
GameObject heroPrefab = Resources.Load<GameObject>("Hero");
// 异步加载（推荐）
ResourceRequest request = Resources.LoadAsync<GameObject>("Hero");
yield return request;
GameObject heroPrefab = request.asset as GameObject;
```

整个过程涉及：字符串查找 → 磁盘 IO → 反序列化 → 依赖恢复 → GPU 上传，可能需要几毫秒到几十毫秒。这也是为什么 `Resources.Load()` 不能频繁调用。

## 2.3 Instantiate 为什么也会卡？

Load 完只是资源进入内存，而 Instantiate 做的是复制 GameObject：Clone → Transform → Animator → Renderer → Awake() → OnEnable()。如果 Prefab 很复杂，Instantiate 仍然可能卡顿。

> **资源加载 ≠ 对象创建**，这是两个完全不同的过程。后面讲对象池时还会回到这里。

## 2.4 为什么官方越来越不推荐 Resources？

这是高频面试题。回答 "不能热更新" 只答对了结果，根因是五个设计缺陷：

1. **全局目录**：所有人都可以往里放，最后越来越大，没人知道哪些资源已不用、哪些还在引用
2. **全部进入安装包**：即使 100GB 也全部打进安装包，玩家必须全部下载
3. **无法做版本管理**：更新一个 Boss.prefab 意味着整个安装包重新 Build
4. **无法按需下载**：登录界面只需要 Logo + LoginUI + 几个按钮，但 Resources 要求全部安装
5. **无法控制生命周期**：什么时候卸载？谁负责？Resources 完全不知道

所以大型项目通常需要自己写 ResourceManager 来管理引用计数、缓存、异步加载、卸载。

## 2.5 Resources 最大的问题是什么？

**Resources 根本没有"资源管理"。** 它只提供 `Load() → 返回对象`，剩下的缓存、生命周期、依赖、版本、下载、更新、释放全部要自己做。

> Resources 更像**一个方便的加载 API**，而不是**完整的资源管理系统**。

## 2.6 通往 AssetBundle

如果我们重新设计，会希望资源可以拆包、单独下载/更新、异步加载、统计引用、自动释放。Resources 做不到，于是 Unity 在它之上设计了 AssetBundle。

> **AssetBundle 最大的思想：资源不再跟 Player 绑定，而是变成独立的数据包。**

------

# 第三章 AssetBundle 原理

## 3.1 为什么 AssetBundle 会出现？

Resources 的根本问题：所有资源都进入安装包 → 不能单独更新 → 不能按需下载 → 生命周期无法管理。

想一个问题：修改一张图片，如果资源跟程序绑定在一起，就要重新 Build + 重新发布整个安装包。对商业游戏不可接受。

> **为什么资源一定要跟程序一起发布？** Unity 工程师的答案是：程序负责运行，资源负责存储，两者解耦。于是 AssetBundle 出现了。

## 3.2 什么是 AssetBundle？

AssetBundle 不是 zip/rar 这种压缩包，而是 **Unity 自定义的资源序列化文件**。它本质上是一个 Unity Object 数据库，内部结构：

```
AssetBundle
├── Header（文件头）
├── Object Table（对象表）
├── Serialized Data（序列化数据：GameObject, Material, Texture, Animation, Mesh, Shader）
├── Dependency Info（依赖信息）
├── Resource Data（真正资源数据）
└── Metadata（元数据）
```

> **AssetBundle 更像一个 Unity Object 数据库，而不是一个普通压缩包。**

## 3.3 BuildPipeline 到底干了什么？

`BuildPipeline.BuildAssetBundles(...)` 可以理解为资源编译器，整个流程分五步：

1. **扫描资源**：找到所有需要打包的资源（Hero.prefab, Enemy.prefab, Sword.prefab...）
2. **分析依赖**：建立依赖树（Hero.prefab → Hero.mat → Hero.png），这一步至关重要 —— Manifest、公共包、Addressables 全部依赖这里
3. **序列化**：将 Transform、Mesh、Material、Animator 全部转换为 Binary Data（Position/Rotation/Scale 都写入 Serialized File）
4. **生成 Manifest**：记录 Bundle 之间的依赖关系（Hero.ab 依赖 Texture.ab, Animation.ab），运行时加载时 Unity 据此知道加载顺序
5. **生成 AssetBundle**：将对象数据 + 依赖信息 + Metadata 写入 `.ab` 文件

```csharp
// 完整打包脚本示例
using UnityEditor;
using System.IO;

public class AssetBundleBuilder
{
    [MenuItem("Assets/Build AssetBundles")]
    static void BuildAllAssetBundles()
    {
        string outputPath = "AssetBundles";
        if (!Directory.Exists(outputPath))
            Directory.CreateDirectory(outputPath);
        
        BuildPipeline.BuildAssetBundles(outputPath, 
            BuildAssetBundleOptions.ChunkBasedCompression, 
            BuildTarget.StandaloneWindows64);
        
        Debug.Log("AssetBundles build completed: " + outputPath);
    }
}
```

> **Manifest 最大的作用不是更新，而是记录依赖关系。** 运行时加载 Hero.ab 时，Unity 已知应先去加载 Texture.ab 和 Animation.ab。

## 3.4 AssetBundle 为什么支持热更新？

AssetBundle 是独立文件（Hero.ab, Monster.ab, UI.ab, Audio.ab）。修改 Hero.prefab 后 rebuild，只有 Hero.ab 发生变化。服务器替换 Hero.ab，客户端下载 Hero.ab，程序 Game.exe 完全不用重新安装 —— 这就是热更新。

## 3.5 为什么能按需下载？

100 张地图中玩家只玩 Map01，AssetBundle 方案只需下载 Map01.ab，其余 99 张不用下载。而 Resources 方案必须全部安装。

> **AssetBundle 真正改变的是资源发布方式，而不是 Load API。**

## 3.6 为什么能减少内存？

AssetBundle 不会自动减少内存。真正减少的是**同时存在内存里的资源数量**。进入地图 Load Map01，离开地图 Unload Map01，内存一直保持在 500MB 而非 8GB。

> **AssetBundle 真正提供的是生命周期控制能力。**

## 3.7 一个 AssetBundle 的完整生命周期

```
服务器 → 下载 → 磁盘 → LoadFromFile() → AssetBundle → LoadAsset() → Unity Object
→ Instantiate() → GameObject → Release() → Unload() → GC
```

注意区分三个层次：**AssetBundle**（资源容器）、**Asset**（Unity Object）、**GameObject**（场景实例），后面讲 ResourceManager 时会重点展开。

> **本章总结**：AssetBundle 最重要的不是"把资源打成包"，而是**让资源从程序中独立出来，成为可以单独构建、下载、更新、加载、释放的数据单元。** Resources 解决"如何加载资源"，AssetBundle 解决"如何管理资源"。

------

# 第四章 AssetBundle 的依赖关系（Dependency）

## 4.1 一个 Prefab 真的是一个文件吗？

Hero.prefab 引用了 Mesh、Animator、Material、Weapon、Audio——Material 又引用 Texture、Weapon 又引用 Weapon.prefab → Weapon.mat → Weapon.png。所以 Prefab 本质上是一张**资源依赖图**，而不是一个独立文件。

## 4.2 Unity 怎么知道这些依赖？

Unity 不靠文件名找资源，而是靠 **GUID**。

## 4.3 GUID 是什么？

每个资源文件旁边都有一个 `.meta` 文件。Unity 真正认识的是 `.meta` 里面的 GUID —— 整个项目唯一。任何资源引用 Hero 时，实际保存的是 GUID 而不是路径。

为什么？因为文件名可以改、路径可以改，但 GUID 不变，引用永远不会丢。

## 4.4 Unity 的依赖分析过程

BuildPipeline 分析 Player.prefab：发现引用了 Body.mat 的 GUID → 继续查 Body.mat → 发现引用了 Body.png 的 GUID → 继续查 Body.png → 直到没有新引用。最终形成 Player → Material → Texture 的依赖树。

> **这个过程本质就是 DFS（深度优先搜索）遍历资源引用图。** BuildPipeline 的核心工作就是图遍历。这和你刷的算法题本质一样：遍历所有资源 → DFS 找所有引用 → 建立有向图 → 输出依赖关系。AssetBundle 的依赖分析就是一个图遍历问题的工程化应用。

## 4.5 直接依赖 vs 间接依赖

- **直接依赖（Direct Dependency）**：Player.prefab 直接引用 Body.mat
- **间接依赖（Indirect Dependency）**：Player.prefab 没有直接引用 Body.png，但通过 Body.mat 间接依赖

AssetBundle 打包时两种依赖都会统计。

## 4.6 为什么会重复打包？

Hero.prefab 和 Enemy.prefab 都引用了 Common.mat → Common.png。如果分别打包为 Hero.ab 和 Enemy.ab，Common.png 会出现两份，安装包直接变大。这就是**重复打包（Duplicate Assets）**。

## 4.7 为什么 BuildPipeline 不自动去重？

Unity 无法判断你的设计意图 —— 你可能希望 Hero 和 Enemy 一起更新，也可能希望它们独立更新。所以默认按 Bundle 打包，是否抽公共资源交给开发者决定。

## 4.8 什么是 Common Bundle（公共包）？

将共享资源抽到 Common.ab，Hero.ab 和 Enemy.ab 都引用它，Texture 只保存一份：

```
Hero.ab ──→ Common.ab ──→ Texture
Enemy.ab ──→
```

## 4.9 Manifest 为什么知道依赖？

Hero 不包含 Texture 了，运行时怎么知道 Texture 在哪？答案在 `Hero.manifest`：`Dependencies: Common.ab`。加载 Hero 前，Unity 先去加载 Common.ab。

> **Manifest 真正记录的是 Bundle 与 Bundle 的依赖关系，不是 Prefab 或 Texture 之间的依赖。**

## 4.10 一个完整的依赖加载流程

```
Load Hero.ab → 读取 Manifest → 发现依赖 Common.ab → Load Common.ab
→ Load Hero.ab → Load Hero.prefab → Instantiate → GameObject
```

如果依赖链更长（如加载场景时），需要递归处理所有依赖。代码示例：

```csharp
IEnumerator LoadAssetWithDeps(string path, string assetName)
{
    // 1. 加载 Manifest
    AssetBundle manifestBundle = AssetBundle.LoadFromFile(
        Path.Combine(Application.streamingAssetsPath, "AssetBundles"));
    AssetBundleManifest manifest = 
        manifestBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
    
    // 2. 获取依赖列表并加载所有依赖 Bundle
    string bundleName = path.Substring(path.LastIndexOf('/') + 1);
    string[] deps = manifest.GetAllDependencies(bundleName);
    foreach (string dep in deps)
    {
        yield return AssetBundle.LoadFromFileAsync(
            Path.Combine(Application.streamingAssetsPath, dep));
    }
    
    // 3. 加载目标 Bundle
    AssetBundle bundle = AssetBundle.LoadFromFile(path);
    GameObject asset = bundle.LoadAsset<GameObject>(assetName);
    
    manifestBundle.Unload(false);
}
```

> **本章总结**：Unity BuildPipeline 遍历整个资源依赖图，并生成 Bundle 之间的依赖关系。GUID 保证引用不会因重命名而失效；BuildPipeline 根据 GUID 遍历；Manifest 保存 Bundle 间依赖；Common Bundle 解决重复打包；AssetBundle 不会自动去重，需要开发者规划。

------

# 第五章 AssetBundle 加载流程

> **这一章回答的是：一个 Prefab 从硬盘到屏幕，中间到底发生了什么？**

## 5.1 看似三行代码，背后四个层次

```csharp
AssetBundle ab = AssetBundle.LoadFromFile(path);
GameObject prefab = ab.LoadAsset<GameObject>("Hero");
Instantiate(prefab);
```

三行代码背后有四个不同层次，很多初学者会混为一谈：

```
磁盘(Hero.ab) → LoadFromFile() → AssetBundle对象(CPU) → LoadAsset()
→ UnityEngine.Object(CPU) → Instantiate() → Scene → Renderer → GPU
```

## 5.2 第一步：LoadFromFile() — 打开资源数据库

`LoadFromFile()` 真正做的是：打开文件 → 读取 Header → 读取 Metadata → 创建 AssetBundle 对象 → 建立对象索引。此时 Texture、Mesh、Prefab 大部分还没有真正反序列化。就像打开 SQLite 数据库但还没执行查询一样。

> **LoadFromFile() 更多的是打开一个"资源数据库"，而不是加载所有资源。**

## 5.3 第二步：LoadAsset() — 真正读取资源

`LoadAsset<GameObject>("Hero")`：查 Object Table → 找到 Hero → 读取二进制数据 → 反序列化 → 创建 UnityEngine.Object。此时只是创建了 Unity Object（GameObject/Material/Mesh/Texture2D），还**没有进入场景**。

## 5.4 第三步：Instantiate() — 从模板到实例

LoadAsset 返回的是 Prefab（模板），Instantiate 做的事是：复制 → Transform → Component → Hierarchy → Awake() → OnEnable()，最后场景里多了个 `Hero(Clone)`。

> **Prefab 始终存在，Instantiate 只是复制一份。**

## 5.5 一个容易混淆的问题：Texture 何时上 GPU？

```csharp
Texture2D tex = bundle.LoadAsset<Texture2D>("Body");
```

此时 Texture 只在 CPU 内存。真正上传 GPU 发生在 Renderer 执行 Draw Call 或第一次使用 Texture 时。**AssetBundle 只负责资源进入 CPU，GPU 上传属于 Renderer 系统。** 这两个模块职责不同。

## 5.6 Manifest 在运行时仍在工作

每次加载都可能依赖 Manifest。加载 Hero.ab 时：查 Manifest → 发现依赖 Texture.ab 和 Animation.ab → 先加载它们 → 再加载 Hero.ab。否则 Hero 里的 Material 引用根本找不到。

## 5.7 为什么 LoadFromFile 很快但 LoadAsset 可能慢？

`LoadFromFile` 几乎 1ms —— 快的只是**打开 Bundle**。真正耗时的是 `LoadAsset` 里面的：Deserialize、Texture Upload、Mesh Create、Animation Restore。1000 个 Prefab 的 LoadAsset 可能几十毫秒。

## 5.8 为什么要异步加载？

一张 4K Texture 的读取 + 反序列化 + 上传 GPU 可能 30ms。放在主线程会直接让 FPS 从 60 掉到 30 再掉到 10。所以 Unity 提供了异步 API：

```csharp
IEnumerator LoadAssetAsync()
{
    // 异步加载 Bundle
    AssetBundleCreateRequest bundleRequest = 
        AssetBundle.LoadFromFileAsync(path);
    yield return bundleRequest;
    AssetBundle bundle = bundleRequest.assetBundle;
    
    // 异步加载资源
    AssetBundleRequest assetRequest = 
        bundle.LoadAssetAsync<GameObject>("Hero");
    yield return assetRequest;
    GameObject hero = assetRequest.asset as GameObject;
    
    Instantiate(hero);
}
```

异步加载本质就是让 IO、反序列化不阻塞主线程。

## 5.9 完整加载生命周期（建议记住）

```
Build → Hero.prefab → Hero.ab
═══════════════════════════════
运行时:
LoadFromFile() → AssetBundle → LoadAsset() → UnityEngine.Object
→ Instantiate() → Scene Object → Renderer Draw → Upload GPU → 屏幕显示
```

## 5.10 常见面试题

> **Q1：LoadFromFile 和 LoadAsset 的区别？**
> `LoadFromFile` 打开 AssetBundle 文件并建立索引；`LoadAsset` 才真正读取指定资源并反序列化生成 Unity 对象。

> **Q2：为什么 Instantiate 还会卡？**
> 因为 Instantiate 需要创建 GameObject、所有 Component、建立 Transform 层级、调用 Awake、调用 OnEnable，不是简单复制指针。

> **Q3：为什么 AssetBundle 可以异步？**
> 真正耗时的是 IO、反序列化、GPU 上传。异步就是让这些步骤不阻塞主线程。

> **本章总结**：理解四个层次 —— AssetBundle（资源容器）→ LoadAsset（资源对象）→ Instantiate（场景实例）→ Renderer（渲染）→ GPU（显示）。面试时把这几个阶段分开讲，不仅有助于回答问题，也能帮你定位实际性能瓶颈。

------

# 第六章 ResourceManager 设计

> **如果让你设计一个商业项目 ResourceManager，你会怎么做？**

## 6.1 为什么还需要 ResourceManager？

AssetBundle 只提供"打开 Bundle → 读取资源"。但商业项目还需要解决：资源已加载了吗？有没有缓存？引用次数多少？依赖 Bundle 加载了吗？什么时候释放？100 个人同时加载怎么办？这些 AssetBundle 全都不管。

所以大型项目需要封装一层：

```
Game → ResourceManager → AssetBundle → FileSystem
```

> **AssetBundle 只是底层驱动，ResourceManager 才是资源管理的核心。**

## 6.2 最简 ResourceManager — 资源缓存

```csharp
// 最简实现
Dictionary<string, UnityEngine.Object> _cache = new();

public T Load<T>(string path) where T : UnityEngine.Object
{
    if (_cache.TryGetValue(path, out var obj))
        return obj as T;
    
    // 实际加载逻辑...
    var asset = LoadFromBundle<T>(path);
    _cache[path] = asset;
    return asset;
}
```

流程：Load() → 查 Cache → 命中则直接返回 → 未命中则 LoadAsset → 放入 Cache → 返回。这就是**资源缓存（Resource Cache）**。

## 6.3 为什么必须缓存？

如果 UI 每帧都 `Resources.Load("Button")` 而没有缓存，每帧都是磁盘 IO + 反序列化 + 创建对象，CPU 直接爆炸。商业项目几乎所有资源都会缓存。

## 6.4 但缓存又会带来新问题

一直缓存 → 5000 个资源全部留在内存 → OOM。缓存不能永远存在，必须能够释放。

## 6.5 引用计数（Reference Count）

UI 引用 Hero、Battle 也引用 Hero → Hero 引用计数 = 2。UI 关闭 → Ref=1。战斗结束 → Ref=0 → ResourceManager 执行 Unload()。

核心思想：**谁用谁加，不用就减，归零释放。**

## 6.6 为什么不立刻释放？

Hero 现在没人引用，但 10ms 后又进入战斗。立刻 Unload 会导致 Unload → Load → Unload → Load 的疯狂 IO。商业项目一般不会立刻释放，而是放入 **Unused Queue**，等待几十秒或几分钟再回收。

## 6.7 真正的缓存结构

真正缓存的是 `ResourceItem` 而不是裸 `Object*`，因为需要附带引用计数、加载状态、最后访问时间、依赖关系等元数据：

```csharp
public class ResourceItem
{
    public string Path;                          // 资源路径
    public UnityEngine.Object Asset;             // 资源对象
    public AssetBundle Bundle;                   // 所属 Bundle
    public int RefCount;                         // 引用计数
    public bool IsLoading;                       // 是否正在加载
    public bool IsPermanent;                     // 是否常驻内存
    public float LastAccessTime;                 // 最后访问时间
    public List<ResourceItem> Dependencies;      // 依赖项列表
    public List<System.Action<UnityEngine.Object>> PendingCallbacks; // 等待回调
}

public class ResourceManager
{
    private Dictionary<string, ResourceItem> _cache = new();
    
    public T Load<T>(string path) where T : UnityEngine.Object
    {
        if (_cache.TryGetValue(path, out var item))
        {
            item.RefCount++;
            item.LastAccessTime = Time.time;
            return item.Asset as T;
        }
        // ... 异步加载逻辑
    }
    
    public void Unload(string path)
    {
        if (_cache.TryGetValue(path, out var item))
        {
            item.RefCount--;
        }
    }
}
```

## 6.8 为什么要记录依赖？

Hero.ab 依赖 Common.ab → Hero 引用 Texture（来自 Common.ab）。Hero 释放后，Common 能不能释放？不能确定 —— Enemy 可能也在引用 Common.ab。所以 ResourceManager 必须统计依赖关系，否则很容易提前释放导致 Missing Texture。

## 6.9 为什么需要 Loading 状态？（重复请求合并）

三个地方同时调用 `Load("Hero")`，如果没有 Loading 标记，三个线程都会执行 LoadFromFile → LoadAsset → Deserialize，Hero 被加载三次。正确做法是 `Loading=true` 时后续请求全部加入等待列表，只加载一次，完成后一起回调。

```csharp
public void LoadAsync<T>(string path, System.Action<T> callback) where T : UnityEngine.Object
{
    if (_cache.TryGetValue(path, out var item))
    {
        if (item.Asset != null)
        {
            callback(item.Asset as T);  // 已加载，直接回调
            return;
        }
        if (item.IsLoading)
        {
            item.PendingCallbacks.Add(obj => callback(obj as T));  // 等待
            return;
        }
    }
    // ... 开始实际加载
}
```

## 6.10 一个商业项目 ResourceManager 的完整分层

```
Game → ResourceManager → Cache → ReferenceManager → AsyncLoader
→ BundleManager → Manifest → AssetBundle → Disk
```

各层职责：
- **BundleManager**：Bundle 的 Load / Unload
- **Cache**：Object 级别的缓存
- **ReferenceManager**：引用计数 ++ / -- / 释放决策
- **AsyncLoader**：异步任务合并、排队、完成回调
- **ManifestManager**：Bundle 间的依赖关系

> **面试题：为什么商业项目一定要写 ResourceManager？**
> AssetBundle 只负责资源打包和加载，不负责生命周期管理。商业项目需要在 AssetBundle 上封装 ResourceManager，统一管理缓存、引用计数、依赖关系、异步加载、资源释放，避免重复加载和内存泄漏。

> **本章总结**：ResourceManager 本质上是一个围绕资源生命周期构建的管理中心。AssetBundle 解决"如何打包和读取"，ResourceManager 解决"什么时候加载、复用、释放"。一个成熟的 ResourceManager 至少需要：资源缓存、引用计数、依赖管理、异步加载、重复请求合并、卸载策略。

------

# 第七章 Unity 序列化系统（整个资源系统的基础）

> **本章不是面试必问，但理解后你整个 Unity 资源管理体系会突然串起来。**

## 7.1 Unity 为什么能保存一个 Prefab？

一个 Cube 包含 Transform、MeshFilter、MeshRenderer、BoxCollider。点击 Create Prefab 生成 `Cube.prefab`。真正的问题是：**Unity 怎么把内存里的 GameObject 保存到磁盘？** 这个过程就叫**序列化（Serialization）**。

## 7.2 什么是序列化？

> **把内存对象转换成可以保存的数据。** 反过来，从数据恢复内存对象就是**反序列化（Deserialize）**。

内存里的 `class Player { int hp=100; float speed=5; }` 关机就没了。必须变成 `hp=100, speed=5` 写进 `Player.asset`，下次打开 Unity 再恢复。

## 7.3 Unity 保存的是数据，不是代码

Prefab 里面没有代码。`void Update()` 这些已经编译进 `Assembly-CSharp.dll`。Prefab 真正保存的是字段：`Player → hp=100`。

> **Prefab 保存的是数据（Data），不是逻辑（Logic）。** 这也是 Unity 的设计思想：**Data Driven（数据驱动）**。

## 7.4 为什么用 GUID 而非文件路径？（回顾 §4.3）

保存 `Assets/Texture/Body.png` 作为引用 → 改名为 Body2.png → 全部引用失效。Unity 的做法是用 GUID（`9af2d9...`），改名字、改目录、移动文件夹都无所谓，引用基于 GUID 不会丢。

## 7.5 .meta 文件为什么必须提交 Git？

Unity 真正认识的是 `.meta` 文件，所有资源引用的是 `.meta` 里的 GUID。如果 `.gitignore *.meta`，GUID 会丢失导致 Unity 重新生成新 GUID → 所有 Prefab 引用全部 Missing。**团队开发中 .meta 必须提交。**

## 7.6 AssetBundle 的本质

BuildPipeline 做的事就是：Unity Object → Serialize → Binary → AssetBundle。LoadAsset 做的事是反过来：Binary → Deserialize → Unity Object。

> **AssetBundle 不是图片压缩包，它本质上是 Unity 序列化对象数据库。**

## 7.7 Prefab 为什么可以引用另一个 Prefab？

Hero.prefab 引用 Weapon.prefab —— Hero 里面保存的是 Weapon 的 GUID + FileID。加载 Hero 时，Unity 根据 GUID → FileID 恢复 Weapon。**Prefab 其实就是一张 GUID 图。**

## 7.8 为什么 Addressables 能自动分析依赖？

Addressables 也是扫描 GUID → 建立 Dependency Graph → Build。它没有发明新的依赖系统，它只是利用了 Unity 已经存在的 GUID 机制。

> **本章总结**：Unity 所有资源本质上都是可序列化对象，资源之间通过 GUID 建立引用关系；BuildPipeline 遍历资源依赖图生成 AssetBundle；运行时根据这些信息反序列化恢复对象。理解了这一点，`.meta` 的重要性、GUID 不能乱改、AssetBundle 不是 ZIP、Manifest 为什么知道依赖、Addressables 为什么能自动管理依赖 —— 这些看似独立的知识全都是建立在同一个设计之上。

------

# 第八章 Unity Addressables 系统

> **前面的章节讲的是"手动挡"，Addressables 就是 Unity 官方提供的"自动挡"。**

## 8.1 为什么还需要 Addressables？

AssetBundle 已经解决了资源管理的核心问题，但手动管理 AssetBundle 非常痛苦：

- **手动规划分包策略**：哪些资源放哪个 Bundle？谁和谁是公共依赖？
- **手动处理依赖加载**：加载资源前，必须先加载它的所有依赖 Bundle
- **手动管理生命周期**：加载/卸载时机、引用计数都要自己写
- **手动处理热更新**：需要自己写版本比对、下载、校验逻辑
- **资源引用方式不方便**：用字符串路径，重构时容易出错

Addressables 的目标就是**让这些手动工作自动化**。

## 8.2 Addressables 是什么？

> **Addressables 是 Unity 官方在 AssetBundle 之上构建的自动化资源管理层。** 底层仍然是 AssetBundle，但上层提供了自动依赖分析、自动打包、引用计数、异步加载等基础设施。

核心概念：
- **Address（地址）**：每个资源有一个唯一地址（可自定义），通过地址加载而非路径。类似网页 URL 的概念
- **Group（分组）**：资源按组管理，每个 Group 可以配置独立的打包和加载策略
- **Label（标签）**：可以给资源打标签，按标签批量加载/卸载
- **AssetReference**：强类型引用，类似 `[SerializeField]` 但在 Inspector 中直接引用 Addressables 资源，支持重构

```csharp
// 直接引用（避免字符串拼写错误）
public AssetReferenceGameObject heroPrefabRef;
public AssetReferenceTexture2D iconRef;
```

## 8.3 Addressables 和 AssetBundle 的关系

```
┌─────────────────────────────────┐
│         Addressables            │  ← 自动化管理层
│  (Address / Group / Label /     │
│   AssetReference / Catalog)     │
├─────────────────────────────────┤
│         AssetBundle             │  ← 底层打包与加载
│  (Bundle 打包 / 依赖分析 /      │
│   LoadFromFile / LoadAsset)     │
└─────────────────────────────────┘
```

Addressables 并不替代 AssetBundle —— 它就是在 AssetBundle 之上加了一层自动化。你仍然可以理解底层 AssetBundle 的行为，但不必手写所有管理代码。

## 8.4 Group 与 Schema 的设计思路

每个 Group 通过 **Schema** 配置行为：
- **Content Packaging & Loading Schema**：控制资源打进哪个包、从哪里加载（本地 / 远程）
- **Content Update Restriction Schema**：控制热更新时哪些资源可以变更
- **Build and Load Paths**：本地构建路径 vs 远程加载 URL

典型分组策略：
```
Built In Data      → 打进安装包（启动必需资源）
Local Assets       → 本地 StreammingAssets
Remote Assets      → CDN 远程下载
Interactive Assets → 按需从远程加载的资源
```

## 8.5 加载 API

```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

// 1. 通过 Address 加载
Addressables.LoadAssetAsync<GameObject>("Hero").Completed += handle =>
{
    if (handle.Status == AsyncOperationStatus.Succeeded)
        Instantiate(handle.Result);
};

// 2. 通过 AssetReference 加载（推荐，避免字符串硬编码）
public AssetReferenceGameObject heroRef;
heroRef.LoadAssetAsync().Completed += handle =>
{
    Instantiate(handle.Result);
};

// 3. 通过 Label 批量加载
Addressables.LoadAssetsAsync<GameObject>("Enemies", null).Completed += handle =>
{
    foreach (var enemy in handle.Result)
    {
        // 处理所有敌人 Prefab
    }
};

// 4. 场景加载
Addressables.LoadSceneAsync("Level01", LoadSceneMode.Single);

// 5. 实例化（一步到位：加载 + Instantiate）
Addressables.InstantiateAsync("Hero").Completed += handle =>
{
    // handle.Result 已经是场景中的 GameObject
};
```

## 8.6 内存管理与释放

Addressables 内置了引用计数。关键规则：**每个 Load/Instantiate 都要有对应的 Release**。

```csharp
// 加载
var handle = Addressables.LoadAssetAsync<GameObject>("Hero");
yield return handle;
GameObject heroPrefab = handle.Result;

// ... 使用完后释放
Addressables.Release(handle);  // 引用计数 -1

// Instantiate 的释放
var instHandle = Addressables.InstantiateAsync("Hero");
yield return instHandle;
// ... 使用完后释放实例
Addressables.ReleaseInstance(instHandle);  // Destroy + 释放引用
```

常见踩坑：忘记 Release 导致内存泄漏；过早 Release 导致资源被卸而引用还在。

## 8.7 依赖分析与去重

Addressables 通过 **Analyze** 工具自动分析依赖：
- 扫描所有标记的资源 → 建立依赖图 → 检测重复资源 → 建议优化方案
- **Fix Duplicate Asset Dependencies** 规则：自动检测被打入多个 Bundle 的资源，建议将它们抽到公共 Group

## 8.8 热更新方案

Addressables 热更新流程：

1. **Build** 时生成 `catalog_xxx.hash` 和 `catalog_xxx.json`
2. **运行时**检测服务器上的 catalog hash 与本地的差异
3. **有更新** → 下载新 catalog → 比对资源 hash → 下载变化的 Bundle
4. **无更新** → 使用本地缓存

```csharp
// 检查更新
IEnumerator CheckForUpdates()
{
    AsyncOperationHandle<List<string>> checkHandle = 
        Addressables.CheckForCatalogUpdates(false);
    yield return checkHandle;
    
    if (checkHandle.Result.Count > 0)
    {
        // 有更新，下载
        var updateHandle = Addressables.UpdateCatalogs(checkHandle.Result, false);
        yield return updateHandle;
        
        // 获取下载大小
        var sizeHandle = Addressables.GetDownloadSizeAsync(
            checkHandle.Result);
        yield return sizeHandle;
        Debug.Log($"Download size: {sizeHandle.Result} bytes");
        
        // 开始下载
        var downloadHandle = Addressables.DownloadDependenciesAsync(
            checkHandle.Result);
        yield return downloadHandle;
    }
}
```

远程分发方案：
- **Unity CCD（Cloud Content Delivery）**：官方方案，集成度高
- **自定义 CDN**：自己搭建服务器，更灵活（AWS S3、阿里云 OSS 等）

## 8.9 Addressables vs 手动 AssetBundle 对比

| | 手动 AssetBundle | Addressables |
|---|---|---|
| 打包策略 | 手写规则 | 自动分析 + Group |
| 依赖管理 | 手写 Manifest 加载 | 自动处理 |
| 引用计数 | 自己实现 | 内置 |
| 热更新 | 自己写比对/下载 | 内置 CheckForUpdates |
| 资源引用 | 字符串路径 | Address + AssetReference |
| 灵活性 | 高 | 中（受框架约束） |
| 学习成本 | 低（但开发成本高） | 中 |
| 适用阶段 | 理解底层原理 | 实际项目开发 |

> **本章总结**：Addressables 不是 AssetBundle 的替代品，而是 Unity 官方在 AssetBundle 之上提供的自动化管理方案。理解 AssetBundle 底层原理后，使用 Addressables 会更得心应手——你知道它的"自动挡"背后到底在做什么。

------

# 第九章 方案对比与面试总结

## 9.1 三种方案对比

| 维度 | Resources | AssetBundle | Addressables |
|---|---|---|---|
| 热更新 | ❌ | ✅ 手动 | ✅ 内置 |
| 按需下载 | ❌ | ✅ | ✅ |
| 依赖管理 | ❌ | 手动 Manifest | ✅ 自动 |
| 生命周期控制 | ❌ | 需自建 ResourceManager | ✅ 内置引用计数 |
| 资源引用方式 | 字符串路径 | 字符串路径 | Address + AssetReference |
| 内存控制 | ❌ | ✅（手动） | ✅（半自动） |
| 项目启动成本 | 极低 | 高 | 中 |
| 适用项目规模 | Demo / 小游戏 | 商业项目（需自建管线） | 中大型项目（推荐） |

## 9.2 演进路线与适用场景

```
小 Demo / 原型        中型项目               商业大作
    │                    │                      │
Resources ──→ AssetBundle + 自建RM ──→ Addressables（底层仍是AB）
```

- **Resources**：适合 Jam / 原型 / 学习阶段，发布前应迁移
- **AssetBundle + 自建 ResourceManager**：老项目 / 需要极致控制的项目
- **Addressables**：2024+ 新项目的推荐选择

## 9.3 高频面试题汇总

> **Q: Resources 和 AssetBundle 的区别？**
> Resources 是 Build 时索引 + 运行时加载 API，缺乏生命周期管理、不能热更新、不能按需下载。AssetBundle 将资源打包为独立文件，支持热更新、按需下载，资源与程序解耦。

> **Q: AssetBundle 的依赖是如何管理的？**
> BuildPipeline 通过 GUID 遍历资源依赖图（本质是 DFS），生成 Manifest 记录 Bundle 间依赖关系。运行时根据 Manifest 先加载依赖 Bundle 再加载目标 Bundle。

> **Q: Manifest 的作用是什么？**
> 记录 AssetBundle 之间的依赖关系。运行时加载资源前先读 Manifest，确保所有依赖 Bundle 先被加载。

> **Q: 如何避免 AssetBundle 重复打包？**
> 使用 Common Bundle（公共包）将共享资源抽离。或使用 Addressables 的 Analyze 工具自动检测和去重。

> **Q: 为什么商业项目需要自己的 ResourceManager？**
> AssetBundle 只负责底层加载，ResourceManager 封装缓存、引用计数、异步加载合并、依赖管理、卸载策略等生命周期管理。裸用 AssetBundle 在大型项目中会导致重复加载、内存泄漏等问题。

> **Q: Addressables 和 AssetBundle 是什么关系？**
> Addressables 是 AssetBundle 的上层自动化封装。底层仍是 AssetBundle，上层提供自动依赖分析、打包策略、引用计数、热更新检测等基础设施。

> **Q: 资源加载的卡顿可能发生在哪些阶段？**
> LoadFromFile（磁盘 IO）→ LoadAsset（反序列化，CPU 密集）→ Instantiate（创建对象、Awake/OnEnable）→ 首次渲染（GPU 上传）。需要使用异步 API 和 Profiler 定位具体瓶颈。

> **Q: 为什么 .meta 文件必须提交 Git？**
> .meta 文件包含 GUID，Unity 通过 GUID 而非文件路径来维护资源引用。丢失 .meta 会导致 GUID 重新生成，所有引用断裂。

---

> **全文总结**：Unity 资源管理系统的核心是一条演进链路——Resources 解决了加载问题但缺乏管理能力；AssetBundle 让资源独立于程序，成为可管理的数据单元；ResourceManager 在 AssetBundle 之上补全了生命周期管理；Addressables 则将这一切自动化。而底层贯穿始终的，是 Unity 的序列化系统和 GUID 引用机制。
