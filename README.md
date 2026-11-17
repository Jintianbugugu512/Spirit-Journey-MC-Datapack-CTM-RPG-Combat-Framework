<div align="center">

# 🌌 魂旅 (Spirit Journey)

**大型纯原版 Minecraft CTM-RPG 数据包开发框架**

[![Minecraft Version](https://img.shields.io/badge/Minecraft-1.20.4-brightgreen.svg)](https://www.minecraft.net/)
[![Pack Format](https://img.shields.io/badge/Pack_Format-17-blue.svg)](https://minecraft.wiki/w/Pack_format)
[![Functions](https://img.shields.io/badge/Functions-500+-orange.svg)]()
[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC_BY--NC--SA_4.0-red.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)
[![Status](https://img.shields.io/badge/Status-Archived_(已归档)-lightgrey.svg)]()

[🌐 English](#-english-overview-for-international-creators) • [使用须知](#-使用须知) • [项目背景](#-项目背景) • [项目特性](#-项目特性) • [上手引导](#-项目上手引导) • [授权说明](#-授权说明)

</div>

---

> [!WARNING]
> **项目归档说明 (Archived / AS-IS)**
> 本项目是一个搁置很久的个人半成品弃坑项目，现以现状（AS-IS）归档开源。
> **这不是一套打磨完善、开箱即用的成熟框架**，内部存在遗留问题、技术债与潜在 Bug。作者**不再提供任何后续更新、新版本适配、Bug 修复或技术答疑**。期待“即拿即用”或零基础的朋友请谨慎下载。

---

## 🌐 English Overview (For International Creators)

**Spirit Journey (魂旅)** is an archived, vanilla Minecraft CTM-RPG game framework built entirely with Datapacks for **Minecraft 1.20.4** (`pack_format 17`, heavily utilizing the macro system introduced in 23w33a). 

* **Scale:** 500+ `.mcfunction` files organized into 13 independent modules, accompanied by module documentation, a functional test world, and basic tutorials.
* **Status:** **Archived / Abandoned (AS-IS).** Originally developed over a 3-year span for an unreleased CTM map. Expect technical debt, legacy code, and potential bugs.

### ⚠️ Important Notice & Target Audience

1. **NOT for Beginners (Not Out-of-the-Box):**
   This is an unfinished personal project that has not undergone comprehensive stability testing. Many interfaces are not universally polished, and using it directly to build a map without modification will likely result in bugs. **If you do not have the ability to read, modify, and debug datapacks, we do not recommend downloading this.**
2. **Who is this for? (What can you do with it?)**
   * **Experienced CTM-RPG Map Makers:** You can use this as an underlying foundation to modify and reuse, saving you the trouble of building an RPG system from scratch. Alternatively, you can reference specific modules and **use AI tools (e.g., ChatGPT, Claude) to translate and migrate them into your own datapack**.
   * **Datapack Learners & Researchers:** Ideal for studying how vanilla datapacks can implement RPG skill systems, Boss mechanics, and CTM progression workflows.
3. **Language Note (How to read the docs):**
   All documentation (`docs/`), in-code comments, and the test world are written in **Chinese**. However, since Minecraft command syntax is universal, **feeding the markdown docs and `.mcfunction` files into an AI assistant** will easily bridge the language gap and help you extract the logic.
4. **No Maintenance or Support:**
   This project is permanently archived. The author will **not** fix bugs, update to newer versions, accept feature requests, or provide technical Q&A. You are on your own for troubleshooting—downloading means you acknowledge and accept this.
5. **Version Compatibility:**
   Developed and tested strictly on **Minecraft 1.20.4**. You may attempt to port it to higher versions on your own, but full functionality is not guaranteed.

### 📜 License (CC BY-NC-SA 4.0)
* **Strictly Non-Commercial:** Any form of commercial use (paid maps, paid modpacks, P2W/monetized servers, paid tutorials, etc.) is **strictly prohibited**.
* You are free to modify, redistribute, and publish maps based on this framework for non-commercial purposes without asking for permission—just **credit the original author** and distribute your derivative works under the **same license (CC BY-NC-SA 4.0)**.

---

## ⚠️ 使用须知

1. **不适合零基础直接上手：** 这是个人半成品弃坑项目，没有做全面的稳定性测试，很多接口通用性一般，直接拿来做地图很容易遇到各类 bug。如果你不具备数据包阅读、修改、排错的能力，不建议下载使用。
2. **适合的人群：**
   - **有数据包基础，准备制作 CTM-RPG 地图的制作者**：可以作为底层基础修改复用，省去从零搭建 RPG 系统的工作量，或者参考数据包中的模块，使用 AI 迁移到你的数据包里；
   - **学习向玩家**：用来研究原版数据包实现 RPG 技能、BOSS 机制、CTM 流程的思路，参考代码逻辑。
3. **维护相关：** 项目已经归档弃坑，我不会修复 bug、适配新版本，也不接收功能需求、解答相关技术问题。使用中遇到的所有问题，都需要使用者自行修改排查，下载即代表知晓并接受这点。
4. **版本兼容提醒：** 开发与测试环境仅为 `1.20.4`。更高版本可以自行尝试迁移，但不保证全部功能正常，适配工作需要使用者自己完成。

---

## 📖 项目背景

最早在 Minecraft `23w33a` 快照更新**宏（Macro）**功能时，我萌生了利用该特性搭建一套纯原版 CTM 开发框架的想法，初衷是为自己的 CTM 地图配套开发。整个开发跨度很长，中间有一段长时间停笔，代码里混杂了不同阶段的写法，积攒了不少技术债，早期部分写法也没有统一规范。

最后地图本体几乎没有推进，连完整的一关都没能做出来；但是底层数据包框架反倒写了不少，技能系统这类核心 RPG 内容都有落地。

项目起笔至今已有三年，中间两次长期搁置。不少早期的设计规范时隔太久我已经记不清，存在一定技术债。但模块划分本身比较明确。这次整理归档我也重新啃了一遍自己的旧代码，文档无法保证 100% 准确，如有纰漏还望谅解。

因为个人原因，原地图项目决定不再继续制作，索性把这套底层框架整理开源，供大家参考学习，或给有能力的创作者拿去二次开发。主要为了归档和纪念而开源，未来不再做大规模维护。

---

## ✨ 项目特性

- **原版实现**：无任何 Mod 依赖，深度结合 `1.20.4` 原版特性与函数宏（Macro）。
- **体量**：包含约 500 个 `.mcfunction` 文件，划分为 13 个相对独立的功能模块。
- **配套**：附带**各模块详细文档、功能测试地图、基础使用教学**。
- **AI 协同开发工作流与经验分享**：考虑到弃坑项目无人答疑、近 500 个函数直接阅读成本极高，文档中专门附带了作者后期开发时使用的 **Prompt（提示词）参考与 AI 协同工作流引导**。即使不逐行啃底层指令，也能借助 AI 快速理解范式、扩展新技能或将模块迁移至你的项目中。其实主要是分享一些我在开发这个数据包时用的一些方法经验。

---

## 🧩 项目上手引导

首先，导入**教学测试用存档**，用这个存档可以快速查看该数据包的一些具体实现效果。

这边刚开始，首先先了解几个核心函数：

1. 第一个先看 **`scoreboard_creator`** 吧，这个是整个项目的计分板创建统计函数。
2. 然后第二个看 **`get`** 函数和 **`get_temporary`** 函数：
   - `get` 函数是整个地图的数据库，地图里的所有物品都会写在这里。
   - `get_temporary` 函数是一个中转，它的作用是把 `get` 函数里的指令复制到这里，然后进游戏调用 `get_temporary` 来获取所需要的物品。

因为这个项目函数还是蛮多的，有 13 个模块，所以我把文档也做了 13 份分开来介绍，不然的话太杂太多了。
13 个文档中，数据包条目里的原始客观信息（调用链路、记分板字段、API 参数等）由 AI 辅助整理生成，而使用方式、模块概述、设计思路以及工作流内容为人工编撰。---> *(PS: 结果发现好像我也没写多少 hh，更多是当时我做的一些想法吧。你基本上能看到比较口语的话，那就都是我写的。毕竟我第一次搞文档，我也不晓得咋写)*

因为写完文档之后，我转移存档去做测试地图的时候，转移这个数据包时遇到了一些 bug，里面有可能有些文件结构和函数是不太对的（旧的），但绝大部分应该都是没问题的。（搞这个花了我太多时间，不想改了，抱歉 555）

文档我放在 **`docs/`** 文件夹里了，根据推荐的学习顺序，分成了 `Stage 1`、`Stage 2`、`Stage 3`、`Stage 4`：

- **`Stage 1`**：主要是一些底层的 API，扫一眼看看就行了，没有具体的实现。
- **`Stage 2`**：主要包含了底层战斗系统，还有敌人技能的调用 API 这两个，因为 Stage 3 会用到它们，所以我把它放到了 Stage 2。
- **`Stage 3`**：是这个数据包的核心业务层，它包括了很多具体的复杂实现，比如技能系统、Boss 系统等等。
- **`Stage 4`**：就是一个额外的层，比较独立，可以直接看。主要是一些 CTM 地图中比较杂的系统（比如对话呀、物品展示这些的）。

### 🛠️ 测试存档已知遗留问题
目前测试存档已知的还有一些问题我也没有处理。毕竟太麻烦了，真的花了我太多时间了。这件事，我就把遇到的一些事列在这里吧：
1. 首先，技能**“万剑归宗”**没有在刺剑的攻击那里注册，但是单手剑的攻击是没有问题的。
2. **`y_other`** 这套传送系统其实是可以用的，它可以做类似传送点的机制，你应该用 AI 修一下就可以弄好。我这边没有时间，就没有弄了 555。
3. 目前好像就是绿色的对话点的消除,会影响周围红色的对话点(好像会把附近的红色对话给删掉)
4. 游荡灯笼的文本描述有误,正确的描述应该是：灯笼会像一个炮塔一样静止,而不是跟随玩家.最大锁定单位是两个,而不是3个.持续时间的文本好像也有错误.
5. 哦对了,还有,很多技能物品的文本描述都有问题.因为地图是没有做出来的,很多我是先在程序上先行,但是文本描述我没改,所以会导致很多技能效果和文本描述有一定偏差.具体的技能效果时间还是以代码为准.

---

## 💭 碎碎念

原先以为开源只是点个 `+`，上传 GitHub 就完事了，真正着手整理代码、文档与测试存档，足足花费十余天（累死我了），配套工作远比开发本身还累。这是我的首次开源，相关规范和交付方式都在摸索中 ing，仅希望做到有始有终、尽力而为，疏漏之处……emm，我好像说了都归档了，那如果有疏漏之处就不要来找我了 (doge)。

关于地图本身，虽然叫魂旅 (Spirit Journey)，但其实地图风格和场景不是想走魂系 (Soul) 的暗黑风格的，当时的想法是做比较明亮有活力的基调，比如以植物、绿色这些为基调，但是难度依旧是魂游的难度，当然我做的话可能会宽容一些 hh~（所以取名是 Spirit 而不是 Soul）。

其实我也知道，这种面向小众开发者的框架，CTM 的开发作者本来就是极少数人，而且他们大多数有自己的框架，我这个框架基本上是没什么人会用的。我就是主要因为这个框架确实写了很久，就不想把自己的一些心血烂在自己电脑里吧，就当漂流瓶发出去吧。
生活总有遗憾，虽然地图没有做完，但是把数据包开源出来也算是让自己三年前的想法焕发第二春了~

---

## 📜 授权说明

本项目采用 **[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)** 协议开源：

- 允许在**非商业**前提下修改、二次分发、基于本框架制作并发布地图，发布时保留原作者署名；
- **严禁一切形式的商业化使用**，包括付费地图、付费整合包、氪金服务器、付费教学等；
- 基于本项目修改后对外发布的衍生作品，必须沿用相同协议（CC BY-NC-SA 4.0）；
- 非商业的使用，可以不用联系我，在帖子或作品发布时保留原作者署名即可。
