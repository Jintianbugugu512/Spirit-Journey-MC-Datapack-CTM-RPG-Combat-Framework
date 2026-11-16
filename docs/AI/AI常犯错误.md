# AI 常犯错误记录

本文档记录 AI 在开发过程中常犯的错误，用于提醒和改进。

---

## ❌ 错误 1：非宏语句前加 `$` 前缀

### 错误描述
在 mcfunction 文件中，只有包含宏参数（如 `$(player)`, `$(id)` 等）的命令才需要在行首添加 `$` 前缀。非宏语句不需要也不应该添加 `$`。

### 错误示例
```mcfunction
# ❌ 错误：这些命令不包含宏参数，不应该加 $
$scoreboard players set @e[tag=summon,limit=1,sort=nearest] temp 0
$tag @e[tag=summon,limit=1,sort=nearest] remove summon
$execute at @s run tp @s ^ ^ ^0.4
```

### 正确示例
```mcfunction
# ✅ 正确：不包含宏参数的命令不加 $
scoreboard players set @e[tag=summon,limit=1,sort=nearest] temp 0
tag @e[tag=summon,limit=1,sort=nearest] remove summon
execute at @s run tp @s ^ ^ ^0.4

# ✅ 正确：包含宏参数的命令需要加 $
$execute as @e[tag=light_bullet_player_$(id)] run function skill:sword_skill/sword_light/light_move
$tag @e[tag=summon,limit=1,sort=nearest] add light_bullet_$(player)
$scoreboard players set @e[tag=light_bullet_$(player)] temp 0
```

### 判断规则
**简单判断方法**：
- 命令中包含 `$(xxx)` → **需要加 `$`**
- 命令中不包含 `$(xxx)` → **不需要加 `$`**


```

### 为什么会犯这个错误？
1. **惯性思维**：在宏函数中写了很多带 `$` 的命令，后续命令习惯性加上 `$`
2. **不清楚规则**：不理解 `$` 的作用是标记"这一行包含宏参数需要展开"
3. **复制粘贴**：从带宏参数的行复制后忘记删除 `$`

### 影响
- 带 `$` 的非宏语句会导致命令执行失败
- Minecraft 会报错：`Unknown or incomplete command`
- 功能无法正常运行

---

## 📝 其他常见错误（待补充）

### 错误 2：宏函数参数传递方式错误

#### 错误描述
使用 `with storage` 传递参数时，只会传递 storage 中实际存在的字段，不会传递调用时的宏参数本身（如 `id`）。如果子函数需要使用 `$(id)` 等不在 storage 中的参数，会报错"缺少传入参数"。

#### 错误示例
```mcfunction
# ❌ 错误：check_all_skills.mcfunction
$execute if entity @s[tag=whirlwind_active] run function skill:sword_skill/whirlwind/whirlwind_tick with storage damage:player_$(id)

# 子函数 whirlwind_tick.mcfunction 中使用：
$execute if score @s temp2 matches 0 at @s run function skill:sword_skill/whirlwind/whirlwind_attack {id:"$(id)"}

# 报错：缺少传入参数 id
# 原因：with storage damage:player_1 只传递了 storage 中的字段（player, attack等），没有传递 id 参数本身
```

#### 正确示例
```mcfunction
# ✅ 方式1：直接传递 JSON 参数（推荐）
$execute if entity @s[tag=whirlwind_active] run function skill:sword_skill/whirlwind/whirlwind_tick {id:"$(id)"}

# ✅ 方式2：使用 storage 中已有的字段
# 在 storage damage:player_1 中预先存储 id:"1"
$execute if entity @s[tag=whirlwind_active] run function skill:sword_skill/whirlwind/whirlwind_tick with storage damage:player_$(id)
# 子函数中使用 $(player), $(attack), $(id) 等 storage 中实际存在的字段
```

#### 原因分析
- `with storage damage:player_1` 传递的是 **storage 中的数据内容**，而不是调用时的参数
- 例如 `storage damage:player_1` 包含：`{player:"player_1", attack:100, defense:50}`
- 子函数可以使用 `$(player)`, `$(attack)`, `$(defense)`
- 但**无法使用** `$(id)`，因为 `id` 不在 storage 中

#### 解决方案
1. **直接传递所需参数**（推荐）：`{id:"$(id)"}` - 明确、清晰
2. **修改 storage 结构**：在 storage 中添加 `id` 字段，然后用 `with storage` 传递
3. **避免嵌套宏调用**：在顶层函数中一次性传递所有需要的参数

#### 调试方法
如果遇到"缺少传入参数 xxx"错误：
1. 检查调用时使用的是 `{key:"value"}` 还是 `with storage`
2. 如果是 `with storage`，检查 storage 中是否包含该字段
3. 如果 storage 中没有，改用直接传递 `{key:"value"}` 的方式

---

### 错误 3：粒子命令格式错误（版本兼容性）

#### 错误描述
在 Minecraft 1.20.4 中使用了 1.20.5+ 的新粒子语法，导致命令执行失败。

#### 错误示例
```mcfunction
# ❌ 错误：1.20.5+ 的新语法
particle block{block_state:"minecraft:dirt"} ~ ~0.2 ~ 0.3 0.2 0.3 0 3 force
```

#### 正确示例
```mcfunction
# ✅ 正确：1.20.4 的旧语法
particle block minecraft:dirt ~ ~0.2 ~ 0.3 0.2 0.3 0 3 force
```

#### 版本差异
- **1.20.4 及之前**：`particle block <方块ID>`
- **1.20.5+**：`particle block{block_state:"<方块ID>"}`

#### 其他粒子类型示例
```mcfunction
# 方块粒子（1.20.4）
particle block minecraft:stone ~ ~ ~ 0.5 0.5 0.5 0 10 force
particle block minecraft:dirt ~ ~ ~ 0.3 0.2 0.3 0 5 force

# 物品粒子（1.20.4）
particle item minecraft:diamond ~ ~ ~ 0.2 0.2 0.2 0.05 20 force

# 灰尘粒子（1.20.4）
particle dust 1 0 0 1 ~ ~ ~ 0.5 0.5 0.5 0 10 force
```

#### 为什么会犯这个错误？
1. **训练数据混淆**：AI训练数据包含多个Minecraft版本的命令格式
2. **使用新文档**：参考了1.20.5+的wiki或文档
3. **惯性使用**：习惯了新版本的JSON格式

#### 影响
- 命令执行失败，报错"Expected whitespace to end one argument"
- 粒子效果无法显示

---

### 错误 4：damage 命令对多个实体使用错误

#### 错误描述
`damage` 命令只能对单个实体使用，不能直接用选择器选择多个实体。如果需要对多个实体造成伤害，必须使用 `execute as` 循环处理。

#### 错误示例
```mcfunction
# ❌ 错误：直接对多个玩家使用 damage 命令
damage @a[distance=..3] 8

# 报错：选择器匹配了多个实体，但 damage 命令只接受单个目标
```

#### 正确示例
```mcfunction
# ✅ 正确：使用 execute as 对每个玩家分别执行 damage
execute as @a[distance=..3] run damage @s 8

# ✅ 正确：使用 limit 限制为单个实体（如果只想伤害一个）
damage @a[distance=..3,limit=1] 8
```

#### 原理
- `damage` 命令的目标参数必须是**单个实体**
- 选择器 `@a[distance=..3]` 可能匹配多个玩家
- 需要用 `execute as` 将命令转换为"对每个匹配实体执行"
- 或者用 `limit=1` 明确限制为单个目标

#### 其他类似命令
以下命令也有相同的限制，需要单个实体目标：
- `damage` - 造成伤害
- `ride` - 骑乘控制
- `attribute` - 属性修改
- 某些 `data` 操作

#### 对比：可以接受多实体的命令
这些命令可以直接对多个实体使用：
```mcfunction
# ✅ 这些命令支持多实体选择器
kill @e[type=zombie,distance=..10]
tp @a[distance=..5] ~ ~10 ~
effect give @a[distance=..3] regeneration 10 1
tag @e[tag=enemy,distance=..5] add marked
```

---

### 错误 5：宏函数调用时使用了错误的参数传递方式导致函数未执行

#### 错误描述
在已经是宏函数的上下文中调用子函数时，错误地使用了 `with storage` 来传递宏参数，导致子函数根本没有被执行。应该直接使用 JSON 参数传递。

#### 错误示例
```mcfunction
# player_common.mcfunction（本身是宏函数，有 $(id) 参数）
# ❌ 错误：使用 with storage 传递包含宏参数的 storage 路径
$execute at @s run function soul_disc:damage_reaction/check with storage damage:player_$(id)

# 结果：check 函数根本不会被执行，里面的 say 语句等都不会运行
```

#### 正确示例
```mcfunction
# player_common.mcfunction
# ✅ 正确：直接传递 JSON 参数
$function soul_disc:damage_reaction/check {"id":"$(id)"}

# ✅ 或者如果不需要在 execute 上下文中：
$execute at @s run function soul_disc:damage_reaction/check {"id":"$(id)"}
```

#### 原因分析
1. **`with storage damage:player_$(id)` 在宏展开时有问题**
   - 虽然 `$(id)` 会被展开成 `1`、`2`、`3`
   - 但 `with storage` 后面跟的路径在宏函数中可能无法正确解析
   - 导致函数调用失败，函数内的代码完全不执行

2. **混淆了两种参数传递方式**
   - `with storage` 适用于从预先存储的 storage 中读取数据
   - JSON 参数 `{"key":"value"}` 适用于直接传递具体值
   - 在宏函数中传递宏参数，应该用 JSON 方式

#### 判断规则
**在宏函数中调用子函数：**
- 需要传递当前的宏参数（如 `$(id)`）→ 使用 `{"id":"$(id)"}`
- 需要传递固定的 storage 数据 → 使用 `with storage namespace:path`
- 需要传递 storage 中的某个字段 → 确保该字段真实存在于 storage 中

#### 调试方法
如果子函数完全不执行（连无条件的 `say` 或 `tellraw` 都不显示）：
1. 检查调用语句的参数传递方式
2. 如果使用了 `with storage`，尝试改为 JSON 参数 `{}`
3. 在子函数开头添加无条件的调试语句验证是否被调用

#### 为什么会犯这个错误？
1. **看到其他地方用 `with storage`**：项目中其他函数使用了 `with storage`，习惯性照搬
2. **不理解两种方式的区别**：没有理解 `with storage` 和 JSON 参数的适用场景
3. **宏参数嵌套混淆**：在宏函数中使用 `$(id)` 构造 storage 路径，导致解析失败
