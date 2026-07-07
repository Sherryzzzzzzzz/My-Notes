# Motion Matching 入门：你的第一个 60 分钟

Motion Matching 入门指南。我们将介绍角色蓝图设置、Motion Matching 资产，以及如何使用程序化修正来弥补动画覆盖不足的问题。我们还会讨论如何调试 Motion Matching 设置，以及一系列其他技巧和窍门。

![Te](assets/c377db1a-605c-46af-aa46-fddf445a2af9.jpeg

![Your First 60 Minutes With Motion Matching](assets/0c301cf5-2320-4750-83f5-56bf53cafe37.jpeg)



### 引言

本教程将帮助你熟悉 Unreal 中的 Motion Matching。由于使用的动画集非常有限，完成后的项目预期达不到可发布的质量水准。然而，我们涵盖的方法和原则应该能帮助你用自己的资产达到那个水准。

### 前置条件

在开始本教程之前，请下载 [Lyra Starter Game](https://www.unrealengine.com/marketplace/en-US/learn/lyra) 项目，因为我们将使用其中的一些资产。你还需要基于[第三人称模板](https://dev.epicgames.com/documentation/en-us/unreal-engine/third-person-template-in-unreal-engine)创建一个新项目，然后确保启用以下插件：

- Pose Search
- Chooser
- Animation Insights
- Animation Warping



如果你只对如何在现有项目中设置 Motion Matching 感兴趣，可以直接跳转到"设置你的 Motion Matching 资产"部分。

### 开始使用

从第三人称模板创建新项目后，你需要一些资产来开始。打开 Lyra Starter Game 项目，选择以下动画序列，并使用内容浏览器中的右键 Migrate 工具将它们迁移到你的新项目中。



确保将所有依赖资产（包括网格体、Control Rig 等）一并迁移到新项目中。

|                       |                                                              |                                                              |                                                              |                                                              |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 待机（Idles）         | 起步（Starts）                                               | 停步（Stops）                                                | 移动（Locomotion）                                           | 转身（Pivots）                                               |
| MM_Rifle_Idle_HipFire | MM_Rifle_Walk_Bwd_StartMM_Rifle_Walk_Fwd_StartMM_Rifle_Walk_Left_StartMM_Rifle_Walk_Right_StartMM_Rifle_Jog_Bwd_StartMM_Rifle_Jog_Fwd_StartMM_Rifle_Jog_Left_StartMM_Rifle_Jog_Right_Start | MM_Rifle_Walk_Bwd_StopMM_Rifle_Walk_Left_StopMM_Rifle_Walk_Left_StopMM_Rifle_Walk_Right_StopMM_Rifle_Jog_Bwd_StopMM_Rifle_Jog_Fwd_StopMM_Rile_Jog_Left_StopMM_Rile_Jog_Right_Stop | MM_Rifle_Walk_BwdMM_Rifle_Walk_FwdMM_Rifle_Walk_LeftMM_Rifle_Walk_RightMM_Rifle_Jog_BwdMM_Rifle_Jog_FwdMM_Rifle_Jog_LeftMM_Rifle_Jog_Right | MM_Rifle_Walk_Bwd_PivotMM_Rifle_Walk_Fwd_PivotMM_Rifle_Walk_Left_PivotMM_Rifle_Walk_Right_PivotMM_Rifle_Jog_Bwd_PivotMM_Rifle_Jog_Fwd_PivotMM_Rifle_Jog_Left_PivotMM_Rifle_Jog_Right_Pivot |

[![右键 > 资产操作 > 迁移](assets/2e7e37f8-1562-44d5-ac3e-09129f59e722.png)](https://dev.epicgames.com/community/api/learning/image/2e7e37f8-1562-44d5-ac3e-09129f59e722?resizing_type=fit)



这是一个小规模的动画集合。我们只有最少的稳态走和跑动画覆盖。一个完整的生产级 Motion Matching 设置通常需要比这多得多的动画覆盖，即使是在处理稀疏数据集时也是如此。

### 角色蓝图

现在你已经迁移了动画资产，可以打开你的新项目了。在本教程中，我们将把 BP_ThirdPersonCharacter 改造成我们的玩家角色。你可以在项目的 Content 目录中找到这个蓝图，路径为 /Content/ThirdPerson/Blueprints：

[![img](assets/9bc9ae12-8cb5-4dfd-a762-4681db58dd49.png)](https://dev.epicgames.com/community/api/learning/image/9bc9ae12-8cb5-4dfd-a762-4681db58dd49?resizing_type=fit)

#### 组件设置

打开角色蓝图后，进行以下更改。

1. 选择 Skeletal Mesh Component，将 Mesh 更改为 SKM_Manny

   [![img](assets/d8cf4ffb-e81b-4062-8202-3aa84f6fffb0.png)](https://dev.epicgames.com/community/api/learning/image/d8cf4ffb-e81b-4062-8202-3aa84f6fffb0?resizing_type=fit)

2. 在组件列表中选择 BP_ThirdPersonCharacter，设置 Use Controller Rotation Yaw

   [![img](assets/2d7c41b1-78e7-4607-b495-0d3b0eb06a5b.png)](https://dev.epicgames.com/community/api/learning/image/2d7c41b1-78e7-4607-b495-0d3b0eb06a5b?resizing_type=fit)

3. 选择 Character Movement 组件，更改以下值：

   1. 设置 Max Acceleration = 800.0

   2. Max Walk Speed = 600.0

   3. Min Analog Walk Speed = 300.0

      [![img](assets/0b9af506-6100-41b2-987e-337c3cc17644.png)](https://dev.epicgames.com/community/api/learning/image/0b9af506-6100-41b2-987e-337c3cc17644?resizing_type=fit)

   4. 禁用 Orient Rotation To Movement

      [![img](assets/e230001f-fd52-45ff-a22c-94d980557e07.png)](https://dev.epicgames.com/community/api/learning/image/e230001f-fd52-45ff-a22c-94d980557e07?resizing_type=fit)

#### 设置输入

接下来，我们需要确保控制器的输入能产生与我们已有动画覆盖相匹配的移动效果。

##### 输入映射

回到内容浏览器，找到 IMC_Default 输入映射上下文资产。它位于 /ThirdPerson/Input/ 目录下。

[![img](assets/b6f27231-4cb1-4611-81ca-1e2488927be5.png)](https://dev.epicgames.com/community/api/learning/image/b6f27231-4cb1-4611-81ca-1e2488927be5?resizing_type=fit)

打开输入映射上下文，展开 Gamepad Left Thumbstick 2D-Axis 部分，然后展开 Modifiers。在第二个 Index 中，将 Scalar 值改为 1,1,1，使角色输入比例变化更加离散，这样我们就能将其重映射到动画的速度上。

[![img](assets/9b9f8a2f-96b4-4670-8e9d-ba96d36eff42.png)](https://dev.epicgames.com/community/api/learning/image/9b9f8a2f-96b4-4670-8e9d-ba96d36eff42?resizing_type=fit)

##### 事件图

回到 BP_ThirdPersonCharacter，进入事件图。进行以下更改：

1. 移除跳跃功能，因为与本教程无关

2. 添加三个变量，用于限制来自控制器的输入比例

   1. 一个 float 变量，命名为 RunScale。设为 1.0

   2. 第二个 float 变量，命名为 WalkScale。设为 0.5

   3. 最后一个 float 变量，命名为 RunScaleThreshold。设为 0.75

      [![img](assets/a3913aa3-2484-4a7a-84cb-3f1dabff7ace.png)](https://dev.epicgames.com/community/api/learning/image/a3913aa3-2484-4a7a-84cb-3f1dabff7ace?resizing_type=fit)

3. 添加一个名为 ClampInputScale 的函数。我们将使用这个函数来限制来自控制器的输入比例。当输入比例高于 RunScaleThreshold 时，限制到 RunScale；低于时，限制到 WalkScale。

   1. 该函数应接受一个 Vector2D 变量作为输入。命名为 InputScale。

   2. 它还应输出一个 Vector2D，命名为 ReturnValue。

   3. 函数图应如下所示：

      [![img](assets/d9066dce-c13d-478f-bfdb-27060d2138ed.png)](https://dev.epicgames.com/community/api/learning/image/d9066dce-c13d-478f-bfdb-27060d2138ed?resizing_type=fit)

   4. 回到事件图的顶层，将 ClampInputScale 函数连接到 IA_Move 事件：

      [![img](assets/71aae45b-88ed-4777-b86a-aa984de5c6fe.png)](https://dev.epicgames.com/community/api/learning/image/71aae45b-88ed-4777-b86a-aa984de5c6fe?resizing_type=fit)



这些对组件和事件图的更改构建了一个与我们动画覆盖尽可能匹配的移动模型。我们的走路动画以 300cm/s 移动，跑步以 600cm/s 移动，因此我们在移动组件上设置了相应的值。将输入比例限制在 0.5 或 1.0，会强制角色以 300cm/s 或 600cm/s 的速度移动。虽然在 Motion Matching 中并非必须限制输入比例——你的角色仍然可以有连续的速度值——但这样做可以获得更好的效果，尤其是在动画覆盖有限的情况下。

### 设置你的 Motion Matching 资产

#### Schema

现在我们要设置角色将要使用的 Schema 和数据库。

为你的 Motion Matching 资产创建一个新文件夹后，创建一个 Pose Search Schema。这个资产包含了姿态搜索算法用于分析动画序列的规则。

将从 Lyra 项目迁移过来的 Mannequin 骨骼设置为目标骨骼。将资产命名为 PSS_Mannequin。

[![img](assets/67b15517-5de6-4e83-8e62-0c77a66c48e9.png)](https://dev.epicgames.com/community/api/learning/image/67b15517-5de6-4e83-8e62-0c77a66c48e9?resizing_type=fit)

Schema 包含了在数据库中分析动画的规则。在本示例中，我们将保留默认值，这些默认值对于一般的移动行为应该是合适的。

#### 数据库

##### 走路（Walks）

接下来，创建第一个 Pose Search Database。指向你刚刚创建的 Schema，将资产命名为 PSD_Walks。

[![img](assets/ff1c8f29-245e-42c5-aa03-a44bdd854be2.png)](https://dev.epicgames.com/community/api/learning/image/ff1c8f29-245e-42c5-aa03-a44bdd854be2?resizing_type=fit)

这个数据库将只包含你的走路循环和相应的转身动画。我们将为跑步创建另一个数据库，并为待机+停步和起步分别创建独立的数据库。像这样拆分数据库有助于在稀疏数据集的情况下引导 Motion Matching 系统选择正确的动画帧。

在内容浏览器中，选择 MM_Rifle_Walk_Fwd 及其 Bwd、Left、Right 变体，以及相应的转身走路动画。总共应该是八个序列。将它们拖入数据库的 Asset List 中。

数据库应该会根据 Schema 中列出的通道自动分析这些序列。如果你在 Asset List 中选择这些剪辑，它们应该在数据库视口中播放。

现在在数据库中进行以下更改：

1. 将非转身动画设置为循环。为此，你可能需要在动画编辑器中打开动画序列（在数据库中双击资产即可打开动画序列）。
2. 在非转身动画上禁用 'Reselection of Poses' 标志
3. 将 Looping Cost Bias 设置为 -0.01。这会降低 Motion Matching 系统从循环动画中选择帧的成本。换句话说，它降低了选择转身动画的可能性。

你的 PSD_Walks 数据库现在应该如下所示：



[![img](assets/6cf3679d-992b-4f1a-9813-f9e6be66e420.png)](https://dev.epicgames.com/community/api/learning/image/6cf3679d-992b-4f1a-9813-f9e6be66e420?resizing_type=fit)

##### 跑步（Jogs）

现在，创建第二个数据库，命名为 PSD_Jogs。然后，对相应的跑步和跑步转身动画重复上述过程。确保再次设置 Looping 和 Reselection of Poses 标志。你的数据库应如下所示：



[![img](assets/7ac98e02-7052-4544-a55f-31f6af96d144.png)](https://dev.epicgames.com/community/api/learning/image/7ac98e02-7052-4544-a55f-31f6af96d144?resizing_type=fit)

##### 待机与停步（Idle And Stops）

现在创建第三个数据库，命名为 PSD_IdleAndStops。将待机动画和所有停步变体拖入数据库。

设置与之前相同的标志和属性，使数据库如下所示：



[![img](assets/72deabcd-d703-4c18-b1d1-7d64c173758c.png)](https://dev.epicgames.com/community/api/learning/image/72deabcd-d703-4c18-b1d1-7d64c173758c?resizing_type=fit)

创建最后一个数据库，命名为 PSD_Starts。将所有起步变体动画拖入。

再次设置标志和属性，使数据库如下所示：



[![img](assets/b023ad72-aa23-4b23-a36b-21fababb6b0a.png)](https://dev.epicgames.com/community/api/learning/image/b023ad72-aa23-4b23-a36b-21fababb6b0a?resizing_type=fit)

#### 归一化

最后，我们需要创建一个 Pose Search Normalization 资产。当你需要动态切换活动数据库时，应使用此资产，我们稍后将进行设置。

创建归一化资产后，将其命名为 PSN_Locomotion，并将所有现有数据库添加进去。注意：你还需要打开每个数据库，并在其上设置归一化资产。

[![img](assets/98f6a5e0-aef3-4bd8-ab54-6e230f64cd6b.png)](https://dev.epicgames.com/community/api/learning/image/98f6a5e0-aef3-4bd8-ab54-6e230f64cd6b?resizing_type=fit)

### 设置你的动画蓝图

现在 Motion Matching 资产已经设置好了，是时候在动画蓝图中使用它们了。

创建一个新的动画蓝图，命名为 ABP_Mannequin。将目标骨骼设置为我们从 Lyra Starter Project 迁移过来的 Mannequin 骨骼（SK_Mannequin）。

返回角色蓝图（BP_ThirdPersonCharacter），在 Skeletal Mesh Component 上设置新的动画蓝图。

[![img](assets/1dab30a5-5fa0-48b6-b275-f3432982aeaf.png)](https://dev.epicgames.com/community/api/learning/image/1dab30a5-5fa0-48b6-b275-f3432982aeaf?resizing_type=fit)

#### Motion Matching 节点

回到动画蓝图，进入动画图，放入一个 Motion Matching 节点。在 Database 输入引脚上，指定 PSD_IdleAndStops 作为数据库。

[![img](assets/80e61d38-2fe6-4183-b198-4c9b4fa58fa9.png)](https://dev.epicgames.com/community/api/learning/image/80e61d38-2fe6-4183-b198-4c9b4fa58fa9?resizing_type=fit)

Motion Matching 节点是任何 Motion Matching 实现的核心。该节点将执行分析，以确定哪个动画数据帧与轨迹输入最匹配。

#### Pose History 节点

接下来，我们需要添加一个 Pose History 节点。

[![img](assets/073409ba-673d-4ffc-a421-cd10ef2d3fb4.png)](https://dev.epicgames.com/community/api/learning/image/073409ba-673d-4ffc-a421-cd10ef2d3fb4?resizing_type=fit)

#### 生成轨迹数据

现在我们需要生成一些轨迹数据，以便馈送到 Pose History 节点。

向动画蓝图添加一个类型为 Pose Search Query Trajectory 的变量，命名为 Trajectory：

[![img](assets/921dc558-31a5-4962-b157-a2ba7237db32.png)](https://dev.epicgames.com/community/api/learning/image/921dc558-31a5-4962-b157-a2ba7237db32?resizing_type=fit)

现在，创建一个名为 GenerateTrajectory 的新函数。我们将使用它来收集轨迹数据。放入一个 Pose Search Generate Trajectory 节点，然后按如下方式连接：

1. 将 Trajectory 变量连接到 In Out Trajectory
2. 从 In Trajectory Data 引脚创建一个变量，重命名为 Trajectory Generation Data
3. 将 In Delta Time 引脚连接到 Get Delta Seconds 函数
4. 将 In Anim Instance 连接到 Self 节点
5. 从 In Out Desired Controller Yaw Last Update 引脚创建一个变量，命名为 PreviousDesiredControllerYaw
6. 连接函数的其余部分并设置 Trajectory 变量

你的 GenerateTrajectory 函数现在应如下所示：



[![img](assets/2865d4b6-af25-4b6f-a91a-acae5cb1a069.png)](https://dev.epicgames.com/community/api/learning/image/2865d4b6-af25-4b6f-a91a-acae5cb1a069?resizing_type=fit)



你可以避免通过 Pose Search Generate Trajectory 节点生成轨迹，而是直接通过 Pose History 节点上的 'Generate Trajectory' 属性来完成。这目前是一个实验性功能。

现在，在动画蓝图的事件图中，将 GenerateTrajectory 函数连接到 Event Blueprint Update Animation。



[![img](assets/746acbfb-1e17-417f-8a89-46cf8cb3e4c0.png)](https://dev.epicgames.com/community/api/learning/image/746acbfb-1e17-417f-8a89-46cf8cb3e4c0?resizing_type=fit)

回到动画图，将 Pose History 节点上的 Trajectory 引脚设置为使用 Trajectory 变量。

[![img](assets/93240c4a-759f-4733-a9d4-ffabb03cd0d0.png)](https://dev.epicgames.com/community/api/learning/image/93240c4a-759f-4733-a9d4-ffabb03cd0d0?resizing_type=fit)

### 选择数据库

现在我们要设置一些逻辑，根据玩家角色的移动情况，有上下文地切换传递给 Motion Matching 节点的活动数据库。但首先，让我们 PIE 运行一下，看看当前设置的效果。

你应该能看到 Lyra 网格体在播放你的待机动画。Motion Matching 节点从 PSD_IdleAndStops 数据库中的数据选择了该动画。这在待机时很好，但一旦我们移动角色，显然就需要将数据库切换为包含我们移动动画的数据库。

#### Chooser 表

我们将使用 Chooser Table 来切换馈送给 Motion Matching 节点的数据库。Chooser 允许我们根据一系列输入来有上下文地切换资产。它们类似于数据资产，但包含决定输出哪个资产的内部逻辑。

在内容浏览器中创建一个 Chooser Table（右键 > Miscellaneous > Chooser Table），命名为 CHT_PoseSearchDatabases。

打开 Chooser，进入 Table Settings：

1. 将 Fallback Result 设置为 Asset 类型，指定 PSD_IdleAndStops
2. 在 Context Data 中添加一个条目，指定 Context Object Type Class，输入你的动画蓝图（ABP_Mannequin）。这允许 Chooser 从你的动画蓝图中拉取数据。
3. Output Object Type 应为 PoseSearchDatabase。这是我们希望 Chooser 输出的资产类型

[![img](assets/51d83056-aea8-48d5-958b-035840b7c707.png)](https://dev.epicgames.com/community/api/learning/image/51d83056-aea8-48d5-958b-035840b7c707?resizing_type=fit)

现在添加四个 Asset 行，并分别指定你的每个数据库。这里的顺序很重要。当多行关联的条件都为真时，将返回最上面一行的资产。

[![img](assets/ca71976d-279c-4c8e-bb63-0cf3949da1df.png)](https://dev.epicgames.com/community/api/learning/image/ca71976d-279c-4c8e-bb63-0cf3949da1df?resizing_type=fit)

#### 收集一些状态数据

现在，我们需要在动画蓝图中添加一些变量，用于确定 Chooser 应输出哪个数据库。

回到动画蓝图，添加一个类型为 Character Movement Component 的变量，命名为 CharacterMovement。在事件图中，按如下方式设置 CharacterMovement：

[![img](assets/4feb3a8d-dd4d-4417-8e6a-3148608faf16.png)](https://dev.epicgames.com/community/api/learning/image/4feb3a8d-dd4d-4417-8e6a-3148608faf16?resizing_type=fit)

现在添加以下变量：

1. 两个 boolean 变量：StartedMoving、HasAcceleration
2. 三个 float 变量：Acceleration、LastFrameAcceleration 和 Speed

接下来，添加一个名为 UpdateVariables 的函数。我们将在这里设置我们添加的所有变量。为该函数添加一个名为 DeltaTime 的 float 输入。

1. 按如下方式设置 Acceleration 和 LastFrameAcceleration：

   [![img](assets/1ecd0721-c9aa-4c1f-afd8-70089bcfb96c.png)](https://dev.epicgames.com/community/api/learning/image/1ecd0721-c9aa-4c1f-afd8-70089bcfb96c?resizing_type=fit)

2. 现在按如下方式计算 HasAcceleration 和 StartedMoving：

   [![img](assets/18014367-15ef-4f56-b115-9ed4fd7571fa.png)](https://dev.epicgames.com/community/api/learning/image/18014367-15ef-4f56-b115-9ed4fd7571fa?resizing_type=fit)

3. 最后按如下方式设置 Speed：

   [![img](assets/508bbc06-51c7-497e-9274-f6473b77a84d.png)](https://dev.epicgames.com/community/api/learning/image/508bbc06-51c7-497e-9274-f6473b77a84d?resizing_type=fit)



在上述步骤 1 中，使用了 PropertyAccess 节点来直接在 CharacterMovement 上调用 GetCurrentAcceleration。

回到事件图，连接 Update Variables 函数。

[![img](assets/c65cfb0b-a162-416e-892a-fa7ba2b525b0.png)](https://dev.epicgames.com/community/api/learning/image/c65cfb0b-a162-416e-892a-fa7ba2b525b0?resizing_type=fit)

#### 完成 Chooser 设置

现在我们可以完成 Chooser 中的设置。回到 Chooser 资产，执行以下操作：

1. 添加两个 Bool 列，分别绑定到 StartedMoving 和 HasAcceleration 变量

2. 添加一个 Float Range 列，绑定到 Speed 变量

3. 设置值如下：

   [![img](assets/11a07106-d9d4-480f-81ff-99c11cda5c1c.png)](https://dev.epicgames.com/community/api/learning/image/11a07106-d9d4-480f-81ff-99c11cda5c1c?resizing_type=fit)

4. 



这里的思路是让 Chooser 在角色没有加速度时（即正在停步或已停止）选择 IdleAndStops 数据库，在角色开始移动的那一帧选择 Starts 数据库，在指定速度范围内移动时选择 Walks 或 Jogs 数据库。走路动画的速度约为 300cm/s，因此当角色在 0 到 450.0cm/s 之间移动时，这些动画是最接近的匹配项。当角色移动速度超过 450.0cm/s 时，跑步是最接近的匹配项。

#### 设置数据库

最后，我们要将 Chooser 选择的数据库设置到 Motion Matching 节点上。

回到顶层动画图，选择 Motion Matching 节点。在细节面板中，选择 On Update 函数并点击 Create Binding。这将创建一个新的动画节点函数，每次 Motion Matching 节点更新时都会被调用。将该函数重命名为 MotionMatchingOnUpdate：



[![img](assets/97e2d871-d3e1-45ad-8005-cf40ea19d2bd.png)](https://dev.epicgames.com/community/api/learning/image/97e2d871-d3e1-45ad-8005-cf40ea19d2bd?resizing_type=fit)

在函数内部，你可以设置逻辑来从 Chooser 表中获取 Motion Matching 节点的活动数据库。注意，我们暂时将 Interrupt Mode 设置为 Force Interrupt And Invalidate Continuing Pose；我们稍后会回到这一点。

[![img](assets/2e9f4aed-749d-4abe-8cc5-9c27c4be5661.png)](https://dev.epicgames.com/community/api/learning/image/2e9f4aed-749d-4abe-8cc5-9c27c4be5661?resizing_type=fit)

现在，让我们测试一下有上下文地设置活动数据库的效果。点击 PIE 并移动角色。你应该能看到，当角色移动时，Motion Matching 节点的输入数据库会从 PSD_IdleAndStops 切换到 PSD_Starts，然后根据角色速度切换到 PSD_Walks 或 PSD_Jogs。然而，还有很多工作要做。

### Interrupt Mode（中断模式）

我们之前简要讨论了 Set Database To Search 节点上的 Interrupt Mode。这个属性控制 Motion Matching 系统如何处理数据库的更改——来自上一帧（和上一个数据库）的持续姿态在当前帧的选择中是否仍然有效，还是应该使之前的选择无效，只考虑来自新数据库的动画作为当前帧的姿态？

为了改善 Motion Matching 输出的外观，我们只想在回到待机状态时强制中断，因为在那种情况下我们绝不希望继续播放移动剪辑。

回到动画蓝图，进入 MotionMatchingOnUpdate 函数。

将 Interrupt Mode 引脚提升为变量：

[![img](assets/ce73cc2d-8ed6-4b6f-a6f9-f0607e531094.png)](https://dev.epicgames.com/community/api/learning/image/ce73cc2d-8ed6-4b6f-a6f9-f0607e531094?resizing_type=fit)

现在进入 UpdateVariables 方法，添加以下逻辑，根据角色是否有加速度（即是否在移动）来设置 Interrupt Mode：

[![img](assets/6a1c0bde-7709-4f19-a544-3cfaae72b2d0.png)](https://dev.epicgames.com/community/api/learning/image/6a1c0bde-7709-4f19-a544-3cfaae72b2d0?resizing_type=fit)

如果回到 PIE，你现在应该会发现移动数据库之间的切换更加无缝了。待机和停步动画会在角色停止时立即触发。

### 用 Animation Warping 修复覆盖不足

我们现在有了一个可以运行的 Motion Matching 设置。然而，移动动画提供的覆盖不足以满足我们的移动模型。我们只有两个不同的速度集合（走路和跑步），每个只有四个基本方向。我们可以通过使用 Animation Warping 节点来填补这些空白。

#### Orientation Warping（朝向变形）

首先，我们要修复基本方向（前、后、左、右）之外的覆盖不足。Orientation Warping 动画节点允许我们通过程序化地修改角色的朝向和脚部位置来生成这些基本方向之间的中间姿态。

返回顶层动画图，双击 Motion Matching 节点。这将打开 Motion Matching 节点下的子图。

与其他 Blend Stack 节点类型一样，Motion Matching 节点包含一个可以自定义的子图，用于修改输入变换。目前，子图被设置为不做任何修改地输出来自 Motion Matching 算法的输入变换，但我们可以在这里执行程序化修正。

将一个 Orientation Warping 节点拖入图中，连接它并指定所需的骨骼：

[![img](assets/61986ffe-5fc6-4423-b801-602a025129a7.png)](https://dev.epicgames.com/community/api/learning/image/61986ffe-5fc6-4423-b801-602a025129a7?resizing_type=fit)

在 Orientation Warping 节点之后放置一个 Reset Root Transform 节点。这个节点将重置根骨骼上的变换，以防止朝向变形影响下一帧的姿态搜索计算。

[![img](assets/948158e0-94ff-4139-922a-e6d9dcfe0bfc.png)](https://dev.epicgames.com/community/api/learning/image/948158e0-94ff-4139-922a-e6d9dcfe0bfc?resizing_type=fit)

回到动画图的顶层，在 Motion Matching 节点之后放入一个 Leg IK 节点。这将应用 IK 来将 FK 脚骨位置修正到由 Orientation Warping 节点设置的 IK 脚骨位置。

[![img](assets/4e1043ac-2767-421c-ae1f-a42f79ab3751.png)](https://dev.epicgames.com/community/api/learning/image/4e1043ac-2767-421c-ae1f-a42f79ab3751?resizing_type=fit)

现在你可以再次 PIE 来查看 Orientation Warping 的效果。

你现在应该能看到 Animation Warping 正在为偏离前/后、左/右轴的移动生成中间姿态。



如果你需要调试 Orientation Warping，可以启用 `a.AnimNode.OrientationWarping.Debug` cvar。

#### Stride Warping（步幅变形）

尽管我们限制了角色可以移动的速度，但在某些时候——当角色起步、停步或在走路和跑步之间加速/减速时——仍然可能出现脚步滑动。为了解决这个问题，我们可以使用 Stride Warping 节点。

返回动画蓝图中 Motion Matching 节点内的子图。

在 Reset Root Transform 节点之后，放入一个 Stride Warping 节点并按如下方式设置：

[![img](assets/cd96b757-e070-48b1-8abc-de154323b1e8.png)](https://dev.epicgames.com/community/api/learning/image/cd96b757-e070-48b1-8abc-de154323b1e8?resizing_type=fit)

我们需要一个变量来驱动 Locomotion Speed 输入。添加一个新的 float 变量，命名为 Displacement Speed，以及一个 vector 变量，命名为 World Location。将 Displacement Speed 连接到 Stride Warping 节点上的 Locomotion Speed 输入引脚。

现在我们需要计算这些值。在你的 Update Variables 函数中，按如下方式设置 Displacement Speed 和 World Location：

[![img](assets/50d65184-05c8-4425-ad7f-c3790d3c4ff7.png)](https://dev.epicgames.com/community/api/learning/image/50d65184-05c8-4425-ad7f-c3790d3c4ff7?resizing_type=fit)

回去 PIE。你现在应该能看到腿部的步幅更接近胶囊体的速度了。



如果你需要调试 Stride Warping，`a.AnimNode.StrideWarping.Debug` cvar 会很有用。

### 优化动画选择

如果你 PIE 运行，应该能看到 Motion Matching 设置看起来相当不错。我们有一个可以从待机平滑过渡到移动、从走路切换到跑步、改变方向，然后回到待机的角色。

然而，如果你仔细观察在走路和跑步动画之间切换时，或者偶尔在角色转身后回到移动时，你会看到姿态上的一些不连续性。这个 [Rewind Debugger](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-rewind-debugger-in-unreal-engine) 捕获显示了 Jog_Left_Pivot 动画在 Walk_Left_Pivot 动画播放后被错误地选中的情况。

#### 通过 Rewind Debugger 识别问题

[Rewind Debugger](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-rewind-debugger-in-unreal-engine) 拥有用于调试 Motion Matching 设置问题的丰富工具。在本案例中，我们想了解为什么在转身完成后 Jog_Left_Pivot 动画被选中。

我们可以用录制好的缓冲区做的第一件事是选择 Pose Search 行。Rewind Debugger 的细节面板将显示与主面板中所选帧的动画选择相关的 Pose Search 数据。我们想了解为什么 Jog_Left_Pivot 被选中，因此我们可以拖动到该序列活跃的第一帧。

[![img](assets/288bb651-b9c0-42cd-8dd5-9222efd9c7d2.png)](https://dev.epicgames.com/community/api/learning/image/288bb651-b9c0-42cd-8dd5-9222efd9c7d2?resizing_type=fit)

当你第一次选择 Pose Search 行时，你会在细节面板中看到大量数据。在本案例中，我们关注的是动画而非帧选择，因此你可以选择 'Only Best Asset Pose' 来减少我们正在查看的数据集。

转身后，我们的角色加速回到了跑步速度。但与其按预期过渡到 Jog_Right 动画，我们过渡到了 Jog_Left_Pivot 动画中转身发生之后的部分（第 27 帧，我们可以在调试器中看到）。这是因为此选择的成本是 0.03，而 Jog_Right 动画的成本是 0.104。

有各种方法可以解决这个问题。一个例子可能是降低 Pose Search Database 中的 Looping Cost Bias，以减少选择 Jog_Right 动画的成本，因为它是循环动画，而转身动画不是。或者我们可以修改 Schema 添加额外的通道来优化选择。

然而，在本案例中，我们知道我们绝不希望在转身发生后过渡到转身动画的一部分。转身动画永远不应该被选择来提供连续的走路或跑步动画；我们有稳态的走路和跑步动画来完成这个任务。为此，我们可以通过 'Pose Search: Block Transition' 状态通知来标记转身动画中我们不想过渡进入的区域。

打开 Jog_Left_Pivot 动画并添加一个新的 Notify 轨道。然后在剪辑中添加 'Pose Search: Block Transition' 通知，分别在转身发生之前和之后。Motion Matching 将只允许在这些通知不活跃的帧上过渡到此剪辑。

[![img](assets/276f4e46-1c85-4a67-beab-0983335a22e6.png)](https://dev.epicgames.com/community/api/learning/image/276f4e46-1c85-4a67-beab-0983335a22e6?resizing_type=fit)



为了帮助放置通知，你可以添加一个 MotionExtractorModifier 动画修改器来分析动画的速度。这将显示动画中转身动作发生的位置。然后你可以屏蔽该点之前和之后的区域。你的动画覆盖水平越低，可能需要越激进地使用此方法。

现在对 Walk 和 Jog 数据库中其他每个转身动画重复此过程。

### 总结

至此，你应该拥有一个功能齐全的 Motion Matching 设置。我们已经了解了如何设置角色蓝图、Motion Matching 资产如何设置、如何在动画蓝图中使用相关节点，以及 Animation Warping。我们还简要介绍了一些可用的调试工具。这些理解应该能为你提供在自己的项目中开始构建 Motion Matching 设置所需的知识。
