# 战斗系统 (Combat System)

---

## 系统概述

**战斗系统（Combat System）** 是整个数据包的核心API层，提供统一的伤害计算、血条显示、Debuff管理等功能。所有技能和攻击都通过这个系统处理伤害。

---

## 核心接口

### 🎯 统一伤害接口

备注:其实可以定义出不同的伤害计算公式。比如说像《明日方舟》一样：

1. 物理伤害：攻击力减防御力
2. 魔法伤害：攻击力乘以魔抗百分比
3. 真实伤害:直接结算

这张CTM地图为了简化伤害计算,把魔法伤害的计算公式也改成了攻击力减防御力.但是也把魔法伤害设置成了更高.怪物的魔抗会普遍比物抗低,魔法伤害的倍率也会普遍偏低,用于对付高物防敌人.感觉三种伤害的计算公式应该也蛮多的了,你可以修改其中的计算公式,或者你自己来定义新的计算公式.如果你需要修改已有计算公式,直接修改就可以.如果你还想增加更多伤害计算类型,就新增函数.

#### damage_deal - 伤害处理（核心API）

**路径**: `combat:deal/damage_deal`  
**类型**: 宏函数  
**执行者**: 将要受到伤害的实体

**接口签名**:
```mcfunction
function combat:deal/damage_deal {
    rate: "整数",        # 伤害倍率（百分比）
    type: "类型",        # 伤害类型
    player: "玩家标签",  # 造成伤害的玩家
    interval: "类型"     # 伤害间隔类型
}
```

**参数说明**:

| 参数 | 类型 | 说明 | 示例值 |
|------|------|------|--------|
| `rate` | 字符串 | 伤害倍率（%） | `"100"`, `"150"`, `"400"` |
| `type` | 字符串 | 伤害类型 | `"physic"`, `"magic"`, `"true"` |
| `player` | 字符串 | 玩家标签 | `"player_1"`, `"player_2"`, `"player_3"` |
| `interval` | 字符串 | 伤害间隔类型 | `"cactus"`, `"generic"` |

**调用示例**:
```mcfunction
# 对敌人造成150%物理伤害
execute as @e[tag=enemy,distance=..5] run function combat:deal/damage_deal {
    rate:"150",
    type:"physic",
    player:"player_1",
    interval:"generic"
}

# 技能造成400%魔法伤害
execute as @e[tag=skill_target] run function combat:deal/damage_deal {
    rate:"400",
    type:"magic",
    player:"player_1",
    interval:"cactus"
}

# 真实伤害（无视防御）
execute as @e[tag=enemy,distance=..3] run function combat:deal/damage_deal {
    rate:"35",
    type:"true",
    player:"player_1",
    interval:"cactus"
}
```

**实现逻辑**:
```mcfunction
# 1. 检查攻击间隔（防止重复伤害）
execute store result score @s interval_stick run data get entity @s PortalCooldown
execute if score @s interval_stick matches 1.. run return 2

# 2. 存储伤害参数到 storage
data modify storage damage:$(player) rate set value $(rate)
data merge storage damage:$(player) {interval:"$(interval)"}

# 3. 调用对应的伤害类型函数
function combat:damage_type/$(type)_damage with storage damage:$(player)

# 4. 设置攻击间隔冷却
data modify entity @s PortalCooldown set value 2
```

---

## 伤害类型

### 1. physic_damage - 物理伤害

**路径**: `combat:damage_type/physic_damage`  
**类型**: 宏函数  
**执行者**: 受伤害实体

**计算公式**:
```
最终伤害 = (攻击力 × 倍率%) - 物理防御
保底伤害 = 1点
```

**实现逻辑**:
```mcfunction
# 1. 获取玩家攻击力（使用 generic.luck 属性）
execute store result score @s temp run attribute @a[limit=1,tag=$(player)] generic.luck get

# 2. 计算伤害倍率
scoreboard players set #rate temp $(rate)
scoreboard players operation @s temp *= #rate temp
scoreboard players set #hundred temp 100
scoreboard players operation @s temp /= #hundred temp

# 3. 减去物理防御
scoreboard players operation @s temp -= @s physical_armor

# 4. 转换为百分比（×100）
scoreboard players set #test temp 100
scoreboard players operation @s temp *= #test temp

# 5. 保底伤害判断
execute if score @s temp matches ..100 run damage @s 1 minecraft:cactus
execute if score @s temp matches ..100 run return 1

# 6. 应用计算后的伤害
execute store result storage damage:$(player) damage int 0.01 run scoreboard players get @s temp
function combat:damage_type/pm_damage with storage damage:$(player)
```

---

### 2. magic_damage - 魔法伤害

**路径**: `combat:damage_type/magic_damage`  
**类型**: 宏函数  
**执行者**: 受伤害实体

**计算公式**:
```
最终伤害 = (攻击力 × 倍率%) - 魔法抗性
保底伤害 = 1点
```

**实现逻辑**: 魔攻-魔抗，使用 `magic_resist` 记分板

---

### 3. true_damage - 真实伤害

**路径**: `combat:damage_type/true_damage`  
**类型**: 宏函数  
**执行者**: 受伤害实体

**计算公式**:

```
最终伤害 = 攻击力 × 倍率%
保底伤害 = 1点
无视任何防御
```

**实现逻辑**:
```mcfunction
# 1. 获取攻击力并计算倍率
execute store result score @s temp run attribute @a[limit=1,tag=$(player)] generic.luck get
scoreboard players set #rate temp $(rate)
scoreboard players operation @s temp *= #rate temp
scoreboard players set #hundred temp 100
scoreboard players operation @s temp /= #hundred temp

# 2. 保底伤害
execute if score @s temp matches ..1 run damage @s 1
execute if score @s temp matches ..1 run return 1

# 3. 直接应用伤害（无防御计算）
execute store result storage damage:$(player) damage int 1 run scoreboard players get @s temp
function combat:damage_type/pm_damage with storage damage:$(player)
```

---

### 4. pm_damage - 实际伤害应用

**路径**: `combat:damage_type/pm_damage`  
**类型**: 宏函数  
**执行者**: 受伤害实体

**功能**: 最终将计算好的伤害应用到实体

**实现逻辑**:
```mcfunction
# 减少记分板生命值
scoreboard players remove @s current_health $(damage)

# 防止负数
scoreboard players set @s[scores={current_health=..0}] current_health 0

# 触发伤害效果（0点伤害，只为触发受伤动画）
damage @s 0 minecraft:cactus
```





---

## 血条系统

### healthbar 模块

#### health_bar_get - BOSS血条显示

**路径**: `combat:healthbar/health_bar_get`  
**类型**: 普通函数  
**执行者**: 玩家

**功能**: 为玩家显示附近BOSS的血条

**实现逻辑**:
```mcfunction
# 1. 获取BOSS名称
data modify storage nbt:boss_name name set from entity @e[limit=1,sort=nearest,distance=..20,tag=show_healthbar] CustomName

# 2. 设置bossbar最大值和当前值
execute store result bossbar healthbar max run scoreboard players get @e[limit=1,sort=nearest,tag=show_healthbar] max_health
execute store result bossbar healthbar value run scoreboard players get @e[limit=1,sort=nearest,distance=..20,tag=show_healthbar] current_health

# 3. 显示/隐藏bossbar
execute at @s if entity @e[tag=show_healthbar,distance=..20] run bossbar set healthbar players @s
execute at @s unless entity @e[tag=show_healthbar,distance=..20] run bossbar remove healthbar

# 4. 设置样式
bossbar set healthbar style notched_6

# 5. 更新名称
function combat:healthbar/health_bar_name with storage nbt:boss_name
```

**特点**:
- 20格范围内显示BOSS血条
- 离开范围自动隐藏
- 使用 `notched_6` 样式（6段显示）

---

#### calculate_armor_type - 护甲类型计算

**路径**: `combat:healthbar/calculate_armor_type`  
**类型**: 普通函数  
**执行者**: 怪物实体

**功能**: 根据物理防御和魔抗计算护甲类型标识

**计算规则**:
```
差值 = 物理防御 - 魔法抗性

差值 >= 25  → armor_type = 1 (高物防) → ■
差值 6~24   → armor_type = 2 (均衡型) → ●
差值 <= 5   → armor_type = 3 (高魔抗) → ★
```

**实现逻辑**:
```mcfunction
# 计算差值
scoreboard players operation #defense_diff temp = @s physical_armor
scoreboard players operation #defense_diff temp -= @s magic_resist

# 分类
execute if score #defense_diff temp matches 25.. run scoreboard players set @s armor_type 1
execute if score #defense_diff temp matches 6..24 run scoreboard players set @s armor_type 2
execute if score #defense_diff temp matches ..5 run scoreboard players set @s armor_type 3
```

**用途**: 在敌人名字上显示护甲类型图标，提示玩家使用对应的攻击方式

---

#### bars20 - 血条图标

**路径**: `combat:healthbar/bars20/`

**功能**: 20段血条显示系统

**文件列表**:
- `red_1` ~ `red_3` - 低血量（红色）
- `yellow_4` ~ `yellow_11` - 中血量（黄色）
- `green_12` ~ `green_20` - 高血量（绿色）

**特点**:
- 颜色根据血量百分比变化
- 20段精细显示
- 在敌人名字上方显示

---

#### update_healthbar_display - 更新血条显示

**路径**: `combat:healthbar/update_healthbar_display`  
**类型**: 普通函数

**功能**: 根据当前血量更新敌人名字上的血条显示

---

## Debuff系统

### 1. burn - 燃烧

**路径**: `combat:debuff/burn/`

#### burn/tick - 燃烧持续伤害

**功能**: 每0.75秒造成3点真实伤害

**实现逻辑**:
```mcfunction
# 计算燃烧tick（使用fire记分板）
scoreboard players operation #burn_damage fire = @s fire
scoreboard players set #burn_damage temp 15
scoreboard players operation #burn_damage fire %= #burn_damage temp

# 每15tick（0.75秒）造成3点伤害
execute if score #burn_damage fire matches 0 run scoreboard players remove @s current_health 3
```

---

### 2. defense_reduce - 减防

**路径**: `combat:debuff/defense_reduce/`

#### apply - 应用减防

**功能**: 降低目标的防御力

#### remove - 移除减防

**功能**: 恢复目标的防御力

**用途**: 配合穿甲技能、减甲效果

---

### 3. stun - 眩晕

**路径**: `combat:debuff/stun/manager`

**功能**: 管理眩晕状态（禁止移动和攻击）

---

## 武器攻击

### attack_generic 模块

#### 1. sword - 剑

**路径**: `combat:attack_generic/sword`

**伤害**: 100% 物理伤害  
**回复**: 每次击中回复 4MP

---

#### 2. crossbow - 猎弩

**路径**: `combat:attack_generic/crossbow`

**伤害**: 150% 物理伤害  
**暴击**: 20% 概率触发 250% 伤害  
**回复**: 每次命中回复 8MP

---

#### 3. repeating_crossbow - 连弩

**路径**: `combat:attack_generic/repeating_crossbow_crit`

**伤害**: 30%~50% 物理伤害（随机）  
**范围**: 最多3名敌人  
**回复**: 每次命中回复 2MP

---

#### 4. rapier - 刺剑

**路径**: `combat:attack_generic/rapier`

**伤害**: 
- 主目标 70% 物理伤害
- 附近3个随机敌人 60% 物理伤害

**回复**: 每次击中回复 4MP

---

#### 5. stick - 散杖

**路径**: `combat:attack_generic/stick`

**伤害**: 80% 魔法伤害（AOE穿透）  
**回复**: 每命中1名敌人回复 2MP，BOSS回复 4MP

---

#### 6. focusing_staff - 凝杖

**路径**: `combat:attack_generic/focusing_staff_launch`

**伤害**: 50%~350% 魔法伤害（蓄力时间决定）  
**机制**: 蓄力型远程攻击  
**回复**: 命中敌人回复 4MP，BOSS回复 8MP

---

## 调用链追踪

### 完整伤害流程

```
技能/武器函数
    ↓
combat:deal/damage_deal (统一入口)
    ↓
├─ 检查攻击间隔 (PortalCooldown)
├─ 存储参数到 storage
└─ 调用伤害类型函数
    ↓
combat:damage_type/$(type)_damage
    ↓
├─ physic_damage (攻击力×倍率 - 物防)
├─ magic_damage (攻击力×倍率 - 魔抗)
└─ true_damage (攻击力×倍率)
    ↓
combat:damage_type/pm_damage
    ↓
├─ scoreboard players remove current_health
├─ 防止负数
└─ damage @s 0 (触发受伤动画)
```

---

### 血条更新流程

```
受到伤害
    ↓
current_health 减少
    ↓
每tick执行 (z_loop)
    ↓
combat:healthbar/update_healthbar_display
    ↓
├─ 计算血量百分比
├─ 选择对应颜色的血条
└─ 更新CustomName显示
```

---

### BOSS血条流程

```
玩家靠近BOSS
    ↓
combat:healthbar/health_bar_get
    ↓
├─ 获取BOSS名称
├─ 读取 max_health 和 current_health
├─ 设置 bossbar 最大值和当前值
└─ 显示给玩家
    ↓
玩家离开
    ↓
bossbar remove
```

---

## 实际应用案例

### 案例1: 技能造成伤害

```mcfunction
# 旋风斩技能 - 125%物理伤害

# 1. 标记范围内敌人
tag @e[tag=enemy,distance=..6] add whirlwind_target

# 2. 对每个敌人造成伤害
execute as @e[tag=whirlwind_target] run function combat:deal/damage_deal {
    rate:"125",
    type:"physic",
    player:"player_1",
    interval:"cactus"
}

# 3. 清理标签
tag @e[tag=whirlwind_target] remove whirlwind_target
```

---

### 案例2: 多倍率伤害

```mcfunction
# 重斩技能 - 根据蓄力时间造成不同伤害

# 1秒蓄力 - 100%伤害
execute if score @s charge_time matches 20..39 as @e[tag=target] run function combat:deal/damage_deal {
    rate:"100",
    type:"physic",
    player:"player_1",
    interval:"generic"
}

# 2秒蓄力 - 250%伤害
execute if score @s charge_time matches 40..59 as @e[tag=target] run function combat:deal/damage_deal {
    rate:"250",
    type:"physic",
    player:"player_1",
    interval:"generic"
}

# 3秒满蓄力 - 400%伤害
execute if score @s charge_time matches 60.. as @e[tag=target] run function combat:deal/damage_deal {
    rate:"400",
    type:"physic",
    player:"player_1",
    interval:"generic"
}
```

---

### 案例3: 召唤敌人并设置属性

```mcfunction
# 召唤高物防敌人

summon zombie ~ ~ ~ {
    CustomName:"[\"§2森林僵尸\"]",
    Health:300f,
    Tags:["enemy","summon"]
}

# 设置属性
scoreboard players set @e[tag=summon] max_health 300
scoreboard players set @e[tag=summon] current_health 300
scoreboard players set @e[tag=summon] physical_armor 30
scoreboard players set @e[tag=summon] magic_resist 5

# 计算护甲类型 (30-5=25，属于高物防)
scoreboard players operation @e[tag=summon] last_health = @e[tag=summon] max_health
execute as @e[tag=summon] run function combat:healthbar/calculate_armor_type
# armor_type = 1 (■)

# 清除临时标签
tag @e remove summon
```

---

## 记分板系统

### 生命值相关

| 记分板 | 用途 |
|--------|------|
| `max_health` | 最大生命值 |
| `current_health` | 当前生命值 |
| `last_health` | 上次生命值（用于计算伤害） |

### 属性相关

| 记分板 | 用途 |
|--------|------|
| `physical_armor` | 物理防御 |
| `magic_resist` | 魔法抗性 |
| `armor_type` | 护甲类型标识（1/2/3） |
| `hit_cooldown` | 受击冷却 |

### MP相关

| 记分板 | 用途 |
|--------|------|
| `MP` | 当前魔法值 |
| `max_MP` | 最大魔法值 |
| `MP_regen` | MP回复速度 |

### 其他

| 记分板 | 用途 |
|--------|------|
| `interval_stick` | 攻击间隔冷却 |
| `temp` | 临时伤害计算 |
| `fire` | 燃烧持续时间 |

---

## Storage系统

### damage:player_1/2/3

存储玩家属性和伤害参数：
```json
{
    player: "player_1",
    id: "1",
    attack: 100,
    rate: "150",
    damage: 120,
    interval: "cactus"
}
```

### nbt:boss_name

存储BOSS名称：
```json
{
    name: "[\"§c紫晶核心\"]"
}
```

---

## 设计理念

### 统一接口
- 所有伤害通过同一个入口（`damage_deal`）
- 便于统一管理和调试
- 易于添加新的伤害类型

### 分层设计
```
应用层 (技能/武器)
    ↓
接口层 (damage_deal)
    ↓
类型层 (physic/magic/true)
    ↓
执行层 (pm_damage)
```

---

## 接口总览表

| 接口 | 类型 | 主要参数 | 用途 |
|------|------|---------|------|
| `deal/damage_deal` | 宏函数 | rate, type, player, interval | 统一伤害入口 |
| `damage_type/physic_damage` | 宏函数 | player, rate | 物理伤害计算 |
| `damage_type/magic_damage` | 宏函数 | player, rate | 魔法伤害计算 |
| `damage_type/true_damage` | 宏函数 | player, rate | 真实伤害计算 |
| `damage_type/pm_damage` | 宏函数 | player, damage | 实际伤害应用 |
| `healthbar/health_bar_get` | 普通函数 | 无 | BOSS血条显示 |
| `healthbar/calculate_armor_type` | 普通函数 | 无 | 护甲类型计算 |
| `debuff/burn/tick` | 普通函数 | 无 | 燃烧持续伤害 |
