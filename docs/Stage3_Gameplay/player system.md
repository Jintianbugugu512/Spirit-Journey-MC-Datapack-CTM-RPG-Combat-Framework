# 玩家系统文档 (Player System)

---

## 📋 系统概述

**玩家系统**是一个支持多人游戏的玩家管理框架，通过**玩家ID标签**和**宏函数参数**实现多玩家数据隔离和统一管理。

### 核心特性

- 👥 **多人支持** - 支持多个玩家同时游戏，数据完全隔离(重要备注：开发阶段作者没有联机环境，整套多人逻辑**仅完成框架编写，没有实际联机测试**)
- 🏷️ **ID标签系统** - 每个玩家通过唯一ID标签识别（player_1, player_2, player_3...）
- 📦 **独立存储** - 每个玩家拥有独立的storage存储空间（damage:player_1, damage:player_2...）
- 🔄 **通用处理框架** - 使用宏函数实现一套代码处理所有玩家
- ✅ **易于扩展** - 只需添加一行代码即可注册新玩家

---

## 🎮 当前配置

### 支持玩家数量

系统**目前支持3名玩家**同时游戏：
- `player_1` - 1号玩家
- `player_2` - 2号玩家
- `player_3` - 3号玩家

### 如何注册更多玩家

在 `data/z_loop/functions/player_manager.mcfunction` 中添加新玩家：

```mcfunction
# 当前配置（3人）
execute as @a[tag=player_1] run function player:player_common {"id":"1"}
execute as @a[tag=player_2] run function player:player_common {"id":"2"}
execute as @a[tag=player_3] run function player:player_common {"id":"3"}

# 添加第4个玩家
execute as @a[tag=player_4] run function player:player_common {"id":"4"}

# 添加第5个玩家
execute as @a[tag=player_5] run function player:player_common {"id":"5"}

# 理论上可以无限扩展...
```

**就这么简单！** 添加一行代码，系统就会自动为新玩家：

- ✅ 创建独立的存储空间（damage:player_4）
- ✅ 隔离技能CD、血量、MP等数据
- ✅ 支持所有战斗和技能系统

---

## 🏗️ 系统架构

### 核心文件结构

```
player/functions/
├── player_common.mcfunction    # 通用玩家处理函数（宏函数）
├── attribute/
│   ├── set.mcfunction          # 设置玩家属性
│   └── modify.mcfunction       # 修改玩家属性
├── click_detect.mcfunction     # 点击检测（左键/右键/潜行）
└── shift.mcfunction            # 潜行技能触发

z_loop/functions/
└── player_manager.mcfunction   # 玩家主循环管理器（注册所有玩家）
```

---

## 🔄 工作流程

### 主循环执行流程

```
z_loop:player_manager (每tick执行)
    ↓
对每个玩家执行 player:player_common {"id":"X"}
    ├─ 反作弊检测 (anti_cheat:ban_tp)
    ├─ 点击检测 (player:click_detect)
    ├─ 魂盘系统 (soul_disc:soul_manager)
    ├─ 属性修改 (player:attribute/modify)
    ├─ 技能释放 (player:shift)
    ├─ 技能检测 (skill:check_all_skills)
    ├─ 武器普通攻击
    │   ├─ 单手剑 (combat:attack_generic/sword)
    │   ├─ 刺剑 (combat:attack_generic/rapier)
    │   ├─ 弩 (combat:attack_generic/crossbow)
    │   └─ 法杖 (combat:attack_generic/stick)
    ├─ 血量显示 (combat:healthbar/health_bar_get)
    └─ MP条显示 (combat:mp_bar/mp_change_detector)
```

---

## 🏷️ ID标签系统

### 玩家标签

每个玩家在加入游戏时需要被分配一个唯一的ID标签：

```mcfunction
# 给玩家添加ID标签
/tag <玩家名> add player_1
/tag <玩家名> add player_2
/tag <玩家名> add player_3
```

**注意：** 每个玩家只能有一个ID标签，不要重复添加！

---

## 📦 数据存储系统

### Storage命名规则

每个玩家拥有独立的storage存储空间：

- `damage:player_1` - 1号玩家的数据
- `damage:player_2` - 2号玩家的数据
- `damage:player_3` - 3号玩家的数据
- `damage:player_N` - N号玩家的数据

### 存储内容

玩家的storage存储以下数据：
- 攻击力、魔法强度
- 物理护甲、魔法抗性
- 最大生命值、当前生命值
- MP相关数据
- 技能CD和状态
- 其他战斗数据

### 数据隔离原理

通过宏函数参数 `{"id":"X"}` 动态拼接storage路径：

```mcfunction
# player_common.mcfunction 中
$execute at @s run function soul_disc:soul_manager with storage damage:player_$(id)

# 实际执行时：
# player_1 → with storage damage:player_1
# player_2 → with storage damage:player_2
# player_3 → with storage damage:player_3
```

---

## 🛠️ 宏函数机制

### 通用处理函数

`player:player_common` 是核心宏函数，接受一个参数 `id`：

```mcfunction
# 调用方式
function player:player_common {"id":"1"}
function player:player_common {"id":"2"}
function player:player_common {"id":"3"}
```

### 宏参数展开

在函数内部，`$(id)` 会被替换为实际的ID值：

```mcfunction
# 代码
$execute at @s run function player:shift with storage damage:player_$(id)

# 展开为
# id=1 → with storage damage:player_1
# id=2 → with storage damage:player_2
# id=3 → with storage damage:player_3
```

---

## 🎯 功能模块

### 1. 反作弊检测

```mcfunction
$execute if entity @s[gamemode=adventure,scores={interval_can_tp=0},tag=!Mana] at @s run function anti_cheat:ban_tp {"player":"player_$(id)"}
```

- 检测玩家非法传送

---

### 2. 点击检测

```mcfunction
$execute at @s run function player:click_detect {"player":"detect_$(id)"}
```

- 检测左键点击
- 检测右键点击
- 检测潜行动作

---

### 3. 技能系统

```mcfunction
$execute if score @s put_shift matches 1.. at @s run function player:shift with storage damage:player_$(id)
$function skill:check_all_skills {"id":"$(id)"}
```

- 技能释放检测

---

### 4. 战斗系统

```mcfunction
# 单手剑普通攻击
$execute if entity @s[scores={use_IronSword=1..}] at @s run function combat:attack_generic/sword with storage damage:player_$(id)

# 刺剑普通攻击
$execute if entity @s[scores={use_NetheriteSword=1..}] at @s run function combat:attack_generic/rapier with storage damage:player_$(id)

# 弩攻击
$execute if entity @s[scores={use_Crossbow=1..}] at @s run function combat:attack_generic/crossbow with storage damage:player_$(id)
```

- 武器普通攻击

---

### 5. 血量和MP管理

```mcfunction
# 血量显示
execute at @s run function combat:healthbar/health_bar_get

# MP条显示
execute at @s run function combat:mp_bar/mp_change_detector
```

- 实时血量显示
- MP变化检测
- 血条渲染

---

## 📝 添加新玩家步骤

### 完整流程

#### 步骤1: 在 player_manager 注册玩家

**文件**: `data/z_loop/functions/player_manager.mcfunction`

```mcfunction
# 添加第4个玩家
execute as @a[tag=player_4] run function player:player_common {"id":"4"}
```

#### 步骤2: 给玩家添加标签

```mcfunction
/tag <玩家名> add player_4
```

---

## 🔧 扩展能力

### 理论玩家数量上限

- **技术限制**: 无硬性上限，取决于服务器性能
- **推荐配置**: 1-10人（保证流畅体验）
- **当前配置**: 3人

### 性能考虑

每增加一个玩家，系统会：
- 每tick执行一次 `player_common`（约60-80条命令）
- 创建独立的storage存储空间
- 运行独立的技能和战斗检测

**建议**: 根据服务器性能和游戏设计需求合理配置玩家数量。

---

## 🎮 实际应用

### 多人合作战斗

```mcfunction
# 3个玩家同时攻击boss
# 每个玩家的伤害、技能CD、血量完全独立
execute as @a[tag=player_1] ...  # 玩家1的攻击
execute as @a[tag=player_2] ...  # 玩家2的攻击
execute as @a[tag=player_3] ...  # 玩家3的攻击
```

### 独立技能冷却

```mcfunction
# 玩家1使用技能，只影响player_1的CD
# 玩家2的CD不受影响
execute as @a[tag=player_1] run function skill:use_skill {"id":"1"}
execute as @a[tag=player_2] run function skill:use_skill {"id":"2"}
```

### 独立伤害计算

```mcfunction
# 每个玩家的攻击力独立计算
# 玩家1: 10点攻击力
# 玩家2: 15点攻击力
# 玩家3: 8点攻击力
function combat:deal/damage_deal with storage damage:player_1
function combat:deal/damage_deal with storage damage:player_2
function combat:deal/damage_deal with storage damage:player_3
```

