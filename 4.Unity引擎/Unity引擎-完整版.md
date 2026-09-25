# Unity 引擎知识体系 — 完整版

> 整合自 52 个专题文档，涵盖引擎基础、资产管理、UI系统、物理系统、动画系统、图形渲染、网络通信、性能优化等完整知识体系。

---

# 第一部分：引擎基础知识

## 一、五大坐标系

### 1. 全局坐标系（World Coordinate System）
用于描述场景内所有物体位置和方向的基准坐标系。使用 `transform.position` 获取世界坐标。

### 2. 局部坐标系（Local Coordinate System）
每个物体自身独立的坐标系。模型 Mesh 保存的顶点坐标均为局部坐标系下的坐标。

### 3. 本地坐标系（transform.localPosition）
- 子物体以父物体的坐标点为自身的坐标原点
- 无父物体时等同于全局坐标
- Inspector 视图中显示的为 localPosition 的值

### 4. 屏幕坐标系（Screen Space）
- 建立在屏幕上的二维坐标系，以像素为单位
- 左下角 (0,0)，右上角 (Screen.width, Screen.height)
- 鼠标位置：`Input.mousePosition`

### 5. 视口坐标系（ViewPort Space）
- 屏幕坐标系的标准化版本
- 左下角 (0,0)，右上角 (1,1)

---

## 二、向量计算：点乘与叉乘

### 点乘（Dot Product）
- **公式**：`A·B = |A| × |B| × cos(θ)`
- **用途**：判断夹角（正=锐角，零=直角，负=钝角）、投影长度、力在方向上的分量
- **API**：`Vector3.Dot(a, b)`

### 叉乘（Cross Product）
- **公式**：`A×B = |A| × |B| × sin(θ) × n`（n 为垂直单位向量）
- **用途**：求垂直向量、旋转轴、扭矩、法线
- **API**：`Vector3.Cross(a, b)`

### 归一化（Normalize）
- 消除长度信息，只保留方向
- **API**：`vector.normalized`

### 方向与旋转

| 目的 | 工具 | 示例 |
|------|------|------|
| 计算夹角 | `Vector3.Angle(dirA, dirB)` | 返回值 0~180° |
| 判断左右 | `Vector3.SignedAngle(fwd, targetDir, Vector3.up)` | 返回值 -180~180° |
| 设置旋转 | `transform.eulerAngles` 或 `Quaternion.Euler()` | 直接用欧拉角 |
| 看向目标 | `Quaternion.LookRotation(dirToTarget)` | 立即朝向 |
| 平滑旋转 | `Quaternion.Slerp(start, end, t)` | 球形插值 |
| 固定角速度 | `Quaternion.RotateTowards()` | 匀速转头 |
| 增量旋转 | `transform.rotation *= Quaternion.Euler(0,1,0)` | 每帧转1度 |

**核心原则**：用欧拉角设置值，用四元数做计算和动态旋转。

---

## 三、脚本后端：Mono vs IL2CPP

### JIT 与 AOT

| | AOT（预先编译） | JIT（即时编译） |
|------|------|------|
| **编译时机** | 程序启动之前 | 程序运行时 |
| **启动速度** | 快 | 首次执行有延迟 |
| **性能** | 稳定高效 | 可动态优化 |
| **安全性** | 更难反编译 | 较易反编译 |

### Mono
- 开源跨平台 .NET 实现
- 支持 JIT 编译
- iOS 平台受限（不允许 JIT）

### IL2CPP
- IL → C++ → 本机代码
- AOT 编译，性能提升 1.5-2x
- iOS 必须使用
- 不支持 `System.Reflection.Emit`

### 引擎架构层次
```
核心系统（渲染/物理/音频/输入/资源管理）
  ↕
抽象层（场景管理/对象组件系统）
  ↕
脚本系统（Mono/IL2CPP）
  ↕
编辑器层（Inspector/Asset Pipeline）
```

---

## 四、生命周期函数

### 执行顺序
```
Awake → OnEnable → Start → FixedUpdate → Update → LateUpdate → OnDisable → OnDestroy
```

### 各阶段说明

| 函数 | 调用时机 | 用途 |
|------|------|------|
| **Awake()** | 脚本实例加载时 | 初始化变量和组件 |
| **OnEnable()** | 对象变为可用时 | 订阅事件 |
| **Start()** | 首次帧更新之前 | 进一步初始化 |
| **FixedUpdate()** | 固定时间步（默认 0.02s） | 物理相关操作 |
| **Update()** | 每帧 | 游戏逻辑 |
| **LateUpdate()** | Update 之后 | 摄像机跟随 |
| **OnDisable()** | 对象不可用时 | 取消订阅 |
| **OnDestroy()** | 销毁时 | 清理资源 |

### 暂停与退出
- `OnApplicationPause(bool)`：应用暂停/恢复
- `OnApplicationFocus(bool)`：获得/失去焦点
- `OnApplicationQuit()`：应用退出前
- `Application.Quit()`：主动退出

---

## 五、常用组件与 API

### Transform
- Position（世界坐标）、Rotation（旋转）、Scale（缩放）
- `transform.Rotate()`：自身旋转
- `transform.RotateAround()`：绕点旋转

### GameObject
- `AddComponent<T>()`、`GetComponent<T>()`、`Destroy()`
- `DontDestroyOnLoad()`：场景切换不销毁

### Input
- `GetKeyDown(KeyCode)`、`GetMouseButtonDown(int)`
- `mousePosition`：屏幕坐标

### Camera
- `Camera.main`：主摄像机（避免每帧调用，应缓存）
- `ScreenPointToRay()`：屏幕坐标转射线

### PlayerPrefs
- `SetInt/GetInt`、`SetFloat/GetFloat`、`SetString/GetString`

### Mathf
- `Round` 四舍五入、`Clamp` 限制范围、`Lerp` 线性插值

### Destroy vs DestroyImmediate
- `Destroy`：帧结束后销毁，内存延迟释放
- `DestroyImmediate`：立即销毁并释放内存

---

## 六、射线检测

### 核心原理
射线（Ray）= 起点（Origin）+ 方向（Direction）。与带 Collider 组件的物体求交，返回 `RaycastHit` 碰撞信息。

### 基础用法
```csharp
if (Physics.Raycast(ray, out RaycastHit hit, maxDistance, layerMask)) {
    // hit.collider、hit.point、hit.normal、hit.distance
}
```

### 常用场景
- **摄像机到鼠标**：`Camera.main.ScreenPointToRay(Input.mousePosition)`
- **射击判定**：枪口位置发射线
- **地面检测**：`Physics.Raycast(transform.position, Vector3.down, 0.1f, groundLayer)`
- **视线遮挡**：AI 判断是否能看到玩家

### 穿模解决方案

| 方案 | 说明 |
|------|------|
| **连续碰撞检测（CCD）** | Rigidbody 设为 Continuous/ContinuousDynamic |
| **射线预检测** | 移动前发射射线预测碰撞 |
| **减小 Fixed Timestep** | 增加物理采样频率 |
| **增大碰撞体厚度** | 静态场景优化 |

### 性能优化
- 零 GC：`Physics.RaycastNonAlloc(ray, results, maxDistance)`
- 体积检测：`Physics.SphereCast`、`Physics.BoxCast`

---

## 七、协程

### 原理
协程 = C# IEnumerator 迭代器状态机 + Unity 主线程调度。**不是线程，始终在主线程执行。**

### 调度时机
- `yield return null`：Update 后、LateUpdate 前
- `yield return new WaitForSeconds(n)`：n 秒后
- `yield return new WaitForEndOfFrame()`：所有渲染完成后
- `yield return new WaitForFixedUpdate()`：FixedUpdate 后
- `yield return new WaitUntil(() => bool)`：条件为真时
- `yield return StartCoroutine(Child())`：等子协程完成

### 生命周期
- GameObject 被 Destroy → 协程自动停止
- `enabled = false` **不会**停止协程！
- `StopCoroutine()` / `StopAllCoroutines()` 手动停止

### 应用场景
- 延时执行（技能冷却）
- 动画序列控制（过场动画）
- 渐变效果（UI 淡入淡出）
- 分帧处理（地图生成、批量创建）
- 异步加载（`Resources.LoadAsync`）
- AI 行为序列

### 协程 vs 线程
| | 协程 | 线程 |
|------|------|------|
| 执行方式 | 主线程串行 | 并行执行 |
| 并发 | 无 | 有 |
| 线程安全 | 天然安全 | 需处理同步 |
| Unity API | 可调用 | 不可直接操作 GameObject |

---

# 第二部分：资产管理

## 一、Prefab 预制体

### 概念
Prefab 是 GameObject 及其组件的**可复用模板**（.prefab 文件），一次创建多次实例化，修改源预制体可同步更新所有实例。

### 优势
- 高效复用（子弹、敌人、道具等批量生成）
- 一致性维护（修改源 = 全部更新）
- 模块化开发（嵌套预制体）
- 对象池基础（减少 GC）

---

## 二、特殊文件夹

| 文件夹 | 打包 | 访问方式 | 用途 |
|------|------|------|------|
| **Resources** | ✅ 强制打包 | `Resources.Load()` | 动态加载预制体 |
| **StreamingAssets** | ✅ 原样拷贝 | `Application.streamingAssetsPath` | 视频/配置文件 |
| **Editor** | ❌ | 仅编辑器 | 编辑器扩展脚本 |
| **Plugins** | ✅ 平台适配 | 自动链接 | 原生库 |

### 持久化文件夹
| 文件夹 | 读写 | 数据保留 | 用途 |
|------|------|------|------|
| `persistentDataPath` | 读写 | 永久 | 存档、设置 |
| `streamingAssetsPath` | 只读 | 随应用 | 初始配置 |
| `temporaryCachePath` | 读写 | 可能被清理 | 临时缓存 |

---

## 三、AssetBundle

### 概念
Unity 资源打包技术，将资源序列化为二进制文件，支持动态加载与热更新。

### 工作流程
1. 设置 AB 包名 → 2. `BuildPipeline.BuildAssetBundles()` → 3. 上传服务器
4. 客户端下载 → 5. `AssetBundle.LoadFromFile()` 加载 → 6. `Unload(false)` 卸载

### 加载方式
- `AssetBundle.LoadFromFile(path)`：同步
- `AssetBundle.LoadFromFileAsync(path)`：异步（推荐）
- `UnityWebRequestAssetBundle`：网络加载 + 缓存

### 压缩算法

| 算法 | 特点 | 适用场景 |
|------|------|------|
| **LZMA** | 最小体积，需全量解压 | 初始安装包 |
| **LZ4** | 按块解压，内存友好 | 移动端热更新（推荐） |
| **不压缩** | 最快，体积最大 | 开发期/PC |

### 打包策略（颗粒度）
- **按逻辑实体**：场景包、角色包
- **按资源类型**：共享包（材质/Shader）、音频包
- **按更新频率**：常驻包 vs 动态包

### 卸载策略
- `Unload(false)`：仅卸载 AB 包，保留已加载资源
- `Unload(true)`：强制卸载所有（易导致资源丢失）

---

## 四、热更新方案

### 核心流程
```
资源标记 → 打包 AB 包 → 生成版本清单（MD5 + 大小）
    ↓
上传服务器
    ↓
客户端下载版本文件 → MD5 比对 → 差异下载
    ↓
优先从 persistentDataPath 加载 → AB 包解压 → 使用
```

### 关键点
- 版本控制：MD5/SHA256 校验
- 增量更新：仅下载变更文件
- 跨平台：不同平台独立 AB 包
- 加密：AES 加密防篡改

---

## 五、资源压缩策略

### 纹理
| 平台 | 推荐格式 |
|------|------|
| Android | ASTC（现代）/ ETC2 |
| iOS | ASTC / PVRTC |
| PC/主机 | BC7 (DXT5) |

### 音频
- 长音频：Vorbis 压缩（96-192kbps）
- 短音效：ADPCM（低 CPU 开销）

### 模型
- GLB（二进制 GLTF）：比 FBX 小 40%
- Draco 压缩：再减 30%-50%

---

# 第三部分：UI 系统

## 一、UGUI 基础

### 核心概念
- **Canvas**：UI 容器，支持三种渲染模式
- **RectTransform**：UI 元素的定位核心（锚点 + 位置 + 尺寸）
- **Canvas Scaler**：屏幕适配缩放

### Canvas 渲染模式

| 模式 | 说明 | 适用 |
|------|------|------|
| **Screen Space - Overlay** | 屏幕最上层，不受相机影响 | HUD、菜单 |
| **Screen Space - Camera** | 绑定指定相机 | 与特定相机关联 |
| **World Space** | 三维空间中的 UI | 3D UI、场景内交互 |

### Image vs RawImage
| | Image | RawImage |
|------|------|------|
| 资源类型 | Sprite | Texture2D |
| 颜色调整 | 内置 | 需 Shader |
| 灵活性 | 一般 | 高 |

---

## 二、锚点系统

锚点用于确定 UI 元素在父容器中的位置和缩放方式：
- **屏幕自适应**：锚点绑定到屏幕边缘，自动适配不同分辨率
- **固定区域定位**：控制相对于父容器的偏移和大小

---

## 三、屏幕适配

### 核心公式
```
摄像机实际宽度 = 摄像机 orthographicSize × 2 × 屏幕宽高比
```

### 适配原则
- 摄像机尺寸 ≥ 有效内容尺寸（避免裁切）
- 摄像机尺寸 ≤ 实际内容尺寸（避免黑边）

### UGUI 适配
- Canvas Scaler：`Scale With Screen Size`
- Reference Resolution：设计分辨率
- Match：0=宽度适配，1=高度适配

---

## 四、UI 事件穿透

### 方法对比

| 方法 | 操作 | 适用 |
|------|------|------|
| 关闭 Raycast Target | 取消勾选组件属性 | 纯装饰元素 |
| Canvas Group Blocks Raycasts | 取消勾选 | 整组穿透 |
| 手动传递事件 | 实现 IPointerClickHandler + ExecuteEvents | 先处理 UI 再穿透 |
| 添加 Physics Raycaster | 挂载到摄像机 | 穿透到 3D 物体 |

---

## 五、重建与重绘

| | 重建（Rebuild） | 重绘（Repaint） |
|------|------|------|
| **阶段** | CPU 端（生成网格数据） | GPU 端（绘制像素） |
| **触发** | UI 属性修改 | 网格数据更新 |
| **瓶颈** | CPU 耗时 | GPU Overdraw |

### 优化策略
- **动静分离**：动态 UI 与静态 UI 分属不同 Canvas
- **批量修改**：隐藏 → 更新 → 显示
- **避免高频刷新**：用事件驱动替代每帧更新

---

## 六、合批机制

### 四级排序
深度（Depth）→ 材质 ID → 纹理 ID → Hierarchy 顺序

### 合批条件
- 相同材质 + 相同纹理（或同一图集）
- 层级连续排列，无其他材质元素插入
- 无 Mask 等特殊组件打断
- Z 轴为 0，无旋转缩放

### 打断合批的因素
- 不同材质/纹理
- Mask 组件（额外增加 2 个 DrawCall）
- 动态修改 `Image.color`（创建新材质实例）
- 不同渲染队列

### 优化建议
- 用 `RectMask2D` 替代 `Mask`
- 同功能 UI 打成同一图集
- 相同材质元素在 Hierarchy 中连续排列

---

## 七、图集（Sprite Atlas）

### 为什么打图集能减少 DrawCall
- 合并纹理：多元素共享同一纹理
- 材质复用：相同材质才能合批
- 静态合批：连续排列自动合并

### 打图集策略
- 单图集 ≤ 2048×2048（移动端）
- 图片大小为 2 的幂次方
- 同功能 UI 放同一图集
- 高频更新元素与静态元素分离

### 常见图集划分
| 图集名 | 内容 | 说明 |
|------|------|------|
| `UIAtlas_Common` | 通用按钮、面板背景 | 所有界面共享 |
| `UIAtlas_Home` | 主界面 UI | 按功能拆分 |
| `UIAtlas_Fight` | 战斗 UI | 场景专用 |

---

# 第四部分：物理系统

## 一、物理引擎组成

Unity 包含 3D 物理和 2D 物理两个独立引擎，核心组件：

### 1. Rigidbody（刚体）
- 赋予物体重力、碰撞反馈等物理属性
- Is Kinematic：从引擎控制中移除，由脚本控制
- Sleeping：速度低于阈值时睡眠，节省性能

### 2. Collider（碰撞器）
- 常用类型：Box、Sphere、Capsule、Mesh（性能开销大）
- 可设置 Physics Material（物理材质）控制摩擦和弹性

### 3. Trigger（触发器）
- `Is Trigger = true`：物体可穿过，仅检测重叠
- `Is Trigger = false`：物理碰撞响应

### 碰撞函数对照表

| 类型 | 进入 | 保持 | 离开 | 参数 |
|------|------|------|------|------|
| 碰撞器 | OnCollisionEnter | OnCollisionStay | OnCollisionExit | Collision |
| 触发器 | OnTriggerEnter | OnTriggerStay | OnTriggerExit | Collider |

### 触发条件
- **碰撞器**：双方 IsTrigger=false + 至少一个非 Kinematic 刚体
- **触发器**：至少一个 IsTrigger=true + 一个刚体（可 Kinematic）

### 4. Joints（关节）
- Hinge Joint：门、钟摆
- Spring Joint：弹性连接
- Fixed Joint：固定相对位置

### 5. Character Controller
- 为角色移动封装的简化物理组件
- 不能穿过静态碰撞物体
- 与 Rigidbody 的区别：不需要物理模拟，适合角色移动

---

## 二、物理更新

**必须使用 `FixedUpdate()`**，因为物理计算需固定时间步长（默认 0.02s），与帧率无关。

---

# 第五部分：动画系统

## 一、Mecanim 系统

### 工作流
```
动画片段（Animation Clip）→ 动画控制器（Animator Controller）→ Animator 组件 → Avatar（骨骼映射）
```

### 动画状态机核心元素
- **Entry**：状态机入口，指向默认状态
- **AnyState**：全局跳转节点（如角色死亡强制中断）
- **过渡条件**：基于 Float/Bool/Trigger 参数触发
- **Exit**：状态机结束

### 参数类型
Float（速度）、Int（状态 ID）、Bool（是否奔跑）、Trigger（攻击触发）

### 动画事件机制
- 在 Animation Clip 中插入事件帧
- 动画播放到指定帧时自动调用绑定方法
- 可传递参数（string、float、object）

---

## 二、Blend Tree（混合树）

### 1D Blend Tree
基于单一参数（如 Speed）在多个动画间平滑过渡：
```
Speed=0  → 站立 100%
Speed=5  → 走路 100%
Speed=10 → 跑步 100%
```

### 2D Blend Tree
基于两个参数（如 Horizontal + Vertical），平滑融合 8 方向移动动画。

---

## 三、动画分层（Layers）

- **Layer 0**：下半身移动
- **Layer 1**：上半身开火
- **Avatar Mask**：指定某层仅作用于部分骨骼
- **Blending Mode**：
  - Override：高层覆盖底层
  - Additive：高层叠加到底层（如在跑步基础上叠加受击晃动）

---

## 四、PlayableGraph

Unity 底层动画控制 API，提供：
- 灵活的程序化动画混合
- 运行时动态创建动画状态
- 更低的性能开销

```csharp
var graph = PlayableGraph.Create("MyGraph");
var output = AnimationPlayableOutput.Create(graph, "Output", animator);
var clipPlayable = AnimationClipPlayable.Create(graph, clip);
output.SetSourcePlayable(clipPlayable);
graph.Play();
```

---

## 五、Motion Matching

### 核心思想
每帧从动画数据库中搜索与当前角色状态（速度、方向、姿势）最匹配的动画帧，无需手动构建状态机。

### 优势
- 大幅减少手工状态机工作量
- 动画过渡更自然流畅

### 挑战
- 需要大量高质量动画数据
- CPU 性能开销
- 动画数据质量要求严格

### 匹配公式
```
cost = bonesCost + trajectoryCost + rootMotionCost
```

---

## 六、动画知识点速查

| 概念 | 要点 |
|------|------|
| **Avatar** | 骨架映射系统，实现动画重定向 |
| **Humanoid** | 人形骨骼，支持重定向/IK/遮罩 |
| **Generic** | 通用骨骼，不支持重定向 |
| **IK（反向动力学）** | 控制子骨骼自动计算父骨骼旋转 |
| **Animation vs Animator** | 单动画播放 vs 状态机管理 |
| **性能优化** | 压缩动画、减少骨骼、动画 LOD |

---

# 第六部分：图形渲染

## 一、摄像机

### 关键属性
- `Clear Flags`：Skybox / Solid Color / Depth Only / Don't Clear
- `Culling Mask`：控制渲染哪些 Layer
- `Clipping Planes`：Near/Far 裁剪面
- `Viewport Rect`：分屏/多摄像机

### 常见问题
- **分屏**：调整 `camera.rect` 划分屏幕区域
- **跟随**：在 `LateUpdate` 中用 `Vector3.Lerp` 平滑移动
- **震动**：协程随机偏移位置，恢复原位
- **粉色画面**：未找到渲染管线材质
- **黑色画面**：Clear Flags 设为 Solid Color 且背景黑色

---

## 二、光照

### 四种光源

| 类型 | 特点 | 适用 |
|------|------|------|
| **Directional Light** | 全局平行照射，无距离限制 | 太阳光 |
| **Point Light** | 向四周发射，随距离衰减 | 灯泡、火焰 |
| **Spot Light** | 锥形区域照射 | 手电筒、车灯 |
| **Area Light** | 平面光源、柔和阴影 | 仅烘焙（光照贴图） |

### 光照烘焙
- 将静态物体光照预计算为 Lightmap
- 标记 Static → Light 设为 Baked → Window > Rendering > Lighting 烘焙

---

## 三、模型与网格

### 模型组成
顶点（Vertex）→ 边（Edge）→ 面（通常为三角面 Face）

### 关键概念
- **法线**：用于光照计算
- **切线**：配合法线贴图实现凹凸效果
- **UV 坐标**：控制纹理映射

### Mesh Filter vs Mesh Renderer
- **Mesh Filter**：存储网格数据
- **Mesh Renderer**：结合材质渲染网格

### 骨骼动画 vs 顶点动画
- 骨骼动画：内存低，适合角色
- 顶点动画：精度高，内存大

### LOD（多层次细节）
- 根据距离切换模型精度
- 优点：减少 GPU 渲染压力
- 缺点：增加内存（多级模型）

---

## 四、材质与着色器

### 材质 = Shader + 纹理 + 属性参数
- **Shader**：定义渲染规则（光照模型、混合模式）
- **纹理**：提供表面细节
- **属性**：颜色、金属度、光滑度等

### Shader 类型
| 类型 | 说明 |
|------|------|
| **Surface Shader** | 封装光照模型，适合 PBR |
| **Unlit Shader** | 忽略光照，UI/特效 |
| **Vertex/Fragment Shader** | 完全自定义 |
| **Shader Graph** | 可视化编辑 |

### PBR 关键属性
- Albedo（基础颜色）
- Metallic（金属度，0-1）
- Smoothness（光滑度）
- Normal Map（法线贴图）

### 动态换色最佳实践
使用 `MaterialPropertyBlock`，避免创建新材质实例：
```csharp
MaterialPropertyBlock props = new MaterialPropertyBlock();
props.SetColor("_Color", Color.blue);
renderer.SetPropertyBlock(props);
```

---

## 五、渲染管线

### 四种类型

| 管线 | 平台 | 画质 | 适用 |
|------|------|------|------|
| **内置管线** | 全平台 | 基础 | 2D/旧项目 |
| **URP** | 移动端/PC/VR | 中等 | 中小型 3D |
| **HDRP** | 高端 PC/主机 | 电影级 | 3A 游戏 |
| **自定义 SRP** | 需手动适配 | 完全自定义 | 特殊需求 |

### SRP（可编程渲染管线）
- `RenderPipelineAsset`：配置入口
- `RenderPipeline`：渲染逻辑（C# 控制整个渲染流程）
- `ScriptableRenderContext`：底层渲染上下文

### 内置管线迁移到 URP
1. 安装 Universal RP 包
2. 创建 URP 配置资产
3. Project Settings > Graphics 切换管线
4. Edit > Render Pipeline > Upgrade Project Materials 批量升级材质

---

## 六、渲染面试速答

- **渲染管线阶段**：应用程序 → 几何处理 → 光栅化 → 像素处理 → FrameBuffer
- **Forward vs Deferred**：Forward O(物体×光源)，Deferred O(物体+光源)
- **DrawCall**：CPU 向 GPU 发起的一次绘制指令
- **Overdraw**：同一像素被多次绘制
- **SRP Batcher**：批量设置材质属性（非合批网格）

---

# 第七部分：网络通信

## 一、Protobuf

### 优势（vs JSON/XML）
- 体积小 50%-90%
- 速度快 20-100 倍
- 强类型 Schema
- 跨语言支持
- 向前/向后兼容

### Unity 使用流程
1. 定义 `.proto` 文件
2. `protoc` 生成 C# 代码
3. 序列化：`msg.ToByteArray()`
4. 反序列化：`Player.Parser.ParseFrom(data)`

### GC 优化
- 对象池复用消息实例
- `ArrayPool<byte>` 复用字节缓冲区
- 避免频繁 new

---

## 二、状态同步

### 核心思想
服务器是权威，只同步对象的关键属性（位置、血量、动作）。
**适用场景**：MMORPG、沙盒游戏。

### 核心组件
- `NetworkIdentity`：标记需同步的对象
- `NetworkTransform`：自动同步位置/旋转
- `[SyncVar]`：同步变量

### 优化
- 只同步变化的数据
- 压缩浮点数精度
- 状态插值平滑显示
- AOI（只同步玩家周围物体）

---

## 三、帧同步

### 核心思想
所有客户端基于相同输入和初始状态独立计算相同逻辑。
**适用场景**：格斗游戏、RTS（如星际争霸）。

### 关键点
- **锁步**：等待所有玩家操作后再推进帧
- **确定性**：定点数计算，避免浮点精度差异
- **逻辑帧率**：15-20 FPS（与渲染帧率 60 FPS 分离）
- **延迟处理**：客户端预测 + 服务器校正

### 帧同步 vs 状态同步

| | 状态同步 | 帧同步 |
|------|------|------|
| **同步内容** | 对象属性 | 玩家输入指令 |
| **计算位置** | 服务器计算 | 所有客户端独立计算 |
| **带宽** | 较高 | 较低 |
| **适用** | 大型开放世界 | 竞技对战 |
| **防作弊** | 强（服务器权威） | 弱 |

---

# 第八部分：性能优化（深度版）

> 参考：Unity 官方文档、知乎/CSDN 高赞技术文章、UGUI 源码分析、PhysX 文档

---

## 一、优化金字塔

性能优化不是无头苍蝇式的乱撞，而是一个从架构到细节的系统工程：

```
                    ┌──────────────┐
                    │  架构设计     │ ← 对象池、ECS、模块化、分层
                    ├──────────────┤
                    │  资源管理     │ ← AssetBundle、Addressables、引用计数
                    ├──────────────┤
                    │  内存优化     │ ← 纹理压缩、资源卸载、堆内存控制
                    ├──────────────┤
                    │  CPU 优化     │ ← DrawCall、GC、算法、脚本
                    ├──────────────┤
                    │  GPU 优化     │ ← Shader、Overdraw、填充率
                    └──────────────┘
```

**核心原则**：从上往下优化。架构问题不能靠底层技巧弥补。

---

## 二、CPU 优化

### 2.1 DrawCall 深度解析

#### 什么是 DrawCall？
DrawCall 是 CPU 向 GPU 发起的一次绘制指令。每次 DrawCall 意味着：CPU 准备数据（设置渲染状态、绑定纹理、传入顶点）→ 发送指令给 GPU → GPU 执行绘制。DrawCall 的代价主要在 **CPU 端**（状态切换、命令提交），而非 GPU 端。

#### DrawCall 是怎么产生的？
每个使用不同材质（Material）的 MeshRenderer / Graphic 都会产生至少一个 DrawCall。如果同一个材质有多个物体，理论上可以合批为一个 DrawCall。

#### 降低 DrawCall 的六大手段

**① Static Batching（静态合批）**
- **原理**：Build 或运行时将标记为 Static 的物体合并为一个大网格，存于内存中
- **条件**：物体标记为 Batching Static、使用相同材质
- **优点**：几乎没有运行时开销
- **缺点**：额外内存开销（合并后网格占用额外内存）、物体不能移动
- **注意**：大量相同模型（如森林里的树）不适合静态合批 → 用 GPU Instancing

**② Dynamic Batching（动态合批）**
- **原理**：Unity 运行时自动将符合条件的网格动态合并
- **条件**：顶点数 < 300（不同 Unity 版本有差异）、相同材质、单 Pass Shader
- **优点**：全自动，物体可以移动
- **缺点**：条件苛刻，很多场景不触发
- **顶点数计算**：`顶点属性数 × 顶点数 ≤ 900`（如 Shader 用了 position+normal+uv，每个顶点 3 个属性，则最多 300 顶点）

**③ GPU Instancing**
- **原理**：用一个 DrawCall 批量渲染多个相同 Mesh + 相同材质的实例
- **适用**：大量相同物体（植被、子弹、敌人）
- **条件**：Shader 需支持 Instancing（`#pragma multi_compile_instancing`）
- **优势**：不增加内存，每个实例可独立设置颜色/位置等属性（通过 MaterialPropertyBlock）

**④ SRP Batcher（URP/HDRP）**
- **不是合批网格**，而是**批量设置材质属性**
- **原理**：将材质属性缓存在 GPU 的 CBUFFER 中，减少每个 DrawCall 的 CPU 设置开销
- **条件**：Shader 需兼容 SRP Batcher（属性声明在 `CBUFFER_START(UnityPerMaterial)` 中）
- **效果**：不减少 DrawCall 数量，但大幅降低每个 DrawCall 的 CPU 耗时

**⑤ 网格合并（Mesh Combine）**
- **原理**：运行时通过 `Mesh.CombineMeshes` 将多个网格合并为一个
- **适用**：需要静态合批效果但物体不标记 Static 的场景
- **注意**：合并后贴图需打在同一图集中

**⑥ 图集合批（Sprite Atlas）**
- **原理**：将多个小图合并为一张大图，使用同一材质
- **适用**：2D 游戏、UI 系统
- **条件**：同图集 + 同材质 + 深度连续

#### DrawCall 目标值
| 平台 | 目标 DrawCall |
|------|------|
| 移动端（低端） | < 50 |
| 移动端（中高端） | < 150 |
| PC | < 1000 |
| 主机 | < 2000 |

---

### 2.2 GC 优化（零 GC 编码）

#### GC 为什么会卡顿？
Unity 早期使用 Boehm GC（非分代），执行时 **Stop-The-World**：暂停所有游戏逻辑，扫描堆内存、标记可达对象、回收不可达对象。一次 GC 可能耗时几十到几百毫秒，直接导致掉帧。

Unity 2019.3+ 引入 **Incremental GC（增量 GC）**，将标记工作分摊到多帧，减少单帧卡顿时间，但总开销略微增加。

#### 高频 GC Alloc 场景与对策

**① 装箱（Boxing）**
```csharp
// ❌ 装箱：值类型→object，在堆上分配新对象
Debug.Log("Score: " + score);         // int 装箱
ArrayList list = new ArrayList();
list.Add(100);                        // int 装箱

// ✅ 避免装箱
Debug.Log($"Score: {score}");          // 字符串插值不装箱（C# 6+）
List<int> list = new List<int>();      // 泛型集合
list.Add(100);                         // 不装箱
```

**② 字符串操作**
```csharp
// ❌ 每次都创建新 string 对象
string info = "Name: " + name + ", Level: " + level;

// ✅ StringBuilder 复用
private StringBuilder _sb = new StringBuilder(256);
void Update() {
    _sb.Clear();
    _sb.Append("Name: ").Append(name).Append(", Level: ").Append(level);
    label.text = _sb.ToString();
}

// ❌ tag 比较产生 GC
if (other.gameObject.tag == "Player") { }

// ✅ 无 GC 比较
if (other.gameObject.CompareTag("Player")) { }
```

**③ LINQ**
```csharp
// ❌ LINQ 在 Update 中 → 每次创建迭代器、委托、闭包
var alive = enemies.Where(e => e.health > 0).ToList();

// ✅ 手写循环
List<Enemy> alive = new List<Enemy>();
for (int i = 0; i < enemies.Count; i++) {
    if (enemies[i].health > 0) alive.Add(enemies[i]);
}
```

**④ Unity API 分配**
```csharp
// ❌ 返回数组的 API 每次都分配新数组
RaycastHit[] hits = Physics.RaycastAll(ray);
Mesh mesh = GetComponent<MeshFilter>().mesh;  // 复制一份！

// ✅ 使用 NonAlloc 版本 + 预分配数组
RaycastHit[] hits = new RaycastHit[10];
int count = Physics.RaycastNonAlloc(ray, hits);
Mesh mesh = GetComponent<MeshFilter>().sharedMesh;  // 共享引用
```

**⑤ 闭包（Closure）**
```csharp
// ❌ 捕获了外部变量 → 每次 new DisplayClass（堆分配）
int score = 100;
button.onClick.AddListener(() => ShowScore(score));

// ✅ 避免捕获：用在 Update 外预先创建委托
```

#### GC 优化核心口诀
> **热点代码（Update/FixedUpdate）零分配；常用对象用池化；字符串拼接 StringBuilder；物理检测 NonAlloc。**

---

### 2.3 对象池（Object Pooling）

#### 为什么需要对象池？
`Instantiate` + `Destroy` 的代价：
- Instantiate：堆分配 + 反序列化 + 组件初始化 + Awake/OnEnable
- Destroy：内存标记 + Finalizer + 触发 GC

对于频繁创建/销毁的对象（子弹、特效、敌人），对象池是最重要的 GC 优化手段。

#### 实现
```csharp
public class ObjectPool<T> where T : Component {
    private Queue<T> _pool = new();
    private T _prefab;
    private Transform _parent;
    
    public ObjectPool(T prefab, int initialSize, Transform parent = null) {
        _prefab = prefab;
        _parent = parent;
        for (int i = 0; i < initialSize; i++) {
            var obj = GameObject.Instantiate(_prefab, _parent);
            obj.gameObject.SetActive(false);
            _pool.Enqueue(obj);
        }
    }
    
    public T Get() {
        T obj = _pool.Count > 0 ? _pool.Dequeue() : GameObject.Instantiate(_prefab, _parent);
        obj.gameObject.SetActive(true);
        return obj;
    }
    
    public void Return(T obj) {
        obj.gameObject.SetActive(false);
        _pool.Enqueue(obj);
    }
}

// Unity 2021+ 官方池 API
using UnityEngine.Pool;
ObjectPool<GameObject> pool = new ObjectPool<GameObject>(
    createFunc: () => Instantiate(prefab),
    actionOnGet: obj => obj.SetActive(true),
    actionOnRelease: obj => obj.SetActive(false),
    actionOnDestroy: obj => Destroy(obj)
);
```

---

### 2.4 脚本优化

#### MonoBehaviour 调用的 C++ → C# 跨域开销
每个 `MonoBehaviour.Update()` 都是引擎从 C++ 调用 C# 的跨域调用。如果有 10000 个 MonoBehaviour，每帧就有 10000 次跨域调用的开销。

```csharp
// ❌ 每个敌人都独立 Update
class Enemy : MonoBehaviour { void Update() { Move(); } }

// ✅ 一个管理器统一 Tick
class EnemyManager : MonoBehaviour {
    List<EnemyData> enemies;
    void Update() {
        for (int i = 0; i < enemies.Count; i++)
            enemies[i].Tick(Time.deltaTime);  // 只 1 次 C++→C# 调用
    }
}
```

#### 缓存引用
```csharp
// ❌ 每帧搜索组件
void Update() { GetComponent<Rigidbody>().AddForce(Vector3.up); }

// ✅ 启动时缓存
private Rigidbody _rb;
void Awake() { _rb = GetComponent<Rigidbody>(); }
void FixedUpdate() { _rb.AddForce(Vector3.up); }
```

#### 数学运算优化
```csharp
// ❌ 开方运算
float dist = Vector3.Distance(a, b);    // 内部调用了 sqrt
if (Vector3.Magnitude(v) > 5f) { }      // sqrt

// ✅ 用平方比较代替
float sqrDist = (a - b).sqrMagnitude;   // 无 sqrt
if (v.sqrMagnitude > 25f) { }           // 5² = 25
```

#### 分帧处理
```csharp
// ❌ 一帧内创建 10000 个物体 → 卡顿
for (int i = 0; i < 10000; i++) Instantiate(prefab);

// ✅ 分帧处理，每帧只创建 10 个
IEnumerator SpawnOverTime() {
    for (int i = 0; i < 10000; i++) {
        Instantiate(prefab);
        if (i % 10 == 0) yield return null;  // 每 10 个让一帧
    }
}
```

#### UGUI 重建优化
- **动静分离**：动态 UI（血条）与静态 UI（背景）分属不同 Canvas
- **关闭 Raycast Target**：不需要点击的 UI 元素关闭 Raycast Target
- **避免 Layout Group 嵌套过深**：每帧重新计算布局开销大
- **Text 不要每帧改**：用事件/计时器控制修改频率

---

## 三、GPU 优化

### 3.1 渲染管线开销分析

GPU 每帧的工作量取决于三个因素：
1. **顶点数量**（Vertex Shader 执行次数）
2. **片元数量**（Fragment Shader 执行次数 = 屏幕像素 × Overdraw 倍数）
3. **Shader 复杂度**（每条指令的 GPU 周期）

### 3.2 Overdraw（过度绘制）

**定义**：同一像素被多次绘制。

**来源**：
- 透明物体叠加（粒子特效、UI 半透明面板）
- UI 多层嵌套
- 未做遮挡剔除的实体

**检测**：Scene 视图 → Shading Mode → Overdraw。越白越严重。

**优化**：
- 不透明物体从近到远渲染（利用 Early-Z 剔除）
- 减少透明 UI 叠加
- 粒子特效减少重叠区域
- 使用 `RectMask2D` 裁剪不可见 UI

### 3.3 Shader 优化

| 手段 | 效果 |
|------|------|
| `half` / `fixed` 代替 `float` | 移动端显著减少寄存器压力 |
| 减少纹理采样（tex2D）次数 | 每次采样都是显存带宽开销 |
| 避免 `if/else` 分支 | GPU 是 SIMD 架构，分支导致两边都执行 |
| 把计算从片元移到顶点 | 顶点数 << 像素数，大幅减少执行次数 |
| 避免 `sin/cos/pow/log` 等复杂函数 | 用查表法或近似计算替代 |
| `Alpha Test (clip)` 比 `Alpha Blend` 好 | ABlend 要读+写 framebuffer，ATest 只写 |
| 避免 `GrabPass` | 需要拷贝当前屏幕 → 显存带宽大 |

### 3.4 纹理优化

#### 压缩格式选型
| 平台 | 推荐格式 | 大小（1024×1024） | 说明 |
|------|------|------|------|
| Android（新） | ASTC 6×6 | 0.67 MB | 现代 GPU 全支持，推荐 |
| Android（兼容） | ETC2 | 1 MB | OpenGL ES 3.0+ |
| iOS | ASTC 4×4 | 1 MB | A8+ 芯片全支持 |
| PC | BC7 / DXT5 | 1 MB | 高质量 RGBA |

#### 关键设置
- **关闭 Read/Write Enabled**：否则内存中保留两份（GPU + CPU 可访问），内存翻倍
- **开启 Generate Mip Maps**：远处物体自动使用低分辨率（但内存增加约 33%）
- **Max Size**：UI 用 1024，3D 贴图按需设置，不要全部 2048
- **Sprite Atlas**：小图合并为 1024×1024 大图

### 3.5 网格优化

- 移动端角色 ≤ 5000 三角面
- 移动端场景物体 ≤ 2000 三角面
- 关闭不需要的顶点属性（Color、Tangent、UV2/UV3）
- 骨骼数量 ≤ 30 根（移动端角色）
- LOD Group：3-4 级 LOD，远处自动切换低模

---

## 四、内存优化

### 4.1 内存构成

Unity 游戏的内存主要由以下部分组成：

| 类别 | 占比 | 说明 |
|------|------|------|
| **纹理** | 30%-50% | 最大头，压缩是关键 |
| **网格** | 10%-20% | 模型顶点数据 |
| **动画** | 10%-15% | AnimationClip 数据 |
| **音频** | 5%-10% | AudioClip |
| **托管堆** | 10%-20% | C# 对象 |
| **其他** | 10% | Shader、RenderTexture 等 |

### 4.2 纹理内存计算

```
纹理内存 = 宽 × 高 × 格式字节数 × MipMap 系数(1.33)

1024×1024 RGBA32 未压缩 = 1024×1024×4×1.33 ≈ 5.3 MB
1024×1024 ASTC 6×6      = 1024×1024×4/36×1.33 ≈ 0.15 MB（约 35 倍压缩！）
```

### 4.3 托管堆内存

Mono 堆内存的特殊性：**只升不降**。堆一旦扩张就不会自动收缩返还给操作系统（即使 GC 后堆内空了，也不会释放给 OS）。

所以内存优化的关键不是 GC 频率，而是**控制堆内存峰值**。一次加载关卡时分配 500MB 的临时数组，即使释放了，堆也是 500MB 不会降回来。

### 4.4 资源生命周期管理

```
加载 → 使用 → 引用计数归零 → 卸载

正确做法：
1. 场景切换时释放上一场景所有资源
2. AssetBundle.Unload(false) 卸载 AB 包，保留已加载资源
3. Resources.UnloadUnusedAssets() 清理无引用资源
4. Addressables.Release() 释放引用
```

### 4.5 Resources 目录的陷阱

Resources 目录下所有资源无条件打包进安装包。即使 100GB 也必须全部下载。且 Resources 加载的资源永远不会被自动卸载，除非手动调用 `Resources.UnloadUnusedAssets()`。

**结论**：大项目完全不用 Resources。小项目或配置表可以用。

---

## 五、资源管理优化

### 5.1 AssetBundle vs Addressables

| | AssetBundle | Addressables |
|------|------|------|
| **加载方式** | 手动管理加载/卸载 | 异步加载，引用计数自动管理 |
| **内存管理** | 手动 Unload | 引用计数归零自动释放 |
| **热更新** | ✅ | ✅ |
| **学习成本** | 高 | 中 |
| **Unity 版本** | 全版本 | 2018.3+ |
| **推荐度** | 大型项目可用 | ✅ 现代最佳方案 |

### 5.2 Addressables 的三个核心概念

1. **Address（地址）**：每个资源的唯一标识符，替代文件路径
2. **Group（分组）**：资源分组，控制打包粒度
3. **Label（标签）**：跨组标记，批量操作（如加载所有"UI"标签的资源）

### 5.3 资源加载策略

- **预加载**：登录场景加载常驻资源（UI 图集、公用 Shader、角色骨骼）
- **按需加载**：打开某个界面时才加载对应资源
- **懒加载**：资源进入视野时才加载（AssetReference）
- **同步加载**：Loading 界面使用，不卡主线程
- **异步加载**：游戏进行中使用，避免卡顿

---

## 六、Profiler 工具链

### 6.1 Unity Profiler

**位置**：Window → Analysis → Profiler

| 模块 | 关注指标 | 目标 |
|------|------|------|
| **CPU Usage** | GC.Alloc 列 | **0 B**（热路径） |
| **GPU Usage** | ms 耗时 | < 16ms（60fps）/ < 33ms（30fps） |
| **Rendering** | Batches、SetPass Calls | 移动端 < 100 / PC < 1000 |
| **Memory** | Total Allocated、Texture、Mesh | 移动端 < 500MB |
| **Audio** | 音频内存、播放数 | 压缩格式 |

### 6.2 Frame Debugger

**位置**：Window → Analysis → Frame Debugger

逐 DrawCall 查看每帧绘制了什么：
- 合批是否生效（看 Batched 标记）
- 哪个 DrawCall 打破了合批（根据材质/纹理切换）
- 渲染顺序是否正确

### 6.3 Memory Profiler

**安装**：Package Manager → Memory Profiler

提供堆内存快照对比：加载关卡前 vs 加载关卡后，精准定位泄漏来源。

### 6.4 第三方工具
- **RenderDoc**：跨平台图形调试器，截帧分析 DrawCall / Shader
- **Snapdragon Profiler**：高通 GPU 专用，移动端性能分析利器
- **Xcode Instruments**：iOS 性能分析（Allocations、Time Profiler）
- **Android GPU Inspector**：Google 官方 Android GPU 分析

---

## 七、移动端特殊优化

### 7.1 移动端 GPU 架构特点

移动端 GPU 是 **TBR（Tile-Based Rendering）** 架构（Mali、Adreno、PowerVR 等），与 PC 的 IMR（Immediate Mode Rendering）不同：

- TBR 将画面分为小块（Tile，如 16×16），逐 tile 渲染
- 对连续内存访问（如顶点数据）非常友好
- 但对频繁的 Framebuffer 切换（如多 RenderTexture 切换）非常敏感
- `GrabPass` 在移动端要完全避免

### 7.2 帧率控制
- `Application.targetFrameRate = 30`（省电）或 `60`（流畅）
- 移动端 30fps 可接受（省电优先）

### 7.3 移动端 Shader 精度
```hlsl
// 移动端推荐精度
half4 color;    // 颜色（16位足够）
fixed alpha;    // 透明度
float3 worldPos; // 世界坐标（需 float）
float2 uv;       // UV 可以用 half
```

### 7.4 包体优化
- IL2CPP + Strip Engine Code
- Managed Stripping Level：Aggressive
- 纹理用 ASTC
- 音频 Vorbis 96kbps
- 移除未使用的 Shader Variant

---

## 八、代码优化速查表（面试必背）

### GC 类

| 问题 | 方案 |
|------|------|
| 字符串拼接产生 GC | StringBuilder 复用（Clear → Append） |
| `tag == "xxx"` 产生 GC | `CompareTag("xxx")` |
| 装箱 | 泛型集合、重写 ToString、避免 object 参数 |
| LINQ 产生 GC | 手写 for 循环 |
| foreach 装箱 | 用 for 或确保 List<T>/数组 |
| Lambda 闭包分配 | 避免在热路径捕获外部变量 |
| 物理检测分配数组 | `RaycastNonAlloc` / `OverlapSphereNonAlloc` |
| `Mesh.vertices` 复制 | 用 `sharedMesh` 或缓存 |

### CPU 类

| 问题 | 方案 |
|------|------|
| GetComponent 每帧调用 | Start/Awake 缓存 |
| GameObject.Find 慢 | 单例 + 引用 |
| SendMessage 反射 | 直接调用方法 |
| Camera.main 每帧 | 缓存引用 |
| Transform 频繁访问 | 缓存 transform（虽然已经是组件引用，多一层间接） |
| Update 太多 | 管理器统一 Update |
| 分帧处理大数据 | 协程 + yield return null |
| `Vector3.Distance` | `sqrMagnitude` 比较 |

### GPU 类

| 问题 | 方案 |
|------|------|
| DrawCall 过高 | 合批 + 图集 + GPU Instancing + SRP Batcher |
| Overdraw | 减少透明叠加 + RectMask2D + 不透明先渲染 |
| 纹理内存大 | ASTC 压缩 + MipMap + 关闭 Read/Write |
| Shader 复杂 | half/fixed 精度 + 移动端简化计算 |
| 网格面数过高 | LOD + 模型减面 |

---

## 九、推荐参考文章

以下是 Unity 性能优化领域公认的高质量参考资源：

### 官方文档
- [Unity - Performance Optimization](https://docs.unity3d.com/Manual/optimization.html)
- [Unity - AssetBundle 文档](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)
- [Unity - Profiler 文档](https://docs.unity3d.com/Manual/Profiler.html)

### 知乎高赞文章
- **《Unity 性能优化全攻略》** — 系统梳理 CPU/GPU/内存三位一体优化
- **《Unity UI 性能优化最佳实践》** — UGUI 合批/重建/动静分离深入分析
- **《Unity 移动端性能优化实战》** — 从 30fps 到 60fps 的完整复盘
- **《IL2CPP 原理与性能分析》** — IL2CPP vs Mono 底层对比

### CSDN 深度教程
- **《Unity GC 及优化超级全面解析》** — GC 原理到零 GC 编码的完整教程
- **《Unity Profiler 深度使用指南》** — Profiler 每个模块的指标解读
- **《Unity DrawCall 优化终极指南》** — 六大合批方式的适用场景与对比

### 社区资源
- [Unite 演讲视频](https://www.youtube.com/@Unity)：每年 Unite 大会的优化专题
- [Catlike Coding](https://catlikecoding.com/)：渲染/Shader 深度教程
- [The Gamedev Guru](https://thegamedev.guru/)：Unity DOTS/ECS 教程

---

> **性能优化核心思想**：
> 1. **用数据说话** — 先 Profiler 定位瓶颈，再优化
> 2. **帕累托法则** — 20% 的代码产生 80% 的性能问题
> 3. **从上到下** — 架构 > 算法 > 代码细节
> 4. **平台适配** — 移动端 ≠ PC，没有一劳永逸的方案
> 5. **Profile, Profile, Profile** — 优化前验证、优化后验证、回归时不引入新问题
