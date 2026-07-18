# AI写代码越写越笨？掌握上下文卫生，3步解决Claude降智难题

> **作者/UP主**: [AI全文总结](https://space.bilibili.com/91394217) (UID 91394217)
> **BV号**: [BV1pRFSzfE4y](https://www.bilibili.com/video/BV1pRFSzfE4y/)
> **栏目定位**: DEEP DIVE | AI ENGINEERING
> **关联公众号**: AI知识工具 / AI代码工具
> **本文档来源**: 基于视频元数据 + 视频抽帧（约 30 帧）逐帧整理

---

## 目录

1. [核心问题：AI 为何越写越笨](#1-核心问题ai-为何越写越笨)
2. [元凶诊断：上下文噪音 Context Noise](#2-元凶诊断上下文噪音-context-noise)
3. [反模式：大杂烩 The Monolith](#3-反模式大杂烩-the-monolith)
4. [分级加载机制：Claude Code 的 Context Layer](#4-分级加载机制claude-code-的-context-layer)
5. [三步落地法：Context Hygiene 实操](#5-三步落地法context-hygiene-实操)
6. [效果验证：Context Horizon](#6-效果验证context-horizon)
7. [未来趋势：Context Hygiene 取代 Prompt Engineering](#7-未来趋势context-hygiene-取代-prompt-engineering)
8. [相关链接与标签](#8-相关链接与标签)

---

## 1. 核心问题：AI 为何越写越笨

### 1.1 现象：从 GENIUS 到 DUMB ZONE

> **你正在经历的"变笨"曲线**

| 阶段 | Phase 01: Project Start | Phase 02: Mid-Term |
|------|-------------------------|---------------------|
| 标签 | **GENIUS** 天才级表现 | **THE DUMB ZONE** "笨蛋区"陷阱 |
| 行为 | 逻辑严密：代码生成一气呵成，几乎不需要修改，完全照需求意图 | 忽略规范：开始破坏既有逻辑，修改 A 文件导致 B 文件崩溃，照单全收 |
|      | 响应迅速：准确执行既定规范，如同经验丰富的资深工程师 | 产生幻觉：无中生有地引用不存在的库或方法，逻辑链断裂 |

### 1.2 资深 AI 开发者 Michael Guo 的诊断

> "这并不是模型本身出现了波动，而是随着上下文的膨胀，你亲手将 AI 送入了一个信息过载的盲区。"
> —— **Michael Guo, Senior AI Developer**

**旁白还原**：
- 资深AI开发者Michael Guo早就指出了真相：不是AI脑子坏了，而是你亲手把它送进了"笨蛋区"。
- 但只要文件一多、对话一长，它就开始出现"痴呆"症状：要么忽略你写好的规范，要么改A文件坏B文件，甚至开始胡编乱造。

---

## 2. 元凶诊断：上下文噪音 Context Noise

### 2.1 关键拐点 CRITICAL THRESHOLD

> **>50% 推理能力拐点**（Context Window 使用率）

研究曲线（EU reasoning score vs Context Window 使用率）显示：

```
推理能力
  ↑
100├──────────────╮
  │              │  
  │   高智商区间  │   降智区间
  │   (稳定100%)  │   (指数下滑)
  │              │  ╲
  │              │   ╲
  │              │    ╲
  0└──────────────┴─────╲─────→ 上下文使用率
   0%      50%        100%
```

### 2.2 50% 阈值的两面

| 使用率区间 | 表现 |
|------------|------|
| **0% – 50%** 使用率 | 推理能力稳定，AI 能够精准定位关键信息，逻辑链条完整，处于"高智商区间" |
| **50% – 100%** 使用率 | 指数级下滑，由于计算注意力被噪音分散，AI 出现幻觉、逻辑断裂，进入"降智区间" |

> 不是AI脑容量不够，是冗余信息形成的"背景噪音"烧干了推理核心。

### 2.3 反直觉的事实

> 一旦你给AI塞的上下文超过了窗口容量的50%，它的推理能力不是线性下降，而是断崖式崩塌。

---

## 3. 反模式：大杂烩 The Monolith

### 3.1 你的做法

**`CLAUDE.md` 文件膨胀到 3,856+ LINES**，把以下 5 个层级的规则全部塞在根目录：

| Section | 主题 |
|---------|------|
| Section 01 | 数据库性能与 SQL 索引优化原则… |
| Section 02 | React 组件生命周期与 Tailwind 规范… |
| Section 03 | AWS S3 存储桶与 CloudFront 加速策略… |
| Section 04 | 手写循环代替函数与 Design Tokens… |
| Section 05 | RESTful API 命名规则与错误响应定义… |

### 3.2 核心问题：认知过载 CognitiveOverload

当 AI 在写 CSS 样式时，它的上下文窗口里还塞满了 SQL 索引和 I/O 屏幕脚本。这会导致 AI **"精神分裂"**，从而产生严重的跑题表现。

**痛点分析**：
- 推理能力被无关干扰项稀释
- 有限的 Token 窗口被垃圾信息抢占

### 3.3 经典类比

> 这就好比要求一名刚入职的程序员，在写下第一行代码前，必须先把公司过去十年的会议记录全部背下来。
>
> 他在写前端组件的时候，脑子里塞满了数据库的连接配置，这能不精神分裂吗？

---

## 4. 分级加载机制：Claude Code 的 Context Layer

Claude Code 采用**双层加载机制**来对抗大杂烩问题。

### 4.1 机制一：祖先级 Ancestor（强制加载）

| 属性 | 内容 |
|------|------|
| 加载位置 | 根目录 RootContext：`./CLAUDE.md` 或 `./.cursorrules` |
| 加载时机 | **AI 启动瞬间 OnStartup** —— 当对话建立的那一刻，规则即刻生效，无需任何手动唤醒 |
| 生命周期 | **永久常驻 AlwaysActive** —— 它是上下文的基石，贯穿对话始终，不会因为多轮交流而被遗忘 |
| 资源代价 | 挤占宝贵的 Context Window；每一轮对话都会被反复计算 |
| 结论 | 根目录的每一个字，都是这一局对话的"出厂设置" |

> 首先是"祖级"规则，也就是你放在项目根目录下的那个规则文件。它的机制是"强制加载"。

### 4.2 机制二：后代级 Descendant（按需加载）

| 属性 | 内容 |
|------|------|
| 加载位置 | 子目录，例如 `backend/CLAUDE.md` / `frontend/CLAUDE.md` |
| 加载时机 | **按需加载 OnDemand** —— AI 进入相关模块时才按需展开 |
| 干扰控制 | AI 如果遇到不相关的代码块就会跳过，**不会让"上下文变多"** |
| 格式化 | 高度格式化，**不需要花费额外的 Token 去精读**，能减少一整轮的隐形费用 |
| 经典心法 | **"看不见，心不烦。"** |

> 只有当你令 AI 去读取这个特定文件夹的时候，它才会去读里面的规则。
> 这种机制能避免在后端写代码的时候，脑子里时不时出现前端的 CSS 样式干扰。

### 4.3 两层机制对比

| 维度 | Ancestor（祖级） | Descendant（后代级） |
|------|------------------|----------------------|
| 类比 | 底层操作系统 | 应用程序 |
| 时机 | OnStartup | OnDemand |
| 范围 | 全局 | 子目录 |
| 开销 | 每一轮都付账 | 用到才付账 |
| 适用 | 强约束规范 | 业务专属规则 |

---

## 5. 三步落地法：Context Hygiene 实操

### 5.1 STEP 01：减肥 —— 根目录的绝对全局铁律

**`Root Context / CLAUDE.md` / Step 01：绝对全局铁律 The Non-Negotiables of the Project**

只放 4 个不可协商项：

| 类别 | 设定 |
|------|------|
| Language | **TypeScript** |
| Indentation | **2 Spaces** |
| Commit Style | **Conventional** |
| Code Style | **Next.js App** |

**严守红线** —— 任何配置都应过项架构总管：`Stack, Repo, Hosts, Global, Drome, Comesthe…`

**严禁出现**：
- No React Components / Public Hooks
- No Prisma Query Details
- No Docker Config Logs

> 核心口号：**KEEP CONTEXT ABSOLUTELY CLEAN**（保持上下文绝对干净）

> 用最新的 Claude.md 空文件，把里面允许的语言全部删死。
> 凡是涉及到数据库或者后端的细节，诸如 React 怎么写、数据库怎么连，统统不放在这儿。

### 5.2 STEP 02：下放 —— 后端目录 Backend 局部注入

**`backend/CLAUDE.md`**

| 主题 | 规则 |
|------|------|
| **数据库** | 必须使用 Prisma Client 进行所有数据库交互（Resort 写法） |
| **错误处理** | 所有 API 逻辑必须包含 `try-catch` 块，统一返回符合规范的 `ErrorResponse` 对象 |
| **身份认证** | 一律集成 Clerk SDK 处理用户身份，所有受保护路由运行 Auth Middleware |

> 第二步：把入后端目录。
> 体现名：只要进入这个文件夹，你就必须遵守后端法则。
> 所有的 API 接口必须用 `try-catch` 结构包裹，并且返回标准的错误响应。

### 5.3 STEP 03：监控 —— 前端组件 Components 视觉规范

**`components/CLAUDE.md`**

| 主题 | 设定 |
|------|------|
| **STYLING 样式** | 原子化 CSS —— 仅使用 Tailwind Utility Classes（严禁 CSS Modules） |
| **ICONS 图标** | 一致性库 —— 仅使用 Lucide-React 库 |
| **MOTION 状态** | 动效绑定 —— 整合动效绑定 Framer Motion |

> 物理隔离必要逻辑，专注 UI 渲染。
> 第三步：给前端组件库同感。
> 在提到文件夹里清空，锁定并限用 Tailwind 的原子类，严禁写 CSS Modules。

---

## 6. 效果验证：Context Horizon

> **自分形势构验证：Context Horizon**（硬核案例：四层架构 + Layer 与"脚手架"设计）

### 6.1 AI 实际可见的上下文 vs 噪音

| AI 实际上下文 `ACTUALCONTEXT` | AI 看到的噪音 `HOARYZON` |
|-------------------------------|--------------------------|
| **Global Rules**: TypeScript, @typescript-eslint | Boarded / Projects: from OSS |
| **Local Frontend Rules**: Tailwind CSS, Linte Xxxs, Farner Modul | Fouse CSS / PagerCSS / SQL |
|                                          | from – Hirrve: Lind Frontend |

### 6.2 指标对比

| 指标 | 实施前 | 实施后 |
|------|--------|--------|
| **Token 饱和率** | 100% | — |
| **幻觉率 (HALLUCINATION RATE)** | 高 | 接近 0% |

> 最关键的是，AI 列此别（隔离）。它的脑子里很多不知道数据库的存在，也不知道后端怎么处理报错。
> 做这一招，效果是立竿见影的。

---

## 7. 未来趋势：Context Hygiene 取代 Prompt Engineering

### 7.1 落地三步曲 Action Plan（总结）

> 🎉 三个核心动作，立刻解放你的 AI 工作流效率

| 步骤 | 动作 | 关键操作 | 方法论 |
|------|------|----------|--------|
| **STEP 01** | 减肥 | 清空祖目录的少量内容 | Focus on `CLAUDE.md` —— 保留最核心的规则 |
| **STEP 02** | 下放 | 特定规则移入子目录 | Localized Context —— 上传/下载时按需加载 |
| **STEP 03** | 监控 | 实时监控 Token 消耗 | Prevent Context Bloat —— 负载时整个使用 % Pixel Red |

> 第一，减肥：立刻检查你的项目，按那些最令人精的存的配要不要全列，只留最核心的语言规范。
> 第二，下放：把数据库与前端的设计后端文件夹，岳以现到前后端分类文件夹，让它们各司其职，井水不犯河水。
> 这就可让AI总是出现精菱不方，强行低核发么会产生出低迷状况。

### 7.2 深度思考：未来的开发者之争

> **Prompt Engineering → Context Hygiene**

未来的竞争点从"如何问"转向"如何喂"。

**定义：上下文卫生 Context Hygiene**

> 未来的开发者之争，不再是措干措技，而是看谁能把"废话、服务"的开销一样，精准地"喂给 AI"上下文。

**角色跃迁：从 Code 灵感到架构师**

> 掌握级行/句调规，比不再像是一个写代码代码的人，而是一个驾驭 AI 大脑的架构师，剑儿在 AI 手里。

> "市场总需求不变，但 AI 让从众变高了，那到最后就是谁能完成提效。"
> —— @howetheworld

**核心金句**：

> **掌握 Context Hygiene：不做工具的搬运工，做 AI 的指令架构师。**

### 7.3 终局视角

> 最后，送大家一个新的概念：**Context Hygiene，上下文卫生**。
> 在应用发尾定会，AI 工具不再是一个特殊的代的人，而是一个掌控AI大脑的架构师。
