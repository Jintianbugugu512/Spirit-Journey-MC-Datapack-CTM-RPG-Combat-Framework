# Z_Loop 主循环系统文档

---

## 📋 系统概述

`z_loop` 是整个数据包的**核心主循环系统**，负责每tick协调和管理所有游戏系统的运行。所有管理器都通过 `minecraft:tick` 函数标签自动每tick执行，**统一调度** - 协调玩家、敌人、实体、技能等所有系统.

**备注：这个模块简单过一遍就可以了,它主要是全局的一个调度器,负责调用各个系统,没有具体的功能实现.**

---

## 执行入口

### minecraft:tick 函数标签

**文件位置**: `data/minecraft/tags/functions/tick.json`

**执行顺序**:
```json
{
  "values": [
    "z_loop:player_manager",      // 1. 玩家管理器
    "z_loop:entity_manager",      // 2. 实体管理器
    "z_loop:kill_manager",        // 3. 清理管理器
    "z_loop:move_manager",        // 4. 移动管理器
    "z_loop:particle_manager",    // 5. 粒子管理器
    "z_loop:interval_manager",    // 6. 间隔管理器
    "z_loop:tag_manager"          // 7. 标签管理器
  ]
}
```

---

## 七大管理器

### 1️⃣ Player Manager - 玩家管理器

**文件**: `player_manager.mcfunction`

**职责**: 管理所有玩家的行为和状态

**执行内容**:
```mcfunction
# 对每个玩家执行通用处理函数
execute as @a[tag=player_1] run function player:player_common {"id":"1"}
execute as @a[tag=player_2] run function player:player_common {"id":"2"}
execute as @a[tag=player_3] run function player:player_common {"id":"3"}

# 玩家自身血量与伤害显示
execute as @a at @s run function combat:healthbar/damage_show_fp/show_damage_fp

# 消耗品与战斗系统管理
function combat:consumable/manage

# 对话系统
function talk:talk_manager

# 技能CD/最大MP限制
function skill:cd/manager

# 重置检测变量
scoreboard players set @a put_shift 0
scoreboard players set @a jump 0
tag @a remove left_click
tag @a remove right_click
```

**调用链**:
```
player_manager
    ↓
player:player_common {"id":"X"}
    ↓
    ├─ anti_cheat:ban_tp (反作弊)
    ├─ player:click_detect (点击检测)
    ├─ soul_disc:soul_manager (魂盘系统)
    ├─ player:attribute/modify (属性修改)
    ├─ player:shift (技能释放)
    ├─ skill:check_all_skills (技能检测)
    ├─ combat:attack_generic/* (武器攻击)
    └─ combat:healthbar/* (血量显示)
```

---

### 2️⃣ Entity Manager - 实体管理器

**文件**: `entity_manager.mcfunction`

**职责**: 管理所有非玩家实体（敌人、Boss、子弹、技能实体等）

**执行内容**:

#### 敌人系统
```mcfunction
# 血量限制
execute as @e[tag=enemy] at @s if score @s current_health >= @s max_health run scoreboard players operation @s current_health = @s max_health

# 血条与伤害显示
execute as @e[tag=enemy] at @s run function combat:healthbar/show_damage
execute as @e[tag=enemy] at @s run function combat:healthbar/update_healthbar_display

# 发光效果管理
effect clear @e[tag=enemy,tag=!can_glow] glowing

# 受击冷却
execute as @e[tag=enemy,scores={hit_cooldown=1..}] run scoreboard players remove @s hit_cooldown 1
```

#### Boss系统（区域优化）
```mcfunction
# 紫晶核心Boss（仅在玩家靠近时执行）
execute if entity @a[x=-1013,y=100,z=1013,distance=..50] if entity @e[tag=chess_grid] run function boss:amethyst_core/chess_grid_tick
execute if entity @a[x=-1013,y=100,z=1013,distance=..50] as @e[tag=amethyst_core] at @s run function boss:amethyst_core/main

# 水滴骑士Boss
execute if entity @a[x=-824,y=100,z=1002,distance=..50] as @e[tag=water_knight] at @s run function boss:water_knight/main

# 铁面教官Boss
execute if entity @a[x=-651,y=100,z=1020,distance=..50] as @e[tag=iron_instructor] at @s run function boss:iron_instructor/main
```

#### 敌人技能实体
```mcfunction
# 攻击预警系统
execute as @e[tag=square_warning] at @s run function enemy_skill:warning/square_warn/tick
execute as @e[tag=circle_warning] at @s run function enemy_skill:warning/circle_warn/tick
execute as @e[tag=purple_warning] at @s run function enemy_skill:warning/purple_warn/tick

# 延迟伤害
execute as @e[tag=delayed_damage] at @s run function enemy_skill:delayed_damage/tick
```

#### 子弹系统
```mcfunction
# 动量子弹
execute as @e[tag=motion_bullet] run function enemy_skill:bullet/motion_bullet/manage

# 瞄准子弹
execute as @e[tag=aim_item_bullet] at @s on passengers run function enemy_skill:bullet/aim_bullet/item_bullet/check_collision
```

#### Debuff管理
```mcfunction
# 减防buff
execute if entity @e[tag=enemy,scores={defense_reduce_duration=1..}] run function combat:debuff/defense_reduce/manager

# 眩晕效果
execute if entity @e[tag=enemy,scores={stun=1..}] run function combat:debuff/stun/manager

# 燃烧伤害
function combat:debuff/burn/manager
```

#### 场景机制
```mcfunction
# 场景追踪炮弹
execute as @e[tag=scene_bomb_bullet] at @s run function area:amethyst_cave/homing_bomb_tick

# 水崩实体
execute as @e[tag=water_burst] at @s run function area:water_cave/water_burst_tick

# 持灯悦灵
execute as @e[tag=hold_lantern] at @s run function area:floating_cemetery/lantern_allay_tick
```

#### 世界交互实体
```mcfunction
# 世界物品显示
execute as @e[type=item_display,tag=item_on_world] at @s run function display:item_show

# 传送点交互
execute as @e[type=interaction,tag=tp_point] at @s run function y_other:teleport/tp_point

# 门交互
execute as @e[type=interaction,tag=door] at @s run function y_other:z_budao/tran/door
```

**性能优化要点**:
- ✅ Boss系统使用区域检测（仅在玩家附近执行）
- ✅ Debuff使用条件检测（仅在有目标时执行）
- ✅ 燃烧系统无条件调用（避免每tick NBT查询开销）

---

### 3️⃣ Kill Manager - 清理管理器

**文件**: `kill_manager.mcfunction`

**职责**: 统一管理实体的清理和死亡逻辑

**执行内容**:
```mcfunction
# 倒计时实体归零死亡
kill @e[scores={countDown=..-1}]

# 清理落地箭矢
kill @e[type=arrow,nbt={inGround:1b}]
kill @e[type=minecraft:spectral_arrow,nbt={inGround:1b}]

# 怪物死亡
# 触发亡语技能（死亡前执行）
execute as @e[tag=enemy,scores={current_health=..0}] at @s run function enemy_skill:deathrattle/manager
# 击杀敌人
kill @e[tag=enemy,scores={current_health=..0}]

# 清除落下的物体
execute as @e[tag=fall] at @s unless block ~ ~-0.1 ~ air run kill @e[tag=fall]
```

**设计细节**:
- **倒计时=-1才清除** - 允许用 `countDown=0` 表示"最后1个tick执行特殊逻辑"
- **亡语优先** - 先触发死亡技能，再击杀敌人
- **箭矢清理** - 防止落地箭矢累积影响性能

**注释说明**:
> 早期开发用来统一kill逻辑的，后期开发的kill逻辑都写在相关模块里，不写这了。

---

### 4️⃣ Move Manager - 移动管理器

**文件**: `move_manager.mcfunction`

**职责**: 管理特殊实体的移动逻辑

**执行内容**:
```mcfunction
# 传送特效实体旋转
execute as @e[tag=tp,scores={countDown=3..}] at @s run function display:other/horizontal_orbit {"tag":"tp","r":4,"h":"0","speed":18}
execute as @e[tag=tp_,scores={countDown=3..}] at @s run function display:other/horizontal_orbit {"tag":"tp_","r":4,"h":"0","speed":"-18"}

# 传送轴心上升
execute as @e[tag=tp_pivot] at @s run tp ~ ~0.04 ~
execute as @e[tag=tp__pivot] at @s run tp ~ ~0.04 ~

# 下落实体
execute as @e[tag=fall] at @s run tp ~ ~-0.1 ~

# 伤害和MP显示上升
execute as @e[tag=damage_shower] at @s run tp ~ ~0.04 ~
execute as @e[tag=MP_shower_] at @s run tp ~ ~0.04 ~

# 万剑归宗剑的移动
execute as @e[tag=sword_attacker] at @s rotated ~ ~ run tp @s ^-0.2 ^0.2 ^

# 悬赏之刃剑的移动
execute as @e[tag=bounty_sword] at @s run function skill:sword_skill/bounty_blade/bounty_blade_move
```

**注释说明**:
> 早期开发用来统一tp移动的，后期开发的移动逻辑都写在相关模块里，不写这了。

---

### 5️⃣ Particle Manager - 粒子管理器

**文件**: `particle_manager.mcfunction`

**职责**: 管理全局粒子效果

**执行内容**:
```mcfunction
# 传送特效
execute as @e[tag=tp] at @s run particle squid_ink ~ ~ ~ 0.3 0.2 0.3 0 5
execute as @e[tag=tp_] at @s run particle glow_squid_ink ~ ~ ~ 0.3 0.2 0.3 0 5

# 狙击子弹
execute as @e[tag=snipe_bullet_] at @s run particle end_rod ~ ~ ~ 0.3 0.2 0.3 0 2

# 散杖子弹
execute as @e[tag=stick_bullet] at @s run particle enchant ~ ~ ~ 0.3 0.2 0.3 0 10

# 剑光
execute as @e[tag=light] at @s run particle end_rod ~ ~ ~ 0 0 0 0 1

# 御血buff
execute as @a[scores={blood_buff=1..}] at @s run particle lava ~ ~ ~ 0.8 0.5 0.8 0 8
```

**注释说明**:
> 早期开发用来统一管理粒子效果了，后期开发的粒子效果都写在相关模块里，不写这了。

---

### 6️⃣ Interval Manager - 间隔管理器

**文件**: `interval_manager.mcfunction`

**职责**: 管理各种计时器和间隔逻辑

**执行内容**:

#### 玩家间隔变量
```mcfunction
# 位置记录间隔（反作弊系统用）
scoreboard players add @a[scores={interval_pos_record=..10}] interval_pos_record 1
scoreboard players set @a[scores={interval_pos_record=10..}] interval_pos_record 0

# 传送冷却
scoreboard players remove @a[scores={interval_can_tp=1..}] interval_can_tp 1

# 输出间隔
scoreboard players remove @a[scores={interval_output=1..}] interval_output 1

# 效果免疫
scoreboard players remove @a[scores={can_be_effect=1..}] can_be_effect 1

# 知识的力量buff
scoreboard players remove @a[scores={knowledge_power=1..}] knowledge_power 1
```

#### 实体倒计时
```mcfunction
# 通用倒计时
scoreboard players remove @e[scores={countDown=0..}] countDown 1

# 散杖攻击间隔
scoreboard players remove @e[scores={interval_stick=1..}] interval_stick 1
```

#### 定时任务
```mcfunction
# 每100tick重置敌人原版血量（防止因原版血量耗尽而死亡）
scoreboard players add #health_reset_timer temp 1
execute if score #health_reset_timer temp matches 100.. run function combat:healthbar/reset_enemy_health
execute if score #health_reset_timer temp matches 100.. run scoreboard players set #health_reset_timer temp 0

# 每5tick管理技能槽位占位符
scoreboard players add #slot_timer temp 1
execute if score #slot_timer temp matches 5.. run function skill:manage_skill_slots
execute if score #slot_timer temp matches 5.. run scoreboard players set #slot_timer temp 0
```

**性能优化**:
- ✅ 敌人血量重置每100tick执行一次
- ✅ 技能槽位占位符每5tick检查一次

---

### 7️⃣ Tag Manager - 标签管理器

**文件**: `tag_manager.mcfunction`

**职责**: 管理实体标签的动态添加和移除

**执行内容**:

```mcfunction
# 为与Boss战斗中的玩家添加tag
execute as @a at @s if entity @e[distance=..50,tag=boss] run tag @s add battle_with_boss
execute as @a at @s unless entity @e[distance=..50,tag=boss] run tag @s remove battle_with_boss

# 为敌方与我方弹射物增加tag
execute as @e[type=arrow] at @s on origin run execute if entity @s[type=player] run tag @e[type=arrow,distance=..0.1] add bullet_from_player
execute as @e[type=arrow] at @s on origin run execute if entity @s[tag=enemy] run tag @e[type=arrow,distance=..0.1] add bullet_from_enemy
```

**标签用途**:
- `battle_with_boss` - 用于特殊UI或效果
- `bullet_from_player` - 区分玩家发射的箭
- `bullet_from_enemy` - 区分敌人发射的箭

**注释说明**:
> 早期开发用来统一管理tag添加的管理器，后期开发tag添加都写在相关模块里，不写这了。

---

## 完整执行流程

### 每个游戏Tick的执行顺序

```
minecraft:tick（游戏引擎自动调用）
    ↓
┌─────────────────────────────────────────────┐
│ 1. player_manager - 玩家管理器               │
│    ├─ 玩家输入检测                           │
│    ├─ 技能CD递减                            │
│    ├─ 技能释放和检测                         │
│    ├─ 武器攻击                              │
│    ├─ 血量/MP显示                           │
│    └─ 重置检测变量                          │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ 2. entity_manager - 实体管理器               │
│    ├─ 敌人血量和血条                         │
│    ├─ Boss AI（区域优化）                    │
│    ├─ 敌人技能实体（预警、延迟伤害）          │
│    ├─ 子弹碰撞检测                          │
│    ├─ Debuff管理（减防、眩晕、燃烧）          │
│    ├─ 场景机制（炮弹、水崩、悦灵）            │
│    └─ 世界交互实体（传送点、门）             │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ 3. kill_manager - 清理管理器                 │
│    ├─ 倒计时实体清除                         │
│    ├─ 落地箭矢清理                          │
│    ├─ 敌人死亡（触发亡语→击杀）              │
│    └─ 落下物体清理                          │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ 4. move_manager - 移动管理器                 │
│    ├─ 传送特效旋转                          │
│    ├─ 显示实体上升                          │
│    ├─ 下落实体移动                          │
│    └─ 技能实体移动                          │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ 5. particle_manager - 粒子管理器             │
│    ├─ 传送特效粒子                          │
│    ├─ 子弹轨迹粒子                          │
│    └─ Buff粒子                             │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ 6. interval_manager - 间隔管理器             │
│    ├─ 玩家间隔变量递减                       │
│    ├─ 实体倒计时递减                        │
│    ├─ 定时任务（每100tick重置血量）          │
│    └─ 定时任务（每5tick管理技能槽位）         │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ 7. tag_manager - 标签管理器                  │
│    ├─ Boss战斗标签                          │
│    └─ 弹射物来源标签                        │
└─────────────────────────────────────────────┘
```

---

### 3. 性能优化策略

#### 区域检测
```mcfunction
# Boss只在玩家附近时执行
execute if entity @a[x=-1013,y=100,z=1013,distance=..50] as @e[tag=amethyst_core] at @s run function boss:amethyst_core/main
```

#### 条件执行
```mcfunction
# Debuff只在有目标时执行
execute if entity @e[tag=enemy,scores={defense_reduce_duration=1..}] run function combat:debuff/defense_reduce/manager
```

#### 间隔执行
```mcfunction
# 每100tick执行一次
scoreboard players add #health_reset_timer temp 1
execute if score #health_reset_timer temp matches 100.. run function combat:healthbar/reset_enemy_health
```

#### 避免重复查询
```mcfunction
# 燃烧系统无条件调用（每tick读取原生Fire值）
# 如果用NBT查询会增加开销
function combat:debuff/burn/manager
```

---

### 4. 历史遗留说明

多个管理器包含注释：
> "早期开发用来统一XXX的，后期开发的XXX都写在相关模块里，不写这了。"

- 早期：集中式管理（所有逻辑写在z_loop）
- 后期：模块化管理（逻辑写在各自模块）
