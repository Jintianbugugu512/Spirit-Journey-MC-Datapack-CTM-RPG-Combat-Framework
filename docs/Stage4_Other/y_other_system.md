# Y_Other 杂项模块文档

---

## 📋 系统概述

`y_other` 是一个**杂项功能集合模块**，包含了各种不属于核心战斗系统的辅助功能、工具、初始化脚本和地图专用机制。这个模块比较杂乱，涵盖了从游戏初始化、传送系统、道具功能到调试工具等各种功能。

---

## 文件结构

```
y_other/functions/
├── init/                         # 初始化系统
│   ├── start.mcfunction          # 游戏开始初始化
│   ├── scoreboard_creator.mcfunction  # 创建所有记分板
│   └── tip.mcfunction            # 提示信息
├── teleport/                     # 传送系统
│   ├── tp_list.mcfunction        # 传送点列表
│   ├── tp_point.mcfunction       # 传送点功能
│   ├── tp_save_point.mcfunction  # 保存传送点
│   ├── tp_particle.mcfunction    # 传送粒子效果
│   └── tp_boss.mcfunction        # Boss传送
├── prop/                         # 道具功能
│   ├── compass.mcfunction        # 罗盘探测功能
│   ├── pitcher_plant.mcfunction  # 瓶子草道具
│   └── torchflower.mcfunction    # 火把花道具
├── get/                          # 快速给予物品（开发用）
│   ├── get.mcfunction            # 给予武器
│   ├── getenemy.mcfunction       # 给予敌人
│   └── getnpc.mcfunction         # 给予NPC
├── z_budao/                      # 地图专用系统（不道地图？）
│   ├── player_manual/            # 玩家手册系统
│   ├── select_player/            # 玩家选择
│   ├── tran/                     # 转换/传送门系统
│   ├── enemy.mcfunction          # 敌人管理
│   ├── entity.mcfunction         # 实体管理
│   ├── add_magic.mcfunction      # 增加魔法强度
│   ├── add_physic.mcfunction     # 增加物理攻击
│   └── reset.mcfunction          # 重置功能
├── debug.mcfunction              # 调试功能
└── name_modify.mcfunction        # 名称修改
```

---

## 核心功能模块

### 🎬 初始化系统 (init/)

#### start.mcfunction - 游戏开始初始化

**功能：** 在玩家准备齐全后执行，初始化游戏基础数据

**初始化内容：**
```mcfunction
# 玩家基础属性
scoreboard players set @a max_Flower 3      # 最大花朵数
scoreboard players set @a max_soul_disc 1   # 最大魂盘数
scoreboard players set @a max_MP 100        # 最大MP
scoreboard players set @a max_HP 20         # 最大HP
scoreboard players set @a strength 50       # 力量
scoreboard players set @a armor 0           # 护甲

# 玩家storage初始化
data merge storage damage:player_1 {
    player:"player_1",
    damage:1,
    rate:1,
    sword_skill_y:283,
    crossbow_skill_y:284,
    stick_skill_y:285
}

# 显示板设置
scoreboard objectives add MP dummy "『MP』"
scoreboard objectives setdisplay sidebar MP
scoreboard objectives add HP health "『HP』"
scoreboard objectives setdisplay below_name HP
```

**调用时机：** 游戏开始时手动调用

---

#### scoreboard_creator.mcfunction - 记分板创建器

**功能：** 创建游戏所需的所有记分板

---

### 🚪 传送系统 (teleport/)

#### 传送点系统

**功能：** 管理地图中的传送点，玩家可以通过交互实体传送到已解锁的地点

**工作流程：**
```
玩家右键点击传送点实体
    ↓
tp_list.mcfunction
    ├─ 检测玩家已解锁的传送点
    ├─ 在聊天框显示可用传送点列表
    └─ 提供可点击的传送链接
    
玩家点击传送链接
    ↓
执行传送命令
    ↓
tp_particle.mcfunction（传送特效）
```

**传送点解锁机制：**
```mcfunction
# 玩家经过传送点时自动解锁
execute if entity @s[tag=0_point] run tag @a[tag=player_1] add 0_point
execute if entity @s[tag=1_point] run tag @a[tag=player_1] add 1_point
...
```

**传送点列表示例：**
```mcfunction
tellraw @p "§l§3---------------『点击以选择传送地点』---------------"
execute if entity @a[tag=player_1,tag=0_point] run tellraw @p [
    {"text":"§e『穴居村落』",
     "clickEvent":{"action":"run_command","value":"/tp @s ~ ~10 ~"},
     "hoverEvent":{"action":"show_text","value":"§9左键此处传送至§e『穴居村落』"}
    }
]
```

**视觉效果：**
- 传送点方块会旋转动画（使用block_display的transformation）
- 传送时播放粒子效果和音效

---

### 🧰 道具功能 (prop/)

#### compass.mcfunction - 探索者罗盘

**功能：** 探测30格内隐藏的物品（技能书、金萝卜等）

**工作原理：**
```mcfunction
# 检测范围内物品
execute unless entity @e[tag=item_on_world,distance=..30] run title @s actionbar "§3『30格内未发现存在任何物品』"
execute if entity @e[tag=item_on_world,distance=..30] run title @s actionbar "§3『罗盘探测到了§l30§3格内存在物品...』"
execute if entity @e[tag=item_on_world,distance=..20] run title @s actionbar "§9『罗盘探测到了§l20§9格内存在物品...』"
execute if entity @e[tag=item_on_world,distance=..10] run title @s actionbar "§4『罗盘探测到了§l10§4格内存在物品...』"

# 设置罗盘指向目标
execute store result score @s compass_x run data get entity @e[tag=item_on_world,limit=1,sort=nearest] Pos[0]
execute store result score @s compass_z run data get entity @e[tag=item_on_world,limit=1,sort=nearest] Pos[2]
```

**提示颜色：**
- 🔵 30格内 - 青色提示
- 🔷 20格内 - 蓝色提示
- 🔴 10格内 - 红色提示

---

#### pitcher_plant / torchflower - 特殊道具

**功能：** 我想把这两个花（火把花和瓶子草）作为场景中的可拾取道具.玩家站在火把花和瓶子草上按 Shift 键就可以拾取,拾取后可以直接进行回蓝或者回血.

---

### 🎮 快速给予系统 (get/)

#### get.mcfunction - 快速给予武器和技能

**功能：** 这个函数是和 gettemp 一起使用的。

get 函数是整个地图的数据库，地图中的所有物品都存储在这里，每一条指令的存储用的是一个 give 指令。当需要获取物品的时候，就把所需物品的 give 指令复制到 get Temporary 中，然后进入地图调用 get temporary 来获取想要的物品。

---

### 🏗️ 地图专用系统 (z_budao/)

**说明：** 你说这个函数为什么叫 budao 呢?很简单啦,因为我基本上项目里有什么垃圾函数不知道放哪里,那就把这些函数扔到这里来.很多函数是废弃不用的,这个可以不看.

---

### 🐛 调试工具

#### debug.mcfunction - 调试功能

**功能：** 开发时的调试辅助功能

---

#### name_modify.mcfunction - 名称修改

**功能：** 修改实体或物品的显示名称

对,这个是我当时做地图的时候用的.我会把这个函数放到一个固定的命令方块里,这样就可以每次快捷地修改这些物品和道具的名字,不用每次都自己输指令.

---

