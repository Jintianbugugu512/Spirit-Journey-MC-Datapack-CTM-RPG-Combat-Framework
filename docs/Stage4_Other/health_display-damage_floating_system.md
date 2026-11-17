# 血量显示与飘字系统

## 📋 系统概述

 该系统在combat这个文件夹模块里.

本系统包含两个核心功能：
1. **敌人头顶血量条显示** - 实时显示敌人的生命值和防御类型
2. **伤害飘字系统** - 当玩家或敌人受到伤害/治疗时显示浮动数字

这边大概有两个：上面那个是敌人的血条和扣血显示，下面两张是玩家的血量和扣血显示。

它们会显示在屏幕的正前面，一般情况下都是能看到的，但偶尔有情况会看不到这个扣血显示，比如玩家被击退，或者玩家看的视角比较刁钻。

![玩家回血](health_display-damage_floating_system.assets/玩家回血.gif)

![玩家扣血](health_display-damage_floating_system.assets/玩家扣血.gif)

![敌人血条](health_display-damage_floating_system.assets/敌人血条.gif)

---

## 🩸 血量条显示系统

### 视觉效果

敌人头顶会显示一个由竖线符号（`|`）组成的血量条，格式如下：
```
§a||||||||||||||||||||  §7150■
```

- **血量条**：由20个竖线组成，根据血量百分比显示不同颜色
- **数值显示**：右侧显示当前血量数字
- **防御类型标识**：血量数字后的符号表示敌人的防御类型

### 颜色分段

| 血量百分比 | 颜色 | 显示范围 |
|-----------|------|---------|
| 60% - 100% | 🟢 绿色 (`§a`) | 12-20 格 |
| 20% - 59% | 🟡 黄色 (`§e`) | 4-11 格 |
| 0% - 19% | 🔴 红色 (`§c`) | 1-3 格 |

未显示的部分用灰色（`§8`）填充。

### 防御类型标识

| 符号 | armor_type | 含义 |
|-----|-----------|------|
| `■` | 1 | 物理防御型 |
| `●` | 2 | 魔法防御型 |
| `★` | 3 | 特殊防御型 |

### 实现原理

#### 1. 血量条计算流程

```
current_health / max_health × 100 = 血量百分比
血量百分比 / 5 = 血量条格数（0-20）
```

**示例**：
- 敌人血量 150/200
- 百分比 = 150 × 100 / 200 = 75%
- 血条格数 = 75 / 5 = 15 格（绿色）

#### 2. 性能优化

**优化1：仅在血量变化时更新**
```mcfunction
# 检测血量是否变化
execute if score @s current_health = @s last_health run return fail

# 更新记录值
scoreboard players operation @s last_health = @s current_health
```

**优化2：防御类型缓存**
```mcfunction
# 从缓存读取，不重新计算
execute if score @s armor_type matches 1 run data modify storage healthbar:display type set value "■"
```

#### 3. 核心代码位置

| 文件路径 | 功能 |
|---------|------|
| `combat:healthbar/update_healthbar_display` | 主更新函数 |
| `combat:healthbar/bars20/green_XX` | 绿色血条（12-20格） |
| `combat:healthbar/bars20/yellow_XX` | 黄色血条（4-11格） |
| `combat:healthbar/bars20/red_XX` | 红色血条（1-3格） |
| `z_loop:entity_manager` | 每tick调用更新 |

#### 4. 调用时机

```mcfunction
# entity_manager.mcfunction
execute as @e[tag=enemy] at @s run function combat:healthbar/update_healthbar_display
```

每tick对所有带有 `enemy` 标签的实体执行，但只有血量变化时才会实际更新显示。

---

## 💥 伤害飘字系统

### 视觉效果

#### 敌人受伤/回血（头顶飘字）

- **位置**：敌人实体上方 2.1 格
- **伤害显示**：`§l5`（白色粗体）
- **治疗显示**：`§2§l3`（绿色粗体）

#### 玩家受伤/回血（面前飘字）

- **位置**：玩家视线前方 1 格、左侧 0.6 格（第一人称视角位置）
- **伤害显示**：`§4§l-5`（红色粗体，带减号）
- **治疗显示**：`§2§l+3`（绿色粗体，带加号）

### 飘字属性

| 属性 | 数值 | 说明 |
|-----|------|------|
| 显示时长 | 10 tick | 约 0.5 秒 |
| 上升速度 | 0.04 格/tick | 每tick向上漂浮 |
| 对齐方式 | center | 居中对齐 |
| 背景色 | 1942251 | 半透明深色背景 |
| 穿墙显示 | true | 可透过方块看到 |

### 颜色规则

| 颜色 | 代码 | 触发条件 |
|-----|------|---------|
| 🔴 红色 | `§4` | 玩家受到伤害 |
| ⚪ 白色 | `§l` | 敌人受到伤害 |
| 🟢 绿色 | `§2` | 玩家/敌人恢复生命 |

### 实现原理

#### 1. 敌人飘字系统

**检测流程**（`combat:healthbar/show_damage`）：

```mcfunction
# 1. 记录当前血量
execute store result score @s h1 run scoreboard players get @s current_health

# 2. 计算血量差值
scoreboard players operation @s h2 -= @s h1

# 3. 伤害飘字（h2为正值 = 血量减少）
execute if entity @s[scores={h2=1..}] run execute store result storage damage:damage damage_ int 1 run scoreboard players get @s h2
execute if entity @s[scores={h2=1..}] at @s run function combat:healthbar/damage

# 4. 治疗飘字（h2为负值 = 血量增加）
execute if entity @s[scores={h2=..-1}] run execute store result storage damage:damage damage_ int -1 run scoreboard players get @s h2
execute if entity @s[scores={h2=..-1}] at @s run function combat:healthbar/heal_

# 5. 更新记录值
scoreboard players operation @s h2 = @s h1
```

**生成飘字**（`combat:healthbar/damage`）：

```mcfunction
# 在敌人上方2.1格召唤text_display实体
execute anchored eyes at @s positioned ~ ~2.1 ~ summon text_display as @s at @s run function combat:healthbar/damage_merge
```

**飘字数据**（`combat:healthbar/damage_merge`）：

```mcfunction
# 白色粗体，带damage_shower标签，10tick后消失
data merge entity @s {
    text:'"§l$(damage_)"',
    Tags:["damage_shower"],
    billboard:center,
    background:1942251,
    see_through:true
}
scoreboard players set @s countDown 10
```

#### 2. 玩家飘字系统

**检测流程**（`combat:healthbar/damage_show_fp/show_damage_fp`）：

```mcfunction
# 1. 读取原版Health数据
execute store result score @s h1 run data get entity @s Health 1

# 2. 计算差值
scoreboard players operation @s h2 -= @s h1

# 3. 伤害飘字（显示红色带减号）
execute if entity @s[scores={h2=1..}] run execute store result storage damage:damage damage_ float 1 run scoreboard players get @s h2
execute if entity @s[scores={h2=1..}] as @s at @s run function combat:healthbar/damage_show_fp/damage

# 4. 治疗飘字（显示绿色带加号）
execute if entity @s[scores={h2=..-1}] as @s at @s run function combat:healthbar/damage_show_fp/heal_

# 5. 更新记录
scoreboard players operation @s h2 = @s h1
```

**生成飘字**（`combat:healthbar/damage_show_fp/damage`）：

```mcfunction
# 在玩家视线相对位置召唤（左前方）
execute anchored eyes at @s positioned ^-0.6 ^ ^1.0 summon text_display as @s at @s run function combat:healthbar/damage_show_fp/damage_merge
```

**飘字数据**（`combat:healthbar/damage_show_fp/damage_merge`）：

```mcfunction
# 红色粗体带减号
data merge entity @s {
    text:'"§4§l-$(damage_)"',
    Tags:["damage_shower"],
    billboard:center,
    background:1942251,
    see_through:true
}
scoreboard players set @s countDown 10
```

#### 3. 飘字动画系统

**上升动画**（`z_loop:move_manager`）：

```mcfunction
# 每tick对所有飘字实体向上移动0.04格
execute as @e[tag=damage_shower] at @s run tp ~ ~0.04 ~
```

**自动消失**：通过 `countDown` 计分板实现，10tick后实体被清除。

#### 4. 调用时机

```mcfunction
# player_manager.mcfunction - 每tick执行
execute as @a at @s run function combat:healthbar/damage_show_fp/show_damage_fp

# entity_manager.mcfunction - 每tick执行
execute as @e[tag=enemy] at @s run function combat:healthbar/show_damage
```

---

## 🔧 技术细节

### 记分板变量

| 变量名 | 用途 |
|-------|------|
| `current_health` | 当前生命值 |
| `max_health` | 最大生命值 |
| `last_health` | 上一tick的生命值（用于变化检测） |
| `h1` | 临时变量：当前血量快照 |
| `h2` | 临时变量：血量差值 |
| `h3`, `h4` | 原版Health处理（玩家系统） |
| `armor_type` | 防御类型（1=物理, 2=魔法, 3=特殊） |
| `countDown` | 飘字倒计时 |

### 数据存储

```mcfunction
# 血量条数据
storage healthbar:display
├── current (int)    # 当前血量数值
└── type (string)    # 防御类型符号

# 飘字数据
storage damage:damage
└── damage_ (int/float)  # 伤害/治疗数值
```

### 坐标系统

**敌人飘字**：世界坐标
```mcfunction
positioned ~ ~2.1 ~  # 实体上方2.1格
```

**玩家飘字**：相对坐标（局部坐标系）
```mcfunction
positioned ^-0.6 ^ ^1.0  # 左0.6格，前1格，眼睛高度
```

---

## 📊 系统流程图

### 血量条更新流程

```
敌人实体 (每tick)
    ↓
检查血量是否变化
    ↓
[未变化] → 跳过更新 (性能优化)
[已变化] → 继续
    ↓
计算血量百分比
    ↓
计算血条格数 (0-20)
    ↓
根据百分比选择颜色
    ↓
从缓存读取防御类型
    ↓
更新CustomName显示
```

### 飘字生成流程

```
玩家/敌人 (每tick)
    ↓
读取当前血量
    ↓
计算与上次的差值
    ↓
[差值 > 0] → 召唤伤害飘字
[差值 < 0] → 召唤治疗飘字
[差值 = 0] → 跳过
    ↓
设置飘字属性（颜色、文本、标签）
    ↓
设置倒计时 (10 tick)
    ↓
每tick上升 0.04 格
    ↓
倒计时归零后消失
```

---

## 📁 文件结构

```
data/combat/functions/healthbar/
├── update_healthbar_display.mcfunction   # 血量条主更新函数
├── show_damage.mcfunction                # 敌人飘字检测
├── damage.mcfunction                     # 敌人伤害飘字召唤
├── heal_.mcfunction                      # 敌人治疗飘字召唤
├── damage_merge.mcfunction               # 敌人伤害飘字数据
├── heal_merge.mcfunction                 # 敌人治疗飘字数据
├── health_bar_get.mcfunction             # Boss血量条获取
├── health_bar_name.mcfunction            # Boss血量条命名
├── bars20/
│   ├── green_20.mcfunction ~ green_12.mcfunction   # 绿色血条
│   ├── yellow_11.mcfunction ~ yellow_4.mcfunction  # 黄色血条
│   └── red_3.mcfunction ~ red_1.mcfunction         # 红色血条
└── damage_show_fp/
    ├── show_damage_fp.mcfunction         # 玩家飘字检测
    ├── damage.mcfunction                 # 玩家伤害飘字召唤
    ├── heal_.mcfunction                  # 玩家治疗飘字召唤
    ├── damage_merge.mcfunction           # 玩家伤害飘字数据
    └── heal_merge.mcfunction             # 玩家治疗飘字数据

data/z_loop/functions/
├── entity_manager.mcfunction             # 敌人系统主循环
├── player_manager.mcfunction             # 玩家系统主循环
└── move_manager.mcfunction               # 飘字动画管理
```
