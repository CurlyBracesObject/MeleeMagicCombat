MeleeMagicCombat
基于虚幻引擎5开发的第三人称动作RPG战斗系统Demo，展示完整的角色战斗、AI行为、技能系统等核心玩法实现。
视频demo link:https://www.bilibili.com/video/BV17nuTzsEFW/?vd_source=c096d37a6e7624ca39a2afef5c3f64d2

技术特性
角色控制系统: 第三人称视角控制，支持移动、跳跃、攻击等基础操作
战斗系统: 近战与魔法技能双重战斗模式
AI敌人系统: 基于行为树的智能敌人，支持巡逻、追击、攻击行为
技能系统: 数据驱动的技能配置，包含冷却时间和特效管理
物理交互: 碰撞检测、轨迹计算、延迟特效等
核心系统实现

角色移动与输入系统

输入映射配置:
使用Enhanced Input System处理输入
Input Action: IA_Move (Vector2D)
Input Action: IA_Look (Vector2D)
Input Action: IA_Jump (Boolean)
Input Action: IA_Attack (Boolean)
Input Action: IA_Skill (Boolean)
角色移动逻辑:
Event Tick执行:

Get Control Rotation → Break Rotator → Make Rotator(0, Yaw, 0)
Get Input Axis Vector → Rotate Vector (Forward/Right)
Add Movement Input (World Direction, Scale Value)
Character Movement Component自动处理物理移动

摄像机控制:
鼠标输入处理:

Input Axis Look → Split Struct Pin (X/Y)
Add Controller Yaw Input (X轴)
Add Controller Pitch Input (Y轴, 需Clamp限制角度)
Spring Arm Component自动处理摄像机跟随
动画系统

动画蓝图架构:
State Machine主要状态:
Idle/Walk/Run (Blend Space 1D: Speed)
Jump (Start → Loop → End)
Attack (Animation Montage)
Skill Cast (Animation Montage with Root Motion)
移动混合空间:
变量更新 (Event Blueprint Update Animation):

Try Get Pawn Owner → Cast to Character
Get Velocity → Vector Length → Speed
Is Falling → 跳跃状态布尔值
将变量传递给State Machine条件

攻击动画处理:
攻击输入触发:

Input Action IA_Attack → Branch (CanAttack)
Montage Play (AttackMontage)
设置CanAttack = False
Montage结束事件 → 重置CanAttack = True
AI敌人系统

AI Controller配置:
BeginPlay执行:

Run Behavior Tree (EnemyBT)
设置Blackboard Asset
Set Blackboard Value: PatrolLocation (初始位置)

视野检测系统:
Pawn Sensing Component配置:
Sight Radius: 1000
Sight Angle: 90
See Pawns: True
On See Pawn事件:

检查Sensed Actor是否为玩家
Set Blackboard Value: TargetActor
Set Blackboard Value: AIState = "Chase"

行为树结构:
Root → Selector
├ Sequence (有目标时)
│   ├ Blackboard Decorator (TargetActor存在)
│   ├ Move To (Target: TargetActor)
│   └ Wait (攻击间隔)
└ Sequence (巡逻状态)
├ Move To (Target: PatrolLocation)
├ Wait (巡逻等待)
└ Set Blackboard Value (随机巡逻点)

技能系统

技能数据结构:
Struct: SkillData
SkillID (Int32)
CooldownTime (Float)
ManaCost (Float)
EffectClass (Class Reference)
AnimMontage (Animation Montage)
SkillIcon (Texture 2D)
技能释放逻辑:
技能输入处理:

Input Action → Get Skill Data from DataTable
Branch: 检查冷却时间和蓝量
如果可用 → Play Animation Montage
启动Timer: 冷却时间倒计时
Spawn Actor: 技能特效
扣除蓝量值

冷却时间管理:
技能冷却系统:

使用Timer Handle数组管理多个技能冷却
Set Timer by Function Name: 每个技能独立计时
UI更新: Get Timer Remaining → 更新进度条
Timer完成 → 技能可用标志重置
战斗与碰撞系统

近战攻击检测:
攻击判定:

Animation Notify → 触发碰撞检测
Sphere Trace for Objects (半径: 100, 距离: 200)
ForEach Hit Result → 过滤敌人Actor
Apply Damage (基础伤害 + 随机值)
播放受击特效和音效

魔法技能投射物:
投射物组件配置:

Projectile Movement Component
Initial Speed: 1000
Max Speed: 1000
Gravity Scale: 0
Sphere Collision Component
Collision Response: Block Dynamic
On Hit Event → Apply Area Damage
Timer Destroy: 生命周期3秒

伤害计算系统:
伤害处理:

Apply Point Damage → 接收Damage Event
计算最终伤害: BaseDamage * DamageMultiplier
更新血量: CurrentHP = Clamp(CurrentHP - Damage, 0, MaxHP)
广播血量变化事件给UI
血量为0 → 触发死亡逻辑

开发环境
引擎版本: Unreal Engine 5.1+
开发语言: Blueprint Visual Scripting
目标平台: Windows
项目结构
MeleeMagicCombat/
├ Blueprints/
│   ├ Characters/        角色相关蓝图
│   ├ AI/               AI控制器和行为树
│   ├ Skills/           技能系统
│   └ UI/               用户界面
├ Content/
│   ├ Maps/             关卡文件
│   ├ Animations/       动画资源
│   └ Effects/          特效资源
└ DataTables/           技能和配置数据




MeleeMagicCombat
Third-person Action RPG combat system demo developed with Unreal Engine 5, showcasing complete character combat, AI behavior, skill system and other core gameplay implementations.
Technical Features
Character Control System: Third-person perspective control with movement, jumping, attack and other basic operations
Combat System: Dual combat mode combining melee and magic skills
AI Enemy System: Behavior tree-based intelligent enemies supporting patrol, chase and attack behaviors
Skill System: Data-driven skill configuration including cooldown management and effect handling
Physics Interaction: Collision detection, trajectory calculation, delayed effects and more
Core System Implementation

Character Movement and Input System

Input Mapping Configuration:
Using Enhanced Input System for input handling
Input Action: IA_Move (Vector2D)
Input Action: IA_Look (Vector2D)
Input Action: IA_Jump (Boolean)
Input Action: IA_Attack (Boolean)
Input Action: IA_Skill (Boolean)
Character Movement Logic:
Event Tick execution:

Get Control Rotation → Break Rotator → Make Rotator(0, Yaw, 0)
Get Input Axis Vector → Rotate Vector (Forward/Right)
Add Movement Input (World Direction, Scale Value)
Character Movement Component automatically handles physics movement

Camera Control:
Mouse input processing:

Input Axis Look → Split Struct Pin (X/Y)
Add Controller Yaw Input (X axis)
Add Controller Pitch Input (Y axis, requires Clamp for angle limitation)
Spring Arm Component automatically handles camera following
Animation System

Animation Blueprint Architecture:
State Machine main states:
Idle/Walk/Run (Blend Space 1D: Speed)
Jump (Start → Loop → End)
Attack (Animation Montage)
Skill Cast (Animation Montage with Root Motion)
Movement Blend Space:
Variable updates (Event Blueprint Update Animation):

Try Get Pawn Owner → Cast to Character
Get Velocity → Vector Length → Speed
Is Falling → Jump state boolean
Pass variables to State Machine conditions

Attack Animation Handling:
Attack input trigger:

Input Action IA_Attack → Branch (CanAttack)
Montage Play (AttackMontage)
Set CanAttack = False
Montage end event → Reset CanAttack = True
AI Enemy System

AI Controller Configuration:
BeginPlay execution:

Run Behavior Tree (EnemyBT)
Set Blackboard Asset
Set Blackboard Value: PatrolLocation (initial position)

Sight Detection System:
Pawn Sensing Component configuration:
Sight Radius: 1000
Sight Angle: 90
See Pawns: True
On See Pawn event:

Check if Sensed Actor is player
Set Blackboard Value: TargetActor
Set Blackboard Value: AIState = "Chase"

Behavior Tree Structure:
Root → Selector
├ Sequence (when target exists)
│   ├ Blackboard Decorator (TargetActor exists)
│   ├ Move To (Target: TargetActor)
│   └ Wait (attack interval)
└ Sequence (patrol state)
├ Move To (Target: PatrolLocation)
├ Wait (patrol wait)
└ Set Blackboard Value (random patrol point)

Skill System

Skill Data Structure:
Struct: SkillData
SkillID (Int32)
CooldownTime (Float)
ManaCost (Float)
EffectClass (Class Reference)
AnimMontage (Animation Montage)
SkillIcon (Texture 2D)
Skill Casting Logic:
Skill input processing:

Input Action → Get Skill Data from DataTable
Branch: Check cooldown time and mana
If available → Play Animation Montage
Start Timer: Cooldown countdown
Spawn Actor: Skill effects
Deduct mana value

Cooldown Management:
Skill cooldown system:

Use Timer Handle array to manage multiple skill cooldowns
Set Timer by Function Name: Independent timing for each skill
UI update: Get Timer Remaining → Update progress bar
Timer completion → Reset skill available flag
Combat and Collision System

Melee Attack Detection:
Attack determination:

Animation Notify → Trigger collision detection
Sphere Trace for Objects (Radius: 100, Distance: 200)
ForEach Hit Result → Filter enemy Actors
Apply Damage (Base damage + Random value)
Play hit effects and audio

Magic Skill Projectiles:
Projectile component configuration:

Projectile Movement Component
Initial Speed: 1000
Max Speed: 1000
Gravity Scale: 0
Sphere Collision Component
Collision Response: Block Dynamic
On Hit Event → Apply Area Damage
Timer Destroy: 3-second lifespan

Damage Calculation System:
Damage processing:

Apply Point Damage → Receive Damage Event
Calculate final damage: BaseDamage * DamageMultiplier
Update health: CurrentHP = Clamp(CurrentHP - Damage, 0, MaxHP)
Broadcast health change event to UI
Health reaches 0 → Trigger death logic

Development Environment
Engine Version: Unreal Engine 5.1+
Development Language: Blueprint Visual Scripting
Target Platform: Windows
Project Structure
MeleeMagicCombat/
├ Blueprints/
│   ├ Characters/        Character-related blueprints
│   ├ AI/               AI controllers and behavior trees
│   ├ Skills/           Skill system
│   └ UI/               User interface
├ Content/
│   ├ Maps/             Level files
│   ├ Animations/       Animation assets
│   └ Effects/          Effect assets
└ DataTables/           Skill and configuration data
