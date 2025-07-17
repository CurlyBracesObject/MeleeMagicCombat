# MeleeMagicCombat

基于虚幻引擎5开发的第三人称动作RPG战斗系统Demo，展示完整的角色战斗、AI行为、技能系统等核心玩法实现。

**视频演示:** [B站Demo视频](https://www.bilibili.com/video/BV17nuTzsEFW/?vd_source=c096d37a6e7624ca39a2afef5c3f64d2)

## 技术特性

- **角色控制系统:** 第三人称视角控制，支持移动、跳跃、攻击等基础操作
- **战斗系统:** 近战与魔法技能双重战斗模式  
- **AI敌人系统:** 基于行为树的智能敌人，支持巡逻、追击、攻击行为
- **技能系统:** 数据驱动的技能配置，包含冷却时间和特效管理
- **物理交互:** 碰撞检测、轨迹计算、延迟特效等

## 核心系统实现

### 1. 角色移动与输入系统

**Enhanced Input System配置:**
```cpp
// 输入动作定义
Input Action: IA_Move (Vector2D)
Input Action: IA_Look (Vector2D) 
Input Action: IA_Jump (Boolean)
Input Action: IA_Attack (Boolean)
Input Action: IA_Skill (Boolean)
```

**角色移动蓝图逻辑:**
```cpp
// Event Tick执行流程
Get Control Rotation → Break Rotator → Make Rotator(0, Yaw, 0)
Get Input Axis Vector → Rotate Vector (Forward/Right)
Add Movement Input (World Direction, Scale Value)
// Character Movement Component自动处理物理移动
```

**摄像机控制实现:**
```cpp
// 鼠标输入处理
Input Axis Look → Split Struct Pin (X/Y)
Add Controller Yaw Input (X轴)
Add Controller Pitch Input (Y轴, Clamp限制角度)
// Spring Arm Component自动处理摄像机跟随
```

### 2. 动画系统架构

**State Machine状态设计:**
```cpp
// 主要动画状态
- Idle/Walk/Run (Blend Space 1D: Speed)
- Jump (Start → Loop → End)
- Attack (Animation Montage)
- Skill Cast (Animation Montage with Root Motion)
```

**动画变量更新逻辑:**
```cpp
// Event Blueprint Update Animation
Try Get Pawn Owner → Cast to Character
Get Velocity → Vector Length → Speed
Is Falling → 跳跃状态布尔值
// 将变量传递给State Machine条件判断
```

**攻击动画控制:**
```cpp
// 攻击输入触发流程
Input Action IA_Attack → Branch (CanAttack)
Montage Play (AttackMontage)
Set CanAttack = False
// Montage结束事件 → 重置CanAttack = True
```

### 3. AI敌人行为系统

**AI Controller初始化:**
```cpp
// BeginPlay执行
Run Behavior Tree (EnemyBT)
Set Blackboard Asset
Set Blackboard Value: PatrolLocation (初始位置)
```

**Pawn Sensing Component视野检测:**
```cpp
// 组件配置
Sight Radius: 1000
Sight Angle: 90
See Pawns: True

// On See Pawn事件处理
检查Sensed Actor是否为玩家
Set Blackboard Value: TargetActor
Set Blackboard Value: AIState = "Chase"
```

**行为树架构设计:**
```
Root (BlackboardData_Enemy)
└─ Selector
   ├─ Sequence (死亡状态)
   │   ├─ Blackboard Based Condition_IsDeath
   │   └─ Wait (50.0s)
   ├─ Sequence (发现玩家状态)
   │   ├─ Blackboard Based Condition_IsSeePawn
   │   ├─ BTTask_AIAttack
   │   └─ Rotate to face BB entry
   └─ Sequence (巡逻状态)
       ├─ BTTask_SelectLocation
       └─ Move To (MoveLocation)
```

### 4. 数据驱动技能系统

**技能数据结构定义:**
```cpp
// Struct: SkillData
- SkillID (Int32)
- CooldownTime (Float)
- ManaCost (Float) 
- EffectClass (Class Reference)
- AnimMontage (Animation Montage)
- SkillIcon (Texture 2D)
```

**技能释放蓝图逻辑:**
```cpp
// 技能输入处理流程
Input Action → Get Skill Data from DataTable
Branch: 检查冷却时间和蓝量
如果可用 → Play Animation Montage
启动Timer: 冷却时间倒计时
Spawn Actor: 技能特效
扣除蓝量值
```

**冷却管理系统:**
```cpp
// Timer Handle数组管理
使用Timer Handle数组管理多个技能冷却
Set Timer by Function Name: 每个技能独立计时
UI更新: Get Timer Remaining → 更新进度条
Timer完成 → 技能可用标志重置
```

### 5. 战斗碰撞检测

**近战攻击判定:**
```cpp
// Animation Notify触发碰撞检测
Animation Notify → 触发碰撞检测
Sphere Trace for Objects (半径: 100, 距离: 200)
ForEach Hit Result → 过滤敌人Actor
Apply Damage (基础伤害 + 随机值)
播放受击特效和音效
```

**魔法投射物系统:**
```cpp
// Projectile Movement Component配置
- Initial Speed: 1000
- Max Speed: 1000
- Gravity Scale: 0

// Sphere Collision Component
- Collision Response: Block Dynamic
- On Hit Event → Apply Area Damage
- Timer Destroy: 生命周期3秒
```

**伤害计算与处理:**
```cpp
// 伤害系统流程
Apply Point Damage → 接收Damage Event
计算最终伤害: BaseDamage * DamageMultiplier
更新血量: CurrentHP = Clamp(CurrentHP - Damage, 0, MaxHP)
广播血量变化事件给UI
血量为0 → 触发死亡逻辑
```

## 开发环境

- **引擎版本:** Unreal Engine 5
- **开发语言:** Blueprint Visual Scripting
- **目标平台:** Windows

## 项目亮点

✅ **完整第三人称控制架构** - Enhanced Input System + Character Movement Component物理同步  
✅ **智能AI行为树** - Pawn Sensing + Blackboard状态管理 + 三状态行为切换  
✅ **数据驱动技能配置** - DataTable + Timer Handle数组 + 实时UI更新  
✅ **动画事件同步** - Animation Montage + Animation Notify精确触发  
✅ **物理碰撞系统** - Sphere Trace + Projectile Movement + 特效同步  

---

# MeleeMagicCombat

Third-person Action RPG combat system demo developed with Unreal Engine 5, showcasing complete character combat, AI behavior, skill system and other core gameplay implementations.

**Video Demo:** [Bilibili Demo Video](https://www.bilibili.com/video/BV17nuTzsEFW/?vd_source=c096d37a6e7624ca39a2afef5c3f64d2)

## Technical Features

- **Character Control System:** Third-person perspective control with movement, jumping, attack and other basic operations
- **Combat System:** Dual combat mode combining melee and magic skills
- **AI Enemy System:** Behavior tree-based intelligent enemies supporting patrol, chase and attack behaviors
- **Skill System:** Data-driven skill configuration including cooldown management and effect handling
- **Physics Interaction:** Collision detection, trajectory calculation, delayed effects and more

## Core System Implementation

### 1. Character Movement and Input System

**Enhanced Input System Configuration:**
```cpp
// Input Action Definitions
Input Action: IA_Move (Vector2D)
Input Action: IA_Look (Vector2D) 
Input Action: IA_Jump (Boolean)
Input Action: IA_Attack (Boolean)
Input Action: IA_Skill (Boolean)
```

**Character Movement Blueprint Logic:**
```cpp
// Event Tick Execution Flow
Get Control Rotation → Break Rotator → Make Rotator(0, Yaw, 0)
Get Input Axis Vector → Rotate Vector (Forward/Right)
Add Movement Input (World Direction, Scale Value)
// Character Movement Component automatically handles physics movement
```

**Camera Control Implementation:**
```cpp
// Mouse Input Processing
Input Axis Look → Split Struct Pin (X/Y)
Add Controller Yaw Input (X axis)
Add Controller Pitch Input (Y axis, Clamp angle limitation)
// Spring Arm Component automatically handles camera following
```

### 2. Animation System Architecture

**State Machine Design:**
```cpp
// Main Animation States
- Idle/Walk/Run (Blend Space 1D: Speed)
- Jump (Start → Loop → End)
- Attack (Animation Montage)
- Skill Cast (Animation Montage with Root Motion)
```

**Animation Variable Update Logic:**
```cpp
// Event Blueprint Update Animation
Try Get Pawn Owner → Cast to Character
Get Velocity → Vector Length → Speed
Is Falling → Jump state boolean
// Pass variables to State Machine condition checks
```

**Attack Animation Control:**
```cpp
// Attack Input Trigger Flow
Input Action IA_Attack → Branch (CanAttack)
Montage Play (AttackMontage)
Set CanAttack = False
// Montage end event → Reset CanAttack = True
```

### 3. AI Enemy Behavior System

**AI Controller Initialization:**
```cpp
// BeginPlay Execution
Run Behavior Tree (EnemyBT)
Set Blackboard Asset
Set Blackboard Value: PatrolLocation (initial position)
```

**Pawn Sensing Component Detection:**
```cpp
// Component Configuration
Sight Radius: 1000
Sight Angle: 90
See Pawns: True

// On See Pawn Event Handling
Check if Sensed Actor is player
Set Blackboard Value: TargetActor
Set Blackboard Value: AIState = "Chase"
```

**Behavior Tree Architecture:**
```
Root (BlackboardData_Enemy)
└─ Selector
   ├─ Sequence (Death State)
   │   ├─ Blackboard Based Condition_IsDeath
   │   └─ Wait (50.0s)
   ├─ Sequence (Player Detection State)
   │   ├─ Blackboard Based Condition_IsSeePawn
   │   ├─ BTTask_AIAttack
   │   └─ Rotate to face BB entry
   └─ Sequence (Patrol State)
       ├─ BTTask_SelectLocation
       └─ Move To (MoveLocation)
```

### 4. Data-Driven Skill System

**Skill Data Structure Definition:**
```cpp
// Struct: SkillData
- SkillID (Int32)
- CooldownTime (Float)
- ManaCost (Float) 
- EffectClass (Class Reference)
- AnimMontage (Animation Montage)
- SkillIcon (Texture 2D)
```

**Skill Casting Blueprint Logic:**
```cpp
// Skill Input Processing Flow
Input Action → Get Skill Data from DataTable
Branch: Check cooldown time and mana
If available → Play Animation Montage
Start Timer: Cooldown countdown
Spawn Actor: Skill effects
Deduct mana value
```

**Cooldown Management System:**
```cpp
// Timer Handle Array Management
Use Timer Handle array to manage multiple skill cooldowns
Set Timer by Function Name: Independent timing for each skill
UI update: Get Timer Remaining → Update progress bar
Timer completion → Reset skill available flag
```

### 5. Combat Collision Detection

**Melee Attack Detection:**
```cpp
// Animation Notify Triggered Collision Detection
Animation Notify → Trigger collision detection
Sphere Trace for Objects (Radius: 100, Distance: 200)
ForEach Hit Result → Filter enemy Actors
Apply Damage (Base damage + Random value)
Play hit effects and audio
```

**Magic Projectile System:**
```cpp
// Projectile Movement Component Configuration
- Initial Speed: 1000
- Max Speed: 1000
- Gravity Scale: 0

// Sphere Collision Component
- Collision Response: Block Dynamic
- On Hit Event → Apply Area Damage
- Timer Destroy: 3-second lifespan
```

**Damage Calculation and Processing:**
```cpp
// Damage System Flow
Apply Point Damage → Receive Damage Event
Calculate final damage: BaseDamage * DamageMultiplier
Update health: CurrentHP = Clamp(CurrentHP - Damage, 0, MaxHP)
Broadcast health change event to UI
Health reaches 0 → Trigger death logic
```

## Development Environment

- **Engine Version:** Unreal Engine 5
- **Development Language:** Blueprint Visual Scripting
- **Target Platform:** Windows

## Project Highlights

✅ **Complete Third-Person Control Architecture** - Enhanced Input System + Character Movement Component physics sync  
✅ **Intelligent AI Behavior Tree** - Pawn Sensing + Blackboard state management + three-state behavior switching  
✅ **Data-Driven Skill Configuration** - DataTable + Timer Handle array + real-time UI updates  
✅ **Animation Event Synchronization** - Animation Montage + Animation Notify precise triggering  
✅ **Physics Collision System** - Sphere Trace + Projectile Movement + effect synchronization
