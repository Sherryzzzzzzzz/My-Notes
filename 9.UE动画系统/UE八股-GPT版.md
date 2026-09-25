# 🧠 模块一：UObject & 引擎底层（超细拆解）

------

# 🔹 1. UObject 基础

## 📌 概念

UObject 是 Unreal Engine 5 中所有可反射、可被 GC 管理的对象基类，是 UE 对 C++ 对象系统的扩展。

------

## ⚙️ 原理

- 所有 UObject 都继承自 UObject
- 不走标准 C++ new/delete
- 通过：
  - 反射系统（UHT 生成代码）
  - GC 系统（引用追踪）
- 每个 UObject：
  - 有 UClass 描述类型信息
  - 有 Outer 构成层级关系

本质：
 👉 **“带元数据 + 自动内存管理的对象系统”**

------

## 📂 源码位置

- UObject 定义：
  - `Runtime/CoreUObject/Public/UObject/Object.h`
- 核心实现：
  - `UObjectGlobals.cpp`
- 创建：
  - `StaticConstructObject_Internal`

------

## 🎯 面试回答（标准版）

> UObject 是 UE 自定义的对象系统，它通过 UHT 生成反射代码，使对象具备运行时类型信息，并结合 GC 实现自动内存管理。相比 C++ 原生对象，它支持序列化、编辑器暴露以及跨模块交互。

------

## 💡 项目结合

你可以这样讲你的战斗系统：

> 我把技能（Skill）设计成 UObject，而不是 Actor，因为它不需要存在于世界中，同时又需要支持配置化和生命周期管理，这样可以更轻量并且方便扩展。

------

# 🔹 2. UObject 创建机制

------

## 📌 概念

UObject 不能用 `new`，必须通过引擎提供的函数创建。

------

## ⚙️ 原理

创建流程：

1. 调用：
   - `NewObject<T>()`
   - `SpawnActor<T>()`（Actor 专用）
2. 内部流程：
   - 分配内存（GUObjectAllocator）
   - 注册到 UObject 数组
   - 设置 Outer / Name
   - 调用构造函数
   - 绑定 UClass

👉 关键：

- 构造函数只是初始化，不负责生命周期

------

## 📂 源码位置

- ```
  NewObject：
  ```

  - `UObjectGlobals.h`

- 核心：

  - `StaticConstructObject_Internal`

------

## 🎯 面试回答

> UObject 不能通过 new 创建，因为需要注册到引擎的对象系统中。创建过程由 NewObject 完成，它不仅分配内存，还会注册对象、初始化反射信息，并纳入 GC 管理。

------

## 💡 项目结合

> 在战斗系统中，我用 NewObject 创建技能实例，这样可以让技能自动被 GC 管理，而不需要手动释放。

------

# 🔹 3. UCLASS / UPROPERTY / UFUNCTION

------

## 📌 概念

UE 的反射宏，用于让类、变量、函数进入反射系统。

------

## ⚙️ 原理

不是宏本身实现功能，而是：

👉 **UHT（Header Tool）解析这些宏，生成代码**

例如：

- 生成 `.generated.h`
- 注册 UClass
- 注册属性和函数

运行时：

- 通过 UClass 查找属性
- 支持编辑器修改 / 蓝图调用

------

## 📂 源码位置

- 宏定义：
  - `ObjectMacros.h`
- 生成代码：
  - `*.generated.h`

------

## 🎯 面试回答

> UCLASS 等宏本身不提供功能，而是标记给 UHT 使用。UHT 会在编译前生成反射代码，使类具备运行时类型信息，从而支持编辑器、序列化和蓝图调用。

------

## 💡 项目结合

> 我在技能系统中使用 UPROPERTY 暴露参数，这样可以直接在编辑器调数值，提高调试效率。

------

# 🔹 4. UClass（类型系统）

------

## 📌 概念

UClass 是 UObject 的类型描述对象，相当于 UE 的 RTTI。

------

## ⚙️ 原理

- 每个 UObject 都有：
  - `UClass* Class`
- UClass 存储：
  - 属性列表
  - 函数列表
  - 父类关系

支持：

- `IsA()`
- `Cast<>`

👉 替代 C++ RTTI

------

## 📂 源码位置

- `UObject/Class.h`

------

## 🎯 面试回答

> UClass 是 UE 自定义的类型系统，它在运行时描述类信息，支持类型判断和反射，相比 C++ RTTI 更可控且可扩展。

------

## 💡 项目结合

> 我在状态机中用 IsA 判断状态类型，避免使用 dynamic_cast，提高性能并符合 UE 体系。

------

# 🔹 5. GC（垃圾回收）

------

## 📌 概念

UE 使用自动垃圾回收管理 UObject 生命周期。

------

## ⚙️ 原理

### 核心算法：

- Mark & Sweep

### 步骤：

1. 从 Root Set 开始
2. 遍历引用（UPROPERTY）
3. 标记存活对象
4. 清除未标记对象

### Root：

- 全局对象
- 被引用对象

👉 关键：

- **只有 UPROPERTY 才会被追踪**

------

## 📂 源码位置

- `GarbageCollection.cpp`

------

## 🎯 面试回答

> UE 使用基于引用追踪的 GC，通过 UPROPERTY 标记引用关系，从 Root Set 开始遍历，未被引用的对象会被回收。

------

## 💡 项目结合

> 我避免在技能系统中使用裸指针引用 UObject，否则可能被 GC 回收导致崩溃。

------

# 🔹 6. 智能指针（UE版）

------

## 📌 概念

UE 提供多种指针类型管理对象引用。

------

## ⚙️ 原理

### 类型：

- ```
  TWeakObjectPtr
  ```

  - 不阻止 GC

- ```
  TSoftObjectPtr
  ```

  - 延迟加载资源

- ```
  TSharedPtr
  ```

  - 非 UObject

------

## 📂 源码位置

- `Templates/SharedPointer.h`

------

## 🎯 面试回答

> UObject 通常用 UPROPERTY 管理，而非 UObject 使用 TSharedPtr。弱引用使用 TWeakObjectPtr 避免影响 GC。

------

## 💡 项目结合

> 子弹系统中，我用对象池 + 非 UObject 管理，避免 GC 带来的性能问题。

------

# 🔹 7. Outer 机制

------

## 📌 概念

UObject 通过 Outer 构成层级关系。

------

## ⚙️ 原理

- 每个 UObject 有 Outer
- 构成树结构：
  - Package → Object → SubObject

作用：

- 生命周期管理
- GC 依赖关系

------

## 📂 源码位置

- `UObjectBase.cpp`

------

## 🎯 面试回答

> Outer 表示对象的归属关系，UE 通过 Outer 构建对象层级，并在 GC 时利用该关系进行引用追踪。

------

## 💡 项目结合

> 我将技能实例的 Outer 设为角色，这样角色销毁时技能也会被正确回收。

------

# 🔹 8. 生命周期（重点）

------

## 📌 概念

UObject 从创建到销毁的完整流程。

------

## ⚙️ 原理

顺序：

1. Constructor
2. PostInitProperties
3. BeginDestroy
4. FinishDestroy

------

## 📂 源码位置

- `UObjectGlobals.cpp`

------

## 🎯 面试回答

> UObject 生命周期由引擎控制，开发者不能直接 delete，对象销毁通过 GC 调用 BeginDestroy 和 FinishDestroy。

------

## 💡 项目结合

> 我在 BeginDestroy 中释放技能缓存资源，避免内存泄漏。

# 🧠 模块二：Actor / Component / Gameplay Framework（高频必杀区）

这一块是腾讯客户端岗**最爱问 + 最容易追问到死**的部分。

------

# 🔹 1. Actor 基础

## 📌 概念

Actor 是 Unreal Engine 5 中**存在于世界中的实体对象**，用于表示角色、物体、子弹等。

------

## ⚙️ 原理

- 继承自 UObject，但：
  - 绑定到 World
  - 有 Transform（位置/旋转/缩放）
- 通过 Level 管理
- 支持：
  - Tick
  - 网络同步
  - 组件挂载

👉 本质：
 **“带空间属性 + 生命周期 + 网络能力的 UObject”**

------

## 📂 源码位置

- 定义：
  - `Runtime/Engine/Classes/GameFramework/Actor.h`
- 实现：
  - `Actor.cpp`

------

## 🎯 面试回答

> Actor 是 UE 中的世界实体，继承自 UObject，并扩展了空间信息、生命周期管理以及网络同步能力。它通过 Level 进行管理，是游戏中大多数实体的基础。

------

## 💡 项目结合

> 在 TPS 项目中，我将子弹设计为 Actor，因为它需要位置、碰撞以及生命周期管理，同时还可能涉及网络同步。

------

# 🔹 2. Actor 生命周期（高频细节题）

------

## 📌 概念

Actor 从创建到销毁的完整流程。

------

## ⚙️ 原理（顺序非常重要）

### 创建阶段

1. Constructor
2. OnConstruction（蓝图）
3. PreInitializeComponents
4. PostInitializeComponents
5. BeginPlay

### 运行阶段

- Tick

### 销毁阶段

1. Destroy()
2. EndPlay
3. BeginDestroy（UObject）
4. GC 回收

------

## 📂 源码位置

- `Actor.cpp`

------

## 🎯 面试回答

> Actor 生命周期分为创建、运行和销毁三个阶段，其中 BeginPlay 是游戏逻辑开始的关键点，而销毁通过 Destroy 标记后由 GC 完成。

------

## 💡 项目结合

> 我在 BeginPlay 中初始化战斗状态机，在 EndPlay 中清理输入绑定，避免野指针问题。

------

# 🔹 3. Component 组件系统

------

## 📌 概念

Component 是用于**组合 Actor 功能的模块化单元**。

------

## ⚙️ 原理

UE 采用：
 👉 **组合优于继承（Composition over Inheritance）**

组件类型：

### ActorComponent

- 无 Transform
- 逻辑组件

### SceneComponent

- 有 Transform
- 可挂载

### PrimitiveComponent

- 可渲染 + 碰撞

------

## 📂 源码位置

- `ActorComponent.h`
- `SceneComponent.h`

------

## 🎯 面试回答

> UE 使用组件系统实现功能解耦，Actor 负责整体逻辑，而组件负责具体功能，例如移动、渲染和碰撞，从而避免继承层级过深。

------

## 💡 项目结合

> 我的战斗系统中，将攻击检测拆成独立 Component，这样可以复用在不同角色上。

------

# 🔹 4. Actor vs Component（必问）

------

## 📌 概念

Actor 是实体，Component 是功能模块。

------

## ⚙️ 原理

区别：

| 属性             | Actor | Component           |
| ---------------- | ----- | ------------------- |
| 是否存在世界     | ✅     | ❌                   |
| 是否有 Transform | ✅     | SceneComponent 才有 |
| 是否可独立存在   | ✅     | ❌                   |

------

## 📂 源码位置

- Actor / Component 类定义

------

## 🎯 面试回答

> Actor 表示世界中的实体，而 Component 是附加在 Actor 上的功能模块。UE 通过组件化设计提升系统的可扩展性和复用性。

------

## 💡 项目结合

> 我避免把所有逻辑写在 Character 中，而是拆成多个 Component，比如输入、战斗、状态管理。

------

# 🔹 5. Tick 机制（高频性能题）

------

## 📌 概念

Tick 是 Actor/Component 每帧执行的更新函数。

------

## ⚙️ 原理

- 每帧调用
- 可配置：
  - 是否启用
  - TickGroup
  - TickInterval

👉 性能关键点：

- Tick 数量过多 → CPU 压力大

------

## 📂 源码位置

- `TickTaskManager.cpp`

------

## 🎯 面试回答

> Tick 是 UE 的逐帧更新机制，但过多 Tick 会带来性能问题，因此通常需要通过事件驱动或降低 Tick 频率进行优化。

------

## 💡 项目结合

> 子弹系统中我避免使用 Tick，而是用生命周期+碰撞事件，提高性能。

------

# 🔹 6. Pawn / Character

------

## 📌 概念

Pawn 是可被控制的 Actor，Character 是带移动系统的 Pawn。

------

## ⚙️ 原理

### Pawn

- 可被 Controller 控制

### Character

- 内置：
  - Capsule
  - SkeletalMesh
  - CharacterMovementComponent

------

## 📂 源码位置

- `Pawn.h`
- `Character.h`

------

## 🎯 面试回答

> Pawn 是可被控制的基础类，而 Character 在其基础上集成了移动和碰撞系统，适用于角色开发。

------

## 💡 项目结合

> 我的角色使用 Character，因为需要内置移动系统和动画支持。

------

# 🔹 7. Controller（控制体系）

------

## 📌 概念

Controller 负责控制 Pawn。

------

## ⚙️ 原理

### PlayerController

- 接收玩家输入

### AIController

- 控制 AI 行为

👉 控制流程：
 Input → Controller → Pawn

------

## 📂 源码位置

- `PlayerController.h`
- `AIController.h`

------

## 🎯 面试回答

> Controller 将输入与角色解耦，使得同一个 Pawn 可以被玩家或 AI 控制。

------

## 💡 项目结合

> 我在 PlayerController 中处理输入，再传递给战斗系统，保证逻辑清晰。

------

# 🔹 8. GameMode / GameState / PlayerState（高频必问）

------

## 📌 概念

UE 的游戏规则与同步体系。

------

## ⚙️ 原理

### GameMode

- 只存在服务器
- 控制规则

### GameState

- 同步全局数据

### PlayerState

- 同步玩家数据

------

## 📂 源码位置

- `GameModeBase.h`
- `GameStateBase.h`

------

## 🎯 面试回答

> GameMode 负责规则但不参与同步，GameState 和 PlayerState 用于同步游戏和玩家数据。

------

## 💡 项目结合

> 我将玩家战斗数据放在 PlayerState 中，保证多人同步一致。

------

# 🔹 9. 输入系统（Enhanced Input）

------

## 📌 概念

UE5 新输入系统。

------

## ⚙️ 原理

- Input Action
- Input Mapping Context
- Trigger / Modifier

流程：
 输入 → Mapping → Action → 回调

------

## 📂 源码位置

- `EnhancedInputSubsystem`

------

## 🎯 面试回答

> Enhanced Input 提供数据驱动的输入映射，支持复杂触发条件，比旧系统更灵活。

------

## 💡 项目结合

> 我用 Input Action 控制攻击输入，并结合状态机判断是否可执行连招。

------

# 🔹 10. Subsystem（架构题）

------

## 📌 概念

全局系统管理类。

------

## ⚙️ 原理

生命周期绑定：

- Engine
- GameInstance
- World

👉 替代 Singleton

------

## 📂 源码位置

- `Subsystem.h`

------

## 🎯 面试回答

> Subsystem 是 UE 推荐的全局系统实现方式，比传统单例更安全，生命周期更清晰。

------

## 💡 项目结合

> 我可以把战斗管理器放在 GameInstanceSubsystem 中，统一管理。

# 🧠 模块三：动画系统（Animation System 深度拆解）

------

# 🔹 1. Animation Blueprint（动画蓝图核心）

------

## 📌 概念

Animation Blueprint 是 Unreal Engine 5 中用于**驱动角色骨骼动画的逻辑系统**，专门作用于 SkeletalMesh。

------

## ⚙️ 原理

Animation Blueprint 本质分两部分：

### 1️⃣ Event Graph（逻辑层）

- 每帧更新变量（Speed、IsInAir 等）
- 从 Character 获取状态

### 2️⃣ Anim Graph（动画计算层）

- 根据变量 → 输出最终 Pose

👉 核心流程：

```
Character → 更新变量 → AnimGraph计算 → 输出骨骼Pose
```

👉 本质：
 **“数据驱动的动画状态计算器”**

------

## 📂 源码位置

- `AnimInstance.h`
- `AnimInstance.cpp`

------

## 🎯 面试回答

> Animation Blueprint 通过 Event Graph 更新动画参数，在 Anim Graph 中根据这些参数进行 Pose 计算，最终驱动骨骼动画，实现逻辑与表现的分离。

------

## 💡 项目结合

> 我在战斗系统中，将角色状态（攻击、移动、受击）同步到 AnimBP，由 AnimGraph 决定播放哪个动画，从而实现逻辑与动画解耦。

------

# 🔹 2. AnimGraph（动画计算核心）

------

## 📌 概念

AnimGraph 是动画蓝图中用于**计算最终骨骼姿态（Pose）**的节点图。

------

## ⚙️ 原理

- 每个节点输出一个 Pose
- 节点之间通过 Blend 组合
- 最终输出：
  - Final Animation Pose

👉 常见节点：

- Sequence Player
- Blend
- State Machine

👉 本质：
 **“Pose 计算 DAG（有向图）”**

------

## 📂 源码位置

- `AnimNode_Base.h`
- `AnimNode_BlendList.cpp`

------

## 🎯 面试回答

> AnimGraph 是一个基于节点的 Pose 计算图，每个节点生成或混合动画姿态，最终输出角色骨骼的最终变换。

------

## 💡 项目结合

> 我使用 Blend 节点实现移动与攻击动画的平滑过渡，避免动画切换突兀。

------

# 🔹 3. State Machine（动画状态机）

------

## 📌 概念

用于管理不同动画状态（Idle / Run / Attack）的切换逻辑。

------

## ⚙️ 原理

组成：

- State（状态）
- Transition（过渡条件）

执行流程：

```
当前State → 判断Transition → 切换State → 播放动画
```

👉 Transition 本质：

- 布尔表达式

👉 本质：
 **“条件驱动的动画切换系统”**

------

## 📂 源码位置

- `AnimNode_StateMachine.cpp`

------

## 🎯 面试回答

> 动画状态机通过状态和过渡条件控制动画切换，适用于移动等稳定状态，但对于复杂技能系统扩展性较差。

------

## 💡 项目结合（关键加分）

> 我没有完全依赖动画状态机，而是使用代码状态机控制战斗逻辑，再驱动动画播放，这样更适合复杂连招系统。

💥（这句话是面试加分点）

------

# 🔹 4. Blend Space（混合空间）

------

## 📌 概念

用于根据参数（如速度、方向）在多个动画之间插值。

------

## ⚙️ 原理

- 输入参数（Speed / Direction）
- 在二维空间中插值动画

👉 本质：
 **“参数驱动的动画插值”**

------

## 📂 源码位置

- `BlendSpace.cpp`

------

## 🎯 面试回答

> Blend Space 通过参数插值多个动画，实现如从走到跑的平滑过渡，常用于移动系统。

------

## 💡 项目结合

> 我用 BlendSpace 实现角色 8方向移动动画，根据输入方向动态混合动画。

------

# 🔹 5. Animation Montage（高频必问）

------

## 📌 概念

用于播放**一次性或可控制的动画片段（如技能、攻击）**。

------

## ⚙️ 原理

结构：

- Slot（插槽）
- Section（分段）
- Notify（事件）

特点：

- 可中断
- 可跳转 Section
- 支持事件回调

👉 本质：
 **“可控制时间轴动画”**

------

## 📂 源码位置

- `AnimMontage.cpp`

------

## 🎯 面试回答

> Montage 用于播放技能等非循环动画，支持分段控制和事件通知，比状态机更适合复杂行为。

------

## 💡 项目结合（重点）

> 我的连招系统基于 Montage，通过 Section 控制不同攻击段，并利用 Notify 作为输入窗口判断是否接下一段攻击。

💥（这是面试杀招）

------

# 🔹 6. AnimNotify（动画通知）

------

## 📌 概念

在动画时间轴中触发事件。

------

## ⚙️ 原理

- 在指定帧触发回调
- 可调用：
  - C++
  - Blueprint

👉 常用于：

- 攻击判定
- 音效
- 特效

------

## 📂 源码位置

- `AnimNotify.h`

------

## 🎯 面试回答

> AnimNotify 用于在动画播放过程中触发事件，实现动画驱动逻辑，例如攻击判定。

------

## 💡 项目结合

> 我在攻击动画中用 Notify 触发伤害检测，确保判定与动画同步。

------

# 🔹 7. Root Motion（进阶）

------

## 📌 概念

由动画驱动角色移动。

------

## ⚙️ 原理

- 从骨骼 Root 提取位移
- 应用于角色

👉 区别：

- 普通：代码控制移动
- Root Motion：动画控制移动

------

## 📂 源码位置

- `CharacterMovementComponent.cpp`

------

## 🎯 面试回答

> Root Motion 将动画中的位移应用到角色，使动作更真实，但在网络同步中较复杂。

------

## 💡 项目结合

> 我在技能冲刺中使用 Root Motion，使角色移动与动画完全一致。

------

# 🔹 8. 动画驱动 vs 逻辑驱动（高频深问）

------

## 📌 概念

两种动画控制模式。

------

## ⚙️ 原理

### 动画驱动

- 动画 → 触发逻辑

### 逻辑驱动

- 逻辑 → 控制动画

------

## 📂 源码位置

- 无固定

------

## 🎯 面试回答（高级）

> 实际项目通常采用逻辑驱动为主，动画驱动为辅，例如用状态机控制动画播放，用 Notify 精确触发事件。

------

## 💡 项目结合（你的核心优势）

> 我的战斗系统是代码状态机驱动动画，同时使用 AnimNotify 做精确判定，这样既保证逻辑可控，又保证表现准确。

------

# 🔹 9. 动画性能优化

------

## 📌 概念

减少动画系统开销。

------

## ⚙️ 原理

- 降低更新频率
- LOD 动画
- 动画压缩

------

## 📂 源码位置

- Animation 优化模块

------

## 🎯 面试回答

> 动画优化主要通过减少计算、使用 LOD 和压缩数据来降低 CPU 和内存开销。

------

## 💡 项目结合

> 对远距离角色降低动画更新频率，提高性能。

# 🧠 模块四：UE5 网络同步（高频 + 深水区）

------

# 🔹 1. UE 网络模型（基础必问）

------

## 📌 概念

Unreal Engine 5 使用 **Client-Server 架构**进行网络同步。

------

## ⚙️ 原理

### 核心结构：

- Server（权威）
- Client（表现）

👉 所有核心逻辑在 Server：

- 位置
- 战斗
- 状态

Client：

- 输入 → 发送给 Server
- 接收结果 → 展示

👉 本质：
 **“服务器权威模型（Server Authoritative）”**

------

## 📂 源码位置

- `NetDriver.cpp`
- `World.cpp`

------

## 🎯 面试回答

> UE 采用服务器权威模型，客户端只负责输入和表现，所有关键逻辑在服务器执行，从而防止作弊并保证一致性。

------

## 💡 项目结合

> 我的战斗系统中，攻击判定在服务器执行，客户端只做动画播放，避免作弊问题。

------

# 🔹 2. Actor Replication（核心机制）

------

## 📌 概念

Actor 可以自动同步到客户端。

------

## ⚙️ 原理

### 开启同步：

```
bReplicates = true;
```

### 同步内容：

- 属性（UPROPERTY Replicated）
- 位置（Movement）
- RPC

👉 同步流程：

1. Server 修改数据
2. NetDriver 检测变化
3. 发送给 Client

👉 本质：
 **“状态同步（State Replication）”**

------

## 📂 源码位置

- `ActorReplication.cpp`
- `RepLayout.cpp`

------

## 🎯 面试回答

> Actor 通过开启 bReplicates 实现自动同步，引擎会检测属性变化并发送给客户端。

------

## 💡 项目结合

> 我将角色 Actor 设置为可复制，并同步血量、状态等关键数据。

------

# 🔹 3. 属性同步（Replicated / RepNotify）

------

## 📌 概念

用于同步变量。

------

## ⚙️ 原理

### 基础：

```
UPROPERTY(Replicated)
int Health;
```

### RepNotify：

```
UPROPERTY(ReplicatedUsing=OnRep_Health)
```

👉 流程：

- Server 改值
- Client 接收
- 调用 OnRep

------

## 📂 源码位置

- `RepLayout.cpp`

------

## 🎯 面试回答

> 属性同步通过 Replicated 标记，RepNotify 可以在客户端接收更新时执行回调，用于驱动表现层逻辑。

------

## 💡 项目结合

> 我用 RepNotify 更新血量 UI，而不是每帧同步 UI。

------

# 🔹 4. RPC（远程调用）（必问）

------

## 📌 概念

用于跨网络调用函数。

------

## ⚙️ 原理

### 三种类型：

#### Server RPC

```
UFUNCTION(Server)
```

👉 Client → Server

#### Client RPC

```
UFUNCTION(Client)
```

👉 Server → Client

#### Multicast RPC

```
UFUNCTION(NetMulticast)
```

👉 Server → 所有客户端

------

## 📂 源码位置

- `NetDriver.cpp`
- `ActorChannel.cpp`

------

## 🎯 面试回答

> RPC 用于在客户端和服务器之间传递行为，例如客户端发送输入到服务器，服务器再广播结果。

------

## 💡 项目结合（重点）

> 攻击输入通过 Server RPC 发送到服务器，服务器判定后用 Multicast 播放攻击动画。

💥（标准答案）

------

# 🔹 5. 网络角色（Role）（高频）

------

## 📌 概念

每个 Actor 在网络中有不同角色。

------

## ⚙️ 原理

### 类型：

- Authority（服务器）
- Autonomous Proxy（本地玩家）
- Simulated Proxy（其他玩家）

👉 控制权限：

- Authority 执行逻辑
- Proxy 只表现

------

## 📂 源码位置

- `Actor.h`

------

## 🎯 面试回答

> UE 通过 Role 区分 Actor 的网络身份，不同角色决定是否拥有执行权。

------

## 💡 项目结合

> 我只在 Authority 上执行伤害计算，客户端仅做预测或表现。

------

# 🔹 6. 网络同步流程（核心理解题）

------

## 📌 概念

完整的输入 → 同步 → 表现流程。

------

## ⚙️ 原理

### 流程：

1. Client 输入
2. 调用 Server RPC
3. Server 执行逻辑
4. 修改变量 / 触发事件
5. 同步到 Client

------

## 📂 源码位置

- NetDriver / ActorChannel

------

## 🎯 面试回答

> UE 网络同步流程是客户端发送输入到服务器，服务器处理逻辑后通过属性同步或 RPC 更新客户端。

------

## 💡 项目结合

> 我的技能系统中，客户端发起攻击请求，服务器判断命中并同步结果。

------

# 🔹 7. Character Movement 同步（高频难点）

------

## 📌 概念

角色移动的网络同步机制。

------

## ⚙️ 原理

UE 做了优化：

👉 Client Prediction（客户端预测）
 👉 Server Correction（服务器修正）

流程：

1. Client 先移动（预测）
2. 发送输入给 Server
3. Server 校验
4. 修正 Client

------

## 📂 源码位置

- `CharacterMovementComponent.cpp`

------

## 🎯 面试回答（加分）

> CharacterMovement 使用客户端预测和服务器校正机制，既保证响应速度，又保证数据一致性。

------

## 💡 项目结合

> 如果做多人 TPS，我会使用 UE 自带移动同步，而不是自己实现。

------

# 🔹 8. 网络优化（高阶）

------

## 📌 概念

减少带宽和延迟。

------

## ⚙️ 原理

### 方法：

- NetUpdateFrequency
- relevancy（相关性）
- dormancy（休眠）
- 压缩数据

------

## 📂 源码位置

- NetDriver

------

## 🎯 面试回答

> 网络优化通过减少同步频率、限制相关性和使用休眠机制降低带宽压力。

------

## 💡 项目结合

> 子弹可以只在附近玩家同步，远处不更新。

------

# 🔹 9. 常见问题（面试杀点）

------

## ❗ Q1：为什么不能在客户端改血量？

### 🎯 回答

> 因为客户端不具备权威，服务器才是唯一可信来源。

------

## ❗ Q2：RPC 和属性同步区别？

### 🎯 回答

> RPC 用于事件，属性同步用于状态。

------

## ❗ Q3：Multicast 可以客户端调用吗？

### 🎯 回答

> 不可以，必须由服务器调用。

# 🧠 模块五：UE5 性能优化（工程能力核心）

------

# 🔹 1. 性能分析工具（第一步永远是分析）

------

## 📌 概念

性能优化的前提是定位瓶颈，UE 提供多种分析工具。

------

## ⚙️ 原理

核心工具：

### 1️⃣ Stat 命令

```
stat unit
stat fps
stat game
```

👉 查看：

- Game Thread
- Render Thread
- GPU

------

### 2️⃣ Unreal Insights

- 线程级别分析
- 函数调用时间

------

### 3️⃣ ProfileGPU

- GPU 渲染分析

------

## 📂 源码位置

- `StatsSystem.cpp`
- `UnrealInsights`

------

## 🎯 面试回答

> 性能优化的第一步是定位瓶颈，我通常使用 Stat 查看线程开销，再用 Unreal Insights 深入分析函数级耗时。

------

## 💡 项目结合

> 在 TPS 项目中，我用 stat unit 发现 GameThread 过高，定位到 Tick 过多的问题。

------

# 🔹 2. Tick 优化（高频必问）

------

## 📌 概念

Tick 是每帧执行的函数，是 CPU 开销主要来源之一。

------

## ⚙️ 原理

- 每个 Actor / Component 都可能 Tick
- 数量多 → 线性增加 CPU 开销

优化方式：

- 关闭 Tick
- 降低频率
- 改事件驱动

------

## 📂 源码位置

- `TickTaskManager.cpp`

------

## 🎯 面试回答

> Tick 是性能热点，应尽量减少使用，优先采用事件驱动或降低 Tick 频率。

------

## 💡 项目结合（重点）

> 我的子弹系统不使用 Tick，而是用碰撞和生命周期控制，大幅减少 CPU 开销。

------

# 🔹 3. Draw Call 优化（GPU核心）

------

## 📌 概念

Draw Call 是 GPU 渲染调用次数。

------

## ⚙️ 原理

- 每次 Draw Call 都有 CPU → GPU 开销
- 数量多 → GPU bottleneck

优化方法：

- Instancing
- 合批（Batching）
- 减少材质数量

------

## 📂 源码位置

- `RendererModule.cpp`

------

## 🎯 面试回答

> Draw Call 是 GPU 性能瓶颈之一，可以通过实例化和合批减少调用次数。

------

## 💡 项目结合

> 大量子弹使用 Instanced Mesh 渲染，减少 Draw Call。

------

# 🔹 4. 对象池（高频工程题）

------

## 📌 概念

避免频繁创建和销毁对象。

------

## ⚙️ 原理

- 预创建对象
- 重复使用

👉 避免：

- malloc/free
- GC 压力

------

## 📂 源码位置

- 无固定（自实现）

------

## 🎯 面试回答

> 对象池通过复用对象避免频繁创建和销毁，降低 CPU 和内存开销。

------

## 💡 项目结合（你的强项）

> 我的 TPS 子弹系统使用对象池管理子弹，大幅降低性能开销。

💥（这题你是加分项）

------

# 🔹 5. 内存优化（GC相关）

------

## 📌 概念

减少内存占用和 GC 压力。

------

## ⚙️ 原理

- UObject 会被 GC 管理
- 数量过多 → GC 卡顿

优化：

- 减少 UObject 数量
- 使用非 UObject（struct）

------

## 📂 源码位置

- `GarbageCollection.cpp`

------

## 🎯 面试回答

> 大量 UObject 会增加 GC 开销，因此应尽量减少不必要的 UObject 使用。

------

## 💡 项目结合

> 子弹数据使用 struct 而不是 UObject，避免 GC 频繁触发。

------

# 🔹 6. LOD（细节层级）

------

## 📌 概念

根据距离降低模型复杂度。

------

## ⚙️ 原理

- 远处用低模
- 减少顶点数

------

## 📂 源码位置

- `StaticMesh.cpp`

------

## 🎯 面试回答

> LOD 通过降低远处物体的细节减少 GPU 开销。

------

## 💡 项目结合

> 场景中远距离敌人使用低精度模型。

------

# 🔹 7. Nanite（UE5核心优化）

------

## 📌 概念

UE5 的虚拟几何系统。

------

## ⚙️ 原理

- 自动 LOD
- 按像素级加载

👉 本质：
 **“GPU驱动的几何流式系统”**

------

## 📂 源码位置

- `NaniteRendering.cpp`

------

## 🎯 面试回答

> Nanite 通过虚拟化几何实现自动 LOD，减少人工优化成本。

------

## 💡 项目结合

> 场景静态模型可以使用 Nanite，减少建模和优化工作量。

------

# 🔹 8. 异步加载（卡顿关键）

------

## 📌 概念

避免同步加载导致卡顿。

------

## ⚙️ 原理

- 异步线程加载资源
- 主线程不阻塞

------

## 📂 源码位置

- `StreamableManager.cpp`

------

## 🎯 面试回答

> 异步加载通过后台线程加载资源，避免主线程卡顿。

------

## 💡 项目结合

> 技能特效使用异步加载，避免战斗中卡顿。

------

# 🔹 9. 网络优化（结合前一模块）

------

## 📌 概念

减少网络带宽消耗。

------

## ⚙️ 原理

- 降低同步频率
- relevancy
- dormancy

------

## 📂 源码位置

- NetDriver

------

## 🎯 面试回答

> 网络优化通过减少同步数据和频率来降低带宽压力。

------

## 💡 项目结合

> 远距离玩家不同步子弹细节。

------

# 🔹 10. CPU vs GPU 瓶颈判断（高频）

------

## 📌 概念

判断性能瓶颈在哪。

------

## ⚙️ 原理

用：

- stat unit

判断：

- Game 高 → CPU 问题
- GPU 高 → 渲染问题

------

## 📂 源码位置

- StatsSystem

------

## 🎯 面试回答

> 通过 stat unit 判断瓶颈是在 CPU 还是 GPU，再针对性优化。

------

## 💡 项目结合

> 我通过 stat unit 判断是 Tick 过多导致 CPU 瓶颈。

------

# 🔹 11. 常见优化思路总结（面试压轴）

------

## 📌 概念

整体优化策略。

------

## ⚙️ 原理

优先级：

1. 减少计算（Tick）
2. 减少渲染（Draw Call）
3. 减少内存（GC）
4. 减少 IO（加载）

------

## 🎯 面试回答（高级总结）

> 性能优化本质是减少不必要的计算、渲染和数据传输，并通过工具定位瓶颈后针对性优化。

------

## 💡 项目结合（终极杀招）

> 在我的 TPS 项目中，我通过对象池减少创建销毁、关闭 Tick 降低 CPU 开销，并用 Instancing 优化渲染，整体提升了性能。

# 🧠 模块六：C++ + UE 宏底层（反射 / 编译 / 对象系统）

------

# 🔹 1. UE 宏本质（UCLASS / UPROPERTY / UFUNCTION）

------

## 📌 概念

这些宏是 Unreal Engine 5 提供的**反射标记系统**，用于让类进入引擎体系。

------

## ⚙️ 原理（核心）

👉 关键点：
 **宏本身几乎不做事！**

真正流程：

### 1️⃣ 写代码

```
UCLASS()
class AMyActor : public AActor
```

### 2️⃣ UHT（Unreal Header Tool）解析

- 扫描宏
- 生成 `.generated.h`

### 3️⃣ 生成内容

- 注册 UClass
- 注册属性
- 注册函数

### 4️⃣ 编译进引擎

------

👉 本质：
 **“编译前代码生成（Code Generation）”**

------

## 📂 源码位置

- 宏定义：
  - `ObjectMacros.h`
- 生成代码：
  - `*.generated.h`

------

## 🎯 面试回答（标准 + 深度）

> UE 的反射宏本质是标记信息，真正的功能由 UHT 在编译前解析并生成代码实现，从而让 C++ 具备运行时反射能力。

------

## 💡 项目结合

> 我在战斗系统中使用 UPROPERTY 暴露参数，使技能数据可以在编辑器中配置，提高迭代效率。

------

# 🔹 2. UHT（Unreal Header Tool）

------

## 📌 概念

UHT 是 UE 的代码生成工具。

------

## ⚙️ 原理

编译流程：

```
C++代码 → UHT → 生成代码 → 编译
```

UHT 做的事：

- 解析 UCLASS 等宏
- 生成：
  - 注册函数
  - 反射信息
  - RPC代码

👉 关键：

- 不是运行时，是**编译前**

------

## 📂 源码位置

- `Engine/Source/Programs/UnrealHeaderTool`

------

## 🎯 面试回答

> UHT 在编译前解析 UE 宏并生成反射代码，使 UE 在 C++ 上实现类似 C# 的反射机制。

------

## 💡 项目结合

> RPC 函数其实也是 UHT 生成网络调用代码实现的。

------

# 🔹 3. 反射系统（Reflection）

------

## 📌 概念

UE 支持运行时获取类型信息。

------

## ⚙️ 原理

核心结构：

- UClass
- UProperty（FProperty）
- UFunction

👉 存储：

- 类名
- 属性列表
- 函数列表

支持：

- 编辑器
- 序列化
- 蓝图调用

------

## 📂 源码位置

- `UObject/Class.h`
- `UObject/Field.h`

------

## 🎯 面试回答

> UE 通过 UClass 和 FProperty 实现反射系统，使对象具备运行时类型信息，从而支持编辑器和蓝图。

------

## 💡 项目结合

> 我可以通过反射动态获取技能属性，实现通用配置系统。

------

# 🔹 4. GENERATED_BODY（核心细节题）

------

## 📌 概念

每个 UObject 类必须写的宏。

------

## ⚙️ 原理

👉 实际展开为：

- 构造函数声明
- 静态注册函数
- 反射信息

👉 关键：

- 把类注册进引擎

------

## 📂 源码位置

- `.generated.h`

------

## 🎯 面试回答

> GENERATED_BODY 宏展开后包含反射注册代码，使类能够被 UE 系统识别。

------

## 💡 项目结合

> 没有这个宏，类无法参与反射，也无法被编辑器识别。

------

# 🔹 5. UE 为什么不用 C++ RTTI（必问）

------

## 📌 概念

UE 自己实现了一套类型系统。

------

## ⚙️ 原理

原因：

### 1️⃣ 性能

- RTTI 开销较大

### 2️⃣ 不可控

- 无法扩展

### 3️⃣ 功能不足

- 不支持编辑器 / 序列化

👉 UE 选择：

- 自定义反射系统

------

## 📂 源码位置

- 类型系统代码

------

## 🎯 面试回答（加分）

> UE 不使用 RTTI，而是自实现反射系统，因为它更高效且可扩展，并能支持编辑器和序列化功能。

------

## 💡 项目结合

> 我在代码中使用 IsA 和 Cast<> 替代 dynamic_cast。

------

# 🔹 6. Cast<> 原理（高频）

------

## 📌 概念

UE 的类型转换函数。

------

## ⚙️ 原理

```
Cast<AMyActor>(Obj)
```

内部：

- 检查 UClass
- 判断 IsA()

👉 本质：
 **“基于反射的安全转换”**

------

## 📂 源码位置

- `Cast.h`

------

## 🎯 面试回答

> Cast<> 通过 UClass 判断类型，比 dynamic_cast 更高效。

------

## 💡 项目结合

> 在状态机中使用 Cast 判断当前状态类型。

------

# 🔹 7. UObject 内存分配

------

## 📌 概念

UObject 不使用标准 new。

------

## ⚙️ 原理

使用：

- GUObjectAllocator

流程：

1. 分配内存
2. 注册对象
3. 初始化

👉 与 GC 绑定

------

## 📂 源码位置

- `UObjectGlobals.cpp`

------

## 🎯 面试回答

> UObject 使用引擎自定义分配器，并注册到对象系统中，从而支持 GC。

------

## 💡 项目结合

> 我避免用 new 创建 UObject，而使用 NewObject。

------

# 🔹 8. FName / FString / FText（高频基础）

------

## 📌 概念

UE 自定义字符串系统。

------

## ⚙️ 原理

### FName

- 哈希 + 索引
- 快速比较

### FString

- 普通字符串

### FText

- 本地化

------

## 📂 源码位置

- `NameTypes.h`

------

## 🎯 面试回答

> FName 用于高效比较，FString 用于字符串操作，FText 用于本地化显示。

------

## 💡 项目结合

> 状态机中用 FName 标识状态，提高性能。

------

# 🔹 9. 模板与容器（UE STL）

------

## 📌 概念

UE 自带容器系统。

------

## ⚙️ 原理

常见：

- TArray
- TMap
- TSet

特点：

- 与 UE 内存系统兼容

------

## 📂 源码位置

- `Containers/Array.h`

------

## 🎯 面试回答

> UE 提供自己的容器以兼容内存管理和序列化系统。

------

## 💡 项目结合

> 我用 TArray 存储技能列表。

------

# 🔹 10. 蓝图调用 C++（底层）

------

## 📌 概念

蓝图可以调用 C++ 函数。

------

## ⚙️ 原理

通过：

```
UFUNCTION(BlueprintCallable)
```

UHT：

- 注册函数
- 蓝图调用时通过反射执行

------

## 📂 源码位置

- `UFunction`

------

## 🎯 面试回答

> 蓝图调用 C++ 是通过反射系统实现的，函数在运行时被查找并执行。

------

## 💡 项目结合

> 我将战斗接口暴露给蓝图，方便策划使用。

------

# 🔹 11. RPC 底层实现（加分）

------

## 📌 概念

RPC 本质也是反射系统。

------

## ⚙️ 原理

UHT：

- 生成网络调用代码

运行时：

- 序列化参数
- 发送网络包

------

## 📂 源码位置

- `NetDriver.cpp`

------

## 🎯 面试回答（高级）

> RPC 是通过 UHT 生成代码并结合反射系统实现的，本质是函数调用的序列化与网络传输。

------

## 💡 项目结合

> 攻击 RPC 实际是参数打包后发送到服务器执行。

------

# ✅ 最终总结（你必须能说出来）

------

# 💥 UE 本质一句话总结（面试王炸）

> Unreal Engine 在 C++ 之上构建了一套完整的对象系统，通过 UHT 进行代码生成，实现反射、序列化和网络功能，从而弥补 C++ 在运行时能力上的不足。

# GAS

# 🧠 一、GAS 总体结构（先建立全局认知）

------

## 📌 概念

GAS 是 Unreal Engine 5 提供的一套**数据驱动 + 网络友好**的技能系统框架。

------

## ⚙️ 本质架构

GAS =

```
Ability（行为）
+ Attribute（属性）
+ Effect（数值变化）
+ Tag（状态标记）
+ ASC（管理中心）
```

👉 核心思想：
 **“行为和数据完全解耦”**

------

# 🔹 二、AbilitySystemComponent（ASC）（核心中枢）

------

## 📌 概念

所有 GAS 功能的核心管理器。

------

## ⚙️ 原理

- 挂在 Actor 上（通常是 Character）
- 管理：
  - Ability
  - Attribute
  - Effect
  - Tag

👉 所有操作都通过 ASC：

```
ApplyEffect
ActivateAbility
AddTag
```

------

## 📂 源码位置

- `AbilitySystemComponent.h`

------

## 🎯 面试回答

> ASC 是 GAS 的核心组件，负责统一管理技能、属性和效果，是所有能力执行的入口。

------

## 💡 项目结合

> 如果我用 GAS，我会把 ASC 挂在角色上，作为战斗系统的核心。

------

# 🔹 三、Gameplay Ability（技能本体）

------

## 📌 概念

表示一个技能（攻击、闪避、施法等）。

------

## ⚙️ 原理

生命周期：

```
TryActivate → Activate → End
```

能力特点：

- 可预测（客户端）
- 可复制（网络）
- 可取消

👉 内部：

- Commit（消耗资源）
- 执行逻辑
- 应用 Effect

------

## 📂 源码位置

- `GameplayAbility.h`

------

## 🎯 面试回答

> GameplayAbility 表示一个技能行为，支持激活、执行和结束，并与网络同步和资源系统结合。

------

## 💡 项目结合

> 攻击技能可以封装为 Ability，里面处理播放动画、扣蓝、触发伤害。

------

# 🔹 四、Gameplay Effect（数值系统核心）

------

## 📌 概念

用于描述属性变化（伤害、回血、Buff）。

------

## ⚙️ 原理

Effect 包含：

- 修改属性
- 持续时间
- 叠加规则

类型：

- Instant（瞬时）
- Duration（持续）
- Infinite（永久）

👉 本质：
 **“数据驱动的数值修改器”**

------

## 📂 源码位置

- `GameplayEffect.h`

------

## 🎯 面试回答

> GameplayEffect 用于修改属性，是 GAS 中数值变化的核心，支持持续时间和叠加规则。

------

## 💡 项目结合

> 我会用 Effect 实现伤害和 Buff，而不是直接改血量。

------

# 🔹 五、Attribute（属性系统）

------

## 📌 概念

角色属性（HP、MP、攻击力等）。

------

## ⚙️ 原理

- 定义在 AttributeSet 中
- 通过 Effect 修改

👉 关键：

- 不直接修改属性
- 必须通过 Effect

------

## 📂 源码位置

- `AttributeSet.h`

------

## 🎯 面试回答

> Attribute 是角色数值，必须通过 GameplayEffect 修改，从而保证统一管理。

------

## 💡 项目结合

> 血量变化通过 Effect 修改，而不是直接赋值。

------

# 🔹 六、Gameplay Tag（状态系统）

------

## 📌 概念

标签系统（类似状态标记）。

------

## ⚙️ 原理

示例：

```
State.Stunned
Ability.Attack
```

用途：

- 判断状态
- 阻止技能
- 触发逻辑

------

## 📂 源码位置

- `GameplayTagContainer.h`

------

## 🎯 面试回答

> GameplayTag 是轻量级状态标记系统，用于控制技能条件和状态判断。

------

## 💡 项目结合

> 我可以用 Tag 标记“正在攻击”，防止重复释放技能。

------

# 🔹 七、Gameplay Cue（表现层）

------

## 📌 概念

用于播放特效、音效。

------

## ⚙️ 原理

- 与 Effect 绑定
- 在客户端触发

------

## 📂 源码位置

- `GameplayCueManager`

------

## 🎯 面试回答

> GameplayCue 用于表现层，如特效和音效，与逻辑解耦。

------

## 💡 项目结合

> 技能命中时触发特效，不影响逻辑。

------

# 🔥 三、GAS 完整工作流程（重点！）

------

# 🎯 从“按下攻击键”开始

------

## 🟢 Step 1：输入触发

```
Input → ASC.TryActivateAbility
```

------

## 🟢 Step 2：检查条件

- 是否有 Ability
- 是否满足 Tag
- 是否资源足够

------

## 🟢 Step 3：网络处理

- Client 预测执行
- Server 验证

------

## 🟢 Step 4：ActivateAbility

执行：

- 播放动画
- 扣蓝（Commit）
- 应用 Effect

------

## 🟢 Step 5：应用 GameplayEffect

```
ApplyGameplayEffectToTarget
```

→ 修改 Attribute

------

## 🟢 Step 6：触发 GameplayCue

- 播放特效 / 音效

------

## 🟢 Step 7：结束 Ability

```
EndAbility
```

------

# 🧠 总流程总结

```
输入
→ Ability 激活
→ 条件检查
→ 网络同步
→ 执行逻辑
→ 应用 Effect
→ 更新属性
→ 播放特效
→ 技能结束
```

------

# 💥 面试“王炸总结”

你必须能说这句话：

> GAS 的核心流程是通过 ASC 激活 Ability，在服务器验证后执行逻辑，通过 GameplayEffect 修改属性，并用 GameplayCue 处理表现，从而实现逻辑与表现分离的技能系统。

------

# 🔥 面试高频深问（非常关键）

------

## ❗ Q1：为什么 Attribute 不能直接改？

👉 答：

> 因为需要统一管理数值变化，并支持网络同步和 Buff 叠加。

------

## ❗ Q2：Ability 和 Effect 区别？

👉 答：

> Ability 是行为，Effect 是数值变化。

------

## ❗ Q3：GAS 为什么复杂？

👉 答：

> 因为它支持数据驱动、网络同步和扩展性，适用于大型项目。

👉 **GAS 的数值计算核心在：GameplayEffect + AttributeSet + Aggregator（属性聚合器）**

------

# 🧩 一、数值计算到底在哪一层做？

------

## 📌 概念

在 Gameplay Ability System 中：

👉 **所有数值变化（伤害、加攻、Buff）都是由 GameplayEffect 完成的**

------

## ⚙️ 分层理解（非常重要）

```
Ability（行为）
↓
GameplayEffect（定义数值变化）
↓
Aggregator（计算数值）
↓
AttributeSet（存储结果）
```

------

👉 所以：

| 层级       | 作用         |
| ---------- | ------------ |
| Ability    | 触发         |
| Effect     | 描述“改多少” |
| Aggregator | 真正算       |
| Attribute  | 存结果       |

------

# 🔹 二、GameplayEffect 是怎么“描述数值”的？

------

## 📌 概念

GameplayEffect 本身**不直接计算数值**，而是定义“修改规则”。

------

## ⚙️ 原理

一个 Effect 包含：

- 修改哪个属性（Health）
- 修改方式：
  - Add
  - Multiply
  - Override
- 数值来源：
  - 固定值
  - ScalableFloat
  - 计算类（Execution）

------

👉 示例逻辑：

```
Damage = Attack * 1.5 - Defense
```

这个不会写在 Ability 里，而是：

👉 写在 Effect 或 Execution 中

------

## 🎯 面试回答

> GameplayEffect 只定义属性修改规则，并不会直接执行计算，实际计算由底层的属性聚合器完成。

------

# 🔹 三、Aggregator（核心！真正算数值的地方）

------

## 📌 概念

Aggregator 是 GAS 内部用于**合并多个数值修改的计算器**。

------

## ⚙️ 原理（核心重点）

每个属性都有一个 Aggregator：

```
FinalValue = BaseValue 
           + Additive 
           * Multiplicative 
           + Override
```

------

### 实际分层：

GAS 会把所有 Effect 拆成：

- Add（加法）
- Mul（乘法）
- Override（覆盖）

------

### 计算顺序（非常重要）：

```
((Base + Add) * Mul) → Override
```

------

👉 举例：

- Base = 100
- +20（Add）
- ×1.5（Mul）

结果：

```
(100 + 20) * 1.5 = 180
```

------

## 📂 源码位置

- `GameplayEffectAggregator.cpp`
- `GameplayEffectTypes.h`

------

## 🎯 面试回答（必须会）

> GAS 使用 Aggregator 聚合所有 GameplayEffect 对属性的修改，并按加法、乘法、覆盖的顺序计算最终数值。

------

## 💡 项目结合

> 如果我实现 Buff 系统，我会把所有 Buff 作为 Effect，让 Aggregator 自动计算最终属性，而不是手动叠加。

------

# 🔹 四、AttributeSet（结果存储 + 回调）

------

## 📌 概念

AttributeSet 存储最终计算结果。

------

## ⚙️ 原理

- 每个属性：
  - BaseValue
  - CurrentValue
- 当值变化时：
  - 触发回调

```
PreAttributeChange
PostGameplayEffectExecute
```

------

👉 关键点：

- Attribute 不负责计算
- 只负责：
  - 存
  - 通知

------

## 📂 源码位置

- `AttributeSet.h`

------

## 🎯 面试回答

> AttributeSet 负责存储属性值，并在数值变化时提供回调接口，但不参与具体计算。

------

## 💡 项目结合

> 我可以在 PostGameplayEffectExecute 中处理死亡逻辑。

------

# 🔹 五、Execution Calculation（高阶：复杂公式）

------

## 📌 概念

用于处理复杂计算（如伤害公式）。

------

## ⚙️ 原理

当简单 Add/Mul 不够时：

👉 使用：

```
GameplayEffectExecutionCalculation
```

------

### 流程：

1. 从 Attribute 读取数据
2. 自定义计算
3. 输出结果

------

👉 示例：

```
Damage = Attack * SkillRatio - Defense
```

------

## 📂 源码位置

- `GameplayEffectExecutionCalculation.h`

------

## 🎯 面试回答（加分）

> 对于复杂公式，GAS 提供 ExecutionCalculation，用于自定义数值计算逻辑。

------

## 💡 项目结合（重点）

> 我的技能伤害可以通过 ExecutionCalculation 实现，而不是写死在代码里。

------

# 🔥 六、完整数值计算流程（必须能讲出来）

------

## 🎯 从“造成伤害”开始

------

### 🟢 Step 1：Ability 触发

```
ActivateAbility
```

------

### 🟢 Step 2：创建 GameplayEffect

- 定义：
  - 修改 Health
  - 数值规则

------

### 🟢 Step 3：应用 Effect

```
ApplyGameplayEffectToTarget
```

------

### 🟢 Step 4：Aggregator 收集所有 Modifier

- Buff
- Debuff
- 装备

------

### 🟢 Step 5：执行计算

```
FinalValue = ((Base + Add) * Mul)
```

或：

👉 ExecutionCalculation

------

### 🟢 Step 6：写入 Attribute

------

### 🟢 Step 7：触发回调

- UI更新
- 死亡检测

------

# 🧠 总流程一句话总结

> GAS 的数值计算流程是：GameplayEffect 定义修改规则，Aggregator 聚合并计算最终值，然后写入 AttributeSet 并触发回调。

------

# 💥 面试“杀手级理解点”（非常重要）

------

## 🔥 关键认知 1

👉 GAS 是：
 **“声明式数值系统（Declarative）”**

不是：
 ❌ 手动算数值

------

## 🔥 关键认知 2

👉 Aggregator = 核心

大部分人只会说 Effect，但：

💥 **真正算数值的是 Aggregator**

------

## 🔥 关键认知 3

👉 Attribute 不参与计算

很多人会答错这里

------

# 🚀 最后：如果面试官继续深挖

可能会问：

------

## ❗ Q：多个 Buff 怎么叠加？

👉 答：

> 通过 Aggregator 按 Add/Mul 规则自动叠加。

------

## ❗ Q：伤害公式写在哪？

👉 答：

> 简单用 Modifier，复杂用 ExecutionCalculation。

------

## ❗ Q：为什么不用直接改血？

👉 答：

> 因为 GAS 需要统一管理数值变化，并支持网络同步和叠加规则。