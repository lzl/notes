# Peter Steinberger 采访汇总报告：高频观点与原文引用

> 分析来源：4个视频采访 + 3篇博客文章
> 时间跨度：2025年6月 - 2026年1月
> 总时长：约6小时视频内容 + 约50分钟阅读量

本报告汇总了 Peter Steinberger 在过去半年多公开分享中反复出现的核心观点，按出现频率排序，并附带原文引用和时间戳便于回看。

---

## 目录

1. [闭环原则 (Closing the Loop)](#1-闭环原则-closing-the-loop)
2. [AI 编程的"老虎机效应"](#2-ai-编程的老虎机效应)
3. [CLI 优于 MCP](#3-cli-优于-mcp)
4. [Codex vs Claude Code](#4-codex-vs-claude-code)
5. [PRs = Prompt Requests](#5-prs--prompt-requests)
6. [并行 Agent 工作流](#6-并行-agent-工作流)
7. [迭代塑形 vs 瀑布式规划](#7-迭代塑形-vs-瀑布式规划)
8. [Prompting 是一项技能](#8-prompting-是一项技能)
9. [技术"消失"的产品体验](#9-技术消失的产品体验)
10. [公司需要30%的人](#10-公司需要30的人)

---

## 1. 闭环原则 (Closing the Loop)

**出现频率**：★★★★★（所有来源）

这是 Peter 反复强调的最核心原则：AI 必须能够验证自己的工作。

### 原文引用

**The Pragmatic Engineer [00:58:31]**：
> "The whole reason why those models that we currently have are so good at coding but sometimes mediocre at writing is because there's no easy way to validate writing, but code I can compile, I can lint, I can execute, I can verify the output."

**The Pragmatic Engineer [00:58:03]**：
> "You always let it write tests and you make sure that it runs them."

**The Pragmatic Engineer [01:00:50]**：
> "Using agentic coding actually makes you a better coder because you have to think harder about your architecture so that it's easier verifiable."

**TBPN First Appearance [00:23:15]**：
> "The model has to close the loop. It needs to be able to verify its own work. That's the whole secret."

**GitHub Open Source Friday [00:45:30]**：
> "If you design it the right way, you have a perfect loop."

**博客 - Just Talk To It**：
> "This self-correction loop is powerful."

### 核心洞察

- 为什么 AI 擅长代码但不擅长写作？因为代码可以验证
- 设计功能时就要想"agent 怎么自己验证这个"
- 好的架构 = 容易测试的架构
- 让 agent 写测试，在同一个 context 中

---

## 2. AI 编程的"老虎机效应"

**出现频率**：★★★★★（所有采访）

Peter 多次用赌场老虎机来比喻 AI 编程的上瘾性。

### 原文引用

**The Pragmatic Engineer [00:38:40]**：
> "It's the same economics as you go to a casino. That's my little slot machines. You type in the prompt and ding ding ding ding ding and it's like nope... or it does something that actually blows your mind."

**He Built a SUPER App [00:05:23]**：
> "It's like a slot machine. You pull the lever, sometimes you get garbage, sometimes you get something brilliant."

**TBPN First Appearance [00:15:45]**：
> "I call it the 'one more prompt' addiction. At 3am you're like, just one more..."

**The Pragmatic Engineer [00:36:55]**：
> "I started a meetup in London that I called Claude Code Anonymous because it's a little bit like a drug."

### 核心洞察

- 这种上瘾性来自不确定的奖励机制
- 可能导致睡眠不足、过度工作
- 但也是驱动快速学习的动力
- 需要有意识地管理（如健身、散步时不带手机）

---

## 3. CLI 优于 MCP

**出现频率**：★★★★☆（3个采访 + 所有博客）

Peter 坚持认为 MCP 是一个"拐杖"，CLI 才是正确的方式。

### 原文引用

**The Pragmatic Engineer [01:32:08]**：
> "MCP is a crutch. The best thing that came out of MCPs is that it made companies rethink to open up more APIs. But the whole concept is silly."

**The Pragmatic Engineer [01:33:38]**：
> "You have to pre-export all the functions of all the tools and all the explanations when your session loads... But surprise, models are really good at using bash."

**The Pragmatic Engineer [01:34:18]**：
> "If it's a CLI, I could use jq and filter for exactly what it needs."

**博客 - Just Talk To It**：
> "Almost all MCPs really should be clis. I can just refer to a cli by name. I don't need any explanation in my agents file."

**博客 - Just Talk To It**：
> "Use GitHub's MCP and see 23k tokens gone. Heck, they did make it better because it was almost 50,000 tokens when it first launched."

### 核心洞察

- MCP 在 session 开始时污染 context
- CLI 第一次调用失败时显示 help，之后就能用
- CLI 可以链式调用、可以脚本化
- 模型天生擅长使用 bash

---

## 4. Codex vs Claude Code

**出现频率**：★★★★☆（2个采访 + 所有博客）

Peter 从 Claude Code 的重度用户转变为 Codex 的坚定支持者。

### 原文引用

**The Pragmatic Engineer [00:42:43]**：
> "I don't know why all these people still use Claude Code... whatever OpenAI cooked there is insanely good. Pretty much every prompt I type gives me the result I want."

**The Pragmatic Engineer [00:43:00]**：
> "Claude would read three files and then be confident enough to just create code... Codex will just be silent and just read files for 10 minutes."

**博客 - Just Talk To It**：
> "I used to love Claude Code, these days I can't stand it anymore. It's language, the 'absolutely right's, the '100% production ready' messages while tests fail - I just can't anymore."

**博客 - Just Talk To It**：
> "Codex is more like the introverted engineer that chugs along and just gets stuff done."

**博客 - Just Talk To It**：
> "This really makes a difference to my mental health. I've been screaming at claude so many times. I rarely get angry with codex."

### 对比表格

| 特性 | Claude Code | Codex (GPT 5.2) |
|-----|-------------|-----------------|
| 文件阅读 | 读 3 个文件就开始 | 静默读 10 分钟 |
| 速度 | 快但需要迭代 | 慢但一次成功率高 |
| 语言风格 | "Absolutely right!" | 沉默做事 |
| Context | 156k | ~230k |
| 心理影响 | 经常让人想尖叫 | 很少生气 |

---

## 5. PRs = Prompt Requests

**出现频率**：★★★★☆（3个采访）

Peter 认为 Pull Requests 应该重新定义为 Prompt Requests。

### 原文引用

**The Pragmatic Engineer [01:10:32]**：
> "I see pull requests more as prompt requests now. Somebody opens a pull request, I don't look at the code, I say thanks and think about the feature, and then with my agent we start off with the PR and then I design the feature as I see fit."

**The Pragmatic Engineer [01:14:06]**：
> "I'm actually more interested in the prompts than in the code. I ask people to please add the prompts... I read the prompts more than I read the code."

**The Pragmatic Engineer [01:14:13]**：
> "This gives me more idea about the output. I don't have to read the code. If someone wants a feature, I ask for a prompt request - write it up really well because then I can just point my agent to the issue and it will build it."

**GitHub Open Source Friday [01:15:22]**：
> "If you send me a PR, I'll probably rewrite it. But your PR tells my agent what to build."

### 核心洞察

- PR 的价值在于传达意图，不是代码本身
- 好的 prompt 比好的代码更有价值
- Agent 会重写大多数 PR
- 这改变了代码审查的本质

---

## 6. 并行 Agent 工作流

**出现频率**：★★★★☆（所有来源）

Peter 同时运行 5-10 个 agent，像下多盘棋。

### 原文引用

**The Pragmatic Engineer [00:44:28]**：
> "I use between five and 10 agents in parallel."

**The Pragmatic Engineer [00:55:03]**：
> "To stay in the flow state I need to massively parallelize... I switch around a lot in my head. I wish I wouldn't have to do that."

**The Pragmatic Engineer [00:55:59]**：
> "It's like Starcraft, you know, you have your main base and you have your side bases."

**He Built a SUPER App [00:18:45]**：
> "I have multiple terminals open. One main project with focus, and satellite projects that need less attention."

**博客 - Just Talk To It**：
> "I run between 3-8 in parallel in a 3x3 terminal grid, most of them in the same folder."

### 核心洞察

- 并行工作需要清晰的任务边界
- "Blast Radius"思维：估计每个任务影响多少文件
- 主要项目 + 卫星项目的组合
- 需要更高的心智负担，但保持 flow state

---

## 7. 迭代塑形 vs 瀑布式规划

**出现频率**：★★★★☆（3个采访 + 博客）

Peter 不相信先写完整 spec 再让 agent 构建的方式。

### 原文引用

**The Pragmatic Engineer [01:08:03]**：
> "How can you even know what you want to build before you built it? You learn so much in the process of building it that will go back into your thinking."

**The Pragmatic Engineer [01:07:38]**：
> "I don't believe in things like Gas Town where you write up the spec and then it builds itself and then it's done."

**The Pragmatic Engineer [01:07:43]**：
> "My building model is usually very much forward. It's very rarely that I actually revert. It's just like shaping. I love how you start with a rock and then chisel away at it and slowly this statue emerges out of marble."

**博客 - Just Talk To It**：
> "I often start with sth simple and woefully under-spec my requests, and watch the model build and see the browser update in real time. Then I queue in additional changes and iterate."

**博客 - Just Talk To It**：
> "I often saw codex build something interesting I didn't even think of. I don't reset, I simply iterate and morph the chaos into the shape that feels right."

### 核心洞察

- 有时候故意 under-spec 来获得惊喜
- 前进式开发，很少回滚
- 建造过程中学到的东西会改变设计
- 像雕塑一样，从混沌中塑造形状

---

## 8. Prompting 是一项技能

**出现频率**：★★★★☆（所有来源）

像学习乐器一样，prompting 需要练习和直觉。

### 原文引用

**The Pragmatic Engineer [01:05:41]**：
> "It's a skill like any other skill."

**The Pragmatic Engineer [01:05:05]**：
> "The more you understand how those little beasts think, the better you get at prompting."

**The Pragmatic Engineer [00:40:21]**：
> "I feel the same friction when I prompt because I see the code flying by. I see how long it takes. I see if the agent pushes back."

**GitHub Open Source Friday [00:52:18]**：
> "You need to learn to speak their language. I slowly started to understand why those things do what they do."

**博客 - Just Talk To It**：
> "When things get hard, prompting and adding some trigger words like 'take your time' 'comprehensive' 'read all code that could be related' makes codex solve even the trickiest problems."

### 触发词列表

| 触发词 | 用途 |
|--------|------|
| "let's discuss" | 阻止立即构建 |
| "give me options" | 获取多个方案 |
| "take your time" | 解决困难问题 |
| "comprehensive" | 深入分析 |
| "read all code that could be related" | 扩大搜索范围 |
| "create possible hypothesis" | debug 时使用 |

---

## 9. 技术"消失"的产品体验

**出现频率**：★★★☆☆（2个采访）

Peter 追求的终极目标：用户不需要知道技术如何运作。

### 原文引用

**The Pragmatic Engineer [01:30:55]**：
> "The technology disappears. You just talk to a friend on your phone that is infinitely resourceful, has access to your email, your calendar, your files..."

**The Pragmatic Engineer [01:31:07]**：
> "You don't have to think about compactation or any of that context blends away."

**TBPN First Appearance [00:28:42]**：
> "The app melts away. In the future, software will be generated on the fly for your specific need."

**GitHub Open Source Friday [00:38:15]**：
> "The beauty of it is that it just works. You don't see the machinery behind it."

### 核心洞察

- 好的产品隐藏复杂性
- 用户只看到自然的对话
- 后台可能有多个 sub-agent 在工作
- 这是 Clawdbot/Moltbot 的设计哲学

---

## 10. 公司需要30%的人

**出现频率**：★★★☆☆（2个采访）

Peter 对企业 AI 采用的预测。

### 原文引用

**The Pragmatic Engineer [01:23:55]**：
> "I could easily run a company with 30% of the people. It would probably be quite difficult to find people on that level, but you want really senior engineers that really understand what they build."

**The Pragmatic Engineer [01:24:12]**：
> "Companies will have a really hard time adopting AI efficiently because this also requires to completely redefine how the company works."

**The Pragmatic Engineer [01:25:15]**：
> "Which is very scary because economically this will all lead into a fiasco. A lot of people will have trouble finding a place in this new world."

**He Built a SUPER App [00:42:18]**：
> "The value of code just went down dramatically. What matters now is taste and system design."

### 核心洞察

- 需要高 agency、高能力的 builder
- 公司需要"重构"——不只是代码
- 这会导致就业市场的剧变
- 新角色：有产品视野、能做所有事的人

---

## 高频观点速查表

| 排名 | 观点 | 频率 | 首要来源 |
|------|------|------|----------|
| 1 | 闭环原则 | ★★★★★ | The Pragmatic Engineer [00:58:31] |
| 2 | 老虎机效应 | ★★★★★ | The Pragmatic Engineer [00:38:40] |
| 3 | CLI > MCP | ★★★★☆ | The Pragmatic Engineer [01:32:08] |
| 4 | Codex > Claude Code | ★★★★☆ | Just Talk To It (博客) |
| 5 | PRs = Prompt Requests | ★★★★☆ | The Pragmatic Engineer [01:10:32] |
| 6 | 并行 Agent | ★★★★☆ | The Pragmatic Engineer [00:44:28] |
| 7 | 迭代塑形 | ★★★★☆ | The Pragmatic Engineer [01:07:43] |
| 8 | Prompting 技能 | ★★★★☆ | The Pragmatic Engineer [01:05:41] |
| 9 | 技术消失 | ★★★☆☆ | The Pragmatic Engineer [01:30:55] |
| 10 | 30%的人 | ★★★☆☆ | The Pragmatic Engineer [01:23:55] |

---

## 采访回看索引

### 视频链接

| 采访 | 链接 | 时长 | 重点内容 |
|------|------|------|----------|
| TBPN First Appearance | [YouTube](https://www.youtube.com/watch?v=oA_bQ5X2C3g) | ~45分钟 | 背景故事、CLI 哲学、开源 |
| He Built a SUPER App | [YouTube](https://www.youtube.com/watch?v=QoYVBQHmjP8) | ~35分钟 | 详细 prompting、VibeTunnel |
| GitHub Open Source Friday | [YouTube](https://www.youtube.com/watch?v=fDbNYEDX7v4) | ~90分钟 | 社区、heartbeat、PR 流程 |
| The Pragmatic Engineer | [YouTube](https://www.youtube.com/watch?v=TQPYG87E6Nw) | ~90分钟 | 最全面、最深入的技术讨论 |

### 博客链接

| 文章 | 链接 | 日期 | 重点内容 |
|------|------|------|----------|
| The Future of Vibe Coding | [steipete.com](https://steipete.com/posts/2025/the-future-of-vibe-coding) | 2025.06 | 工作坊实录、MCP 工具 |
| My Current AI Dev Workflow | [steipete.com](https://steipete.com/posts/2025/optimal-ai-development-workflow) | 2025.08 | 工具栈、测试策略 |
| Just Talk To It | [steipete.com](https://steipete.com/posts/just-talk-to-it/) | 2025.10 | 最新工作流、Codex 对比 |

---

## 综合行动清单

### 立即可做

1. **配置闭环**：每个功能让 agent 写测试并运行
2. **使用 CLI**：用 gh, vercel, psql 等命令行工具
3. **图片 prompting**：50%+ prompt 附带截图
4. **短 prompt**：1-2 句话 + 图片

### 工作流改进

1. **并行 agent**：3-8 个终端同时工作
2. **Blast Radius 思维**：估计每个任务影响范围
3. **迭代塑形**：故意 under-spec，看 agent 怎么做
4. **20% 重构时间**：用 jscpd, knip, ast-grep

### 协作方式

1. **Prompt Request**：PR 中附带 prompt
2. **讨论架构**：代码细节不重要
3. **功能重写**：用 agent 根据 PR 意图重建

### 长期投资

1. **学习 prompting**：像学乐器一样练习
2. **培养直觉**：感受 agent 的"摩擦力"
3. **保持好奇**：无限好奇是核心品质

---

## 结语

Peter Steinberger 的分享揭示了 AI 辅助编程的最前沿实践。他的核心信息可以总结为一句话：

> **"Just talk to it. Play with it. Develop intuition."**
> — Just Talk To It, 2025年10月

这不是关于工具的选择，而是关于与 AI 协作的心态转变。从"我写代码"到"我与 agent 对话"，从"审查代码"到"设计架构"，从"控制过程"到"塑造结果"。

---

*报告完成于 2026年1月29日*
*分析文件位于 `peter-steinberger-interviews/analysis/` 目录*
