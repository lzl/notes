# Peter Steinberger 博客文章分析

> 博客来源：steipete.com
> 涵盖时间：2025年6月 - 2025年10月
> 三篇核心文章：
> 1. The Future of Vibe Coding (2025年6月1日)
> 2. My Current AI Dev Workflow (2025年8月25日)
> 3. Just Talk To It (2025年10月14日)

这三篇博客文章记录了 Peter 从"发现 AI 编程"到"完全掌握 Agentic Engineering"的演进过程，提供了视频采访之外的技术细节和实操经验。

---

## 一、时间线演进：从 Cursor 到 Codex

### 2025年6月：The Future of Vibe Coding
**核心状态**：刚开始探索，主要用 Cursor + Gemini

这是一篇基于3小时现场工作坊的记录，展示了从零开始构建"Vibe Meter"（一个追踪 AI 工具费用的菜单栏应用）的完整过程。

**当时的工具栈**：
- Google AI Studio + Gemini：用于规范生成
- Cursor IDE：主力开发工具
- Claude Code (CLI)：用于大规模重构
- 多个自建 MCP 工具（Peekaboo, Conduit, Terminator, Automator, Claude Code MCP）

**当时的花费**：Cursor 账单已接近 $900/月

### 2025年8月：My Current AI Dev Workflow
**核心状态**：从 VS Code 完全转回 Ghostty 终端

> "After going all-in on VS Code, I went fully back to Ghostty for my main setup."

**关键变化**：
- 硬件：Dell UltraSharp U4025QW 曲面显示器 (3840x1620)，可同时显示4个 Claude 实例
- 工具：Ghostty 终端成为主力，VS Code 仅用于查看代码
- MCP："Even removed my last MCP"——发现 Claude 有时会不被要求就启动 Playwright，反而污染 context
- 工作方式：多个 agent 直接在 main 分支工作，尝试过 worktree 但发现拖慢速度

**测试策略**：
> "The model almost always finds issues when you ask it to write tests IN THE SAME CONTEXT. Context is precious, don't waste it."

### 2025年10月：Just Talk To It
**核心状态**：完全转向 GPT-5-Codex，彻底重塑工作流

> "I've completely moved to cli as daily driver. I run between 3-8 in parallel in a 3x3 terminal grid."

这是他最成熟、最系统的工作流总结。

---

## 二、从 Claude Code 到 Codex 的转变

### 为什么离开 Claude Code

Peter 在博客中直言不讳：

> "I used to love Claude Code, these days I can't stand it anymore... It's language, the 'absolutely right's, the '100% production ready' messages while tests fail - I just can't anymore."

**具体问题**：
1. **语言风格**：过于乐观的"Absolutely right!"让他崩溃
2. **多秒冻结**：经常出现多秒延迟
3. **内存爆炸**：进程膨胀到几 GB
4. **终端闪烁**：尤其在 Ghostty 中
5. **过早自信**：测试失败时还说"100% production ready"

### Codex 的优势

> "Codex is more like the introverted engineer that chugs along and just gets stuff done."

**具体改进**：
- **无随机 Markdown 文件**
- **语言风格**："This really makes a difference to my mental health. I've been screaming at claude so many times. I rarely get angry with codex."
- **速度**：OpenAI 用 Rust 重写，极快
- **消息队列**：可以排队消息
- **Token 效率**：context 填充慢很多
- **可用 context**：~230k vs Claude 的 156k

### 费用对比

> "I currently have 4 OpenAI subs and 1 Anthropic sub, so my overall costs are around 1k/month for basically unlimited tokens. If I'd use API calls, that'd cost my around 10x more."

---

## 三、"Blast Radius"（爆炸半径）概念

这是 Peter 在博客中首次系统阐述的工作方法论：

> "Whenever I work, I think about the 'blast radius'. When I think of a change I have a pretty good feeling about how long it'll take and how many files it will touch."

**实践方法**：
- 可以扔很多"小炸弹"，或者一个"胖子"加几个小的
- 如果扔多个大炸弹 → 无法做 isolated commits，出错时更难回滚
- 用于判断 agent 状态：如果比预期时间长，按 Escape 问"what's the status"

**什么时候该停止**：
> "Don't be afraid of stopping models mid-way, file changes are atomic and they are really good at picking up where they stopped."

---

## 四、为什么不用 Worktree/PR 流程

> "I run one dev server, as I evolve my project I click through it and test multiple changes at once."

**反对 worktree 的理由**：
1. 每个变更一个分支会显著变慢
2. 需要启动多个 dev server 很烦
3. Twitter OAuth 限制只能注册某些域名做回调

**他的替代方案**：
- 3-8 个终端并行运行
- 3x3 终端网格布局
- 大多数在同一个文件夹
- 实验性的放在单独文件夹

---

## 五、为什么不用第三方 Harness

Peter 对各种 AI 编程工具做了点评：

### amp
> "amp moved away from GPT-5 as driver and now calls it their 'oracle'. Meanwhile I use codex and basically constantly work with the smarter model, the oracle."

### Factory
> "Their videos are a bit cringe... images aren't supported (yet) and they have the signature flicker."

### Cursor
> "Its tab completion model is industry leading, if you still write code yourself... but Cursor still has the same bugs that annoyed me back in May."

### 总体判断
> "IMO there's simply not much space between the end user and the model company... Both codex and claude code are getting better with every release, and they all converge to the same ideas and feature set."

---

## 六、Plan Mode 的演进

### 早期（2025年6月）：重度使用 spec-driven development

> "Designing a big spec, then let the model build it, ideally for hours."

### 现在（2025年10月）：认为这是"旧思维"

> "IMO that's the old way of thinking about building software."

**新方法**：
1. 与 Codex 开始讨论
2. 粘贴网站、想法，让它读代码
3. 一起细化新功能
4. 如果复杂，让它写 spec
5. 把 spec 给 GPT-5-Pro review（"surprisingly often, this greatly improves my plan!"）
6. 把有用的粘贴回来

**最有趣的方式（UI 工作）**：
> "I often start with sth simple and woefully under-spec my requests, and watch the model build and see the browser update in real time. Then I queue in additional changes and iterate on the feature."

> "I often saw codex build something interesting I didn't even think of. I don't reset, I simply iterate and morph the chaos into the shape that feels right."

---

## 七、MCP 的真实看法

### 为什么 MCP 不如 CLI

> "Almost all MCPs really should be clis. I can just refer to a cli by name. I don't need any explanation in my agents file."

**CLI 的优势**：
1. Agent 第一次调用会失败，CLI 显示 help menu
2. Context 现在有完整信息如何使用
3. 不需要像 MCP 一样在 context 里付出常量成本

**MCP 的问题**：
> "Use GitHub's MCP and see 23k tokens gone. Heck, they did make it better because it was almost 50,000 tokens when it first launched."

### 唯一保留的 MCP
chrome-devtools-mcp：用于闭环 web 调试

---

## 八、Prompting 技巧演进

### 早期：长 prompt
> "Back when using claude, I used to write very extensive prompts, since this model 'gets me' the more context I supply."

### 现在：极短 prompt + 图片
> "I noticed that my prompts became significantly shorter with codex. Often it's just 1-2 sentences + an image."

**图片的威力**：
> "Adding images is an amazing trick to provide more context... I'd say at least 50% of my prompts contain a screenshot. I rarely annotate that, that works even better but is slower."

### 触发词
> "When things get hard, prompting and adding some trigger words like 'take your time' 'comprehensive' 'read all code that could be related' 'create possible hypothesis' makes codex solve even the trickiest problems."

---

## 九、Agents.md 文件管理

> "My Agent file is currently ~800 lines long and feels like a collection of organizational scar tissue."

**特点**：
- 他没有自己写，是 Codex 写的
- 每次出问题就让它加一条简洁的笔记
- 和 claude.md 做了 symlink（因为 Anthropic 没有标准化）

**内容包括**：
- Git 指令
- 产品解释
- 命名和 API pattern
- React Compiler 笔记
- React 模式偏好
- 数据库迁移管理
- 测试
- ast-grep 规则

**重要发现**：
> "Claude reacts well to 🚨 SCREAMING ALL-CAPS 🚨 commands... that freaks out GPT-5. (Rightfully so.) So drop all of that and just use words like a human."

---

## 十、20% 时间用于重构

> "I spend about 20% of my time on refactoring. Ofc all of that is done by agents, I don't waste my time doing that manually."

**重构时机**：
> "Refactor days are great when I need less focus or I'm tired, since I can make great progress without the need of too much focus or clear thinking."

**典型重构工作**：
- jscpd：检测代码重复
- knip：检测死代码
- eslint 的 react-compiler 和 deprecation 插件
- 合并可以整合的 API routes
- 维护文档
- 拆分过大的文件
- 添加测试和代码注释
- 更新依赖
- 文件重组
- 找出慢的测试并重写
- 应用现代 React pattern

---

## 十一、自建 MCP 工具详解（2025年6月）

### Peekaboo 👻
让 IDE 截图并询问图片问题。例如："这个设置页面是空白的吗？按钮看起来是启用状态吗？" 这是视觉自我修正的一步。

### Conduit MCP 🐱
高级文件操作用于更快重构。提供比基础终端命令更健壮可靠的文件系统操作。

### Terminator 🤖
在循环外管理终端。如果 AI 需要运行 dev server，这个 MCP 在正确分离的非阻塞外部 shell 中运行它。防止主 AI 生成循环被卡住。

### Automator 🎯
IDE 的 AppleScript。

### Claude Code MCP 🧠
> "It allows the Gemini agent in Cursor to delegate specific tasks to Anthropic's Claude Code running in my terminal... This is fantastic for playing to each AI's strengths."

---

## 十二、关键引用汇总

| 文章 | 引用 | 主题 |
|------|------|------|
| Just Talk To It | "Just talk to it. Play with it. Develop intuition." | 核心方法论 |
| Just Talk To It | "I used to love Claude Code, these days I can't stand it anymore" | 工具选择 |
| Just Talk To It | "Models are incredibly clever and no hook will stop them if they are determined" | Agent 行为 |
| Optimal Workflow | "Less is more" | 工具简化 |
| Optimal Workflow | "Even removed my last MCP" | MCP 评估 |
| Optimal Workflow | "Tests IN THE SAME CONTEXT. Context is precious" | 测试策略 |
| Future of Vibe Coding | "The better your spec, the less friction you'll have" | 规范的价值 |
| Future of Vibe Coding | "AI critiquing AI" - 让一个 AI review 另一个的输出 | spec 迭代 |
| Future of Vibe Coding | "Screenshots are your best friend" | Prompting 技巧 |

---

## 十三、可落地的行动清单

### 工具配置
1. 配置 Ghostty 终端 + 3x3 网格布局
2. 获取 OpenAI 订阅（$20/月），考虑多个订阅
3. 设置 tmux 处理后台任务
4. 使用 CLI 而非 MCP（gh, vercel, psql, axiom）

### 工作流程
1. **Blast Radius 思维**：每个任务前估计影响范围
2. **并行工作**：3-8 个 agent 同时运行
3. **图片优先**：50%+ prompt 应该包含截图
4. **短 prompt**：1-2 句话 + 图片
5. **20% 重构**：定期用 jscpd, knip, ast-grep 清理

### Prompting 模式
1. 开始时："let's discuss" 或 "give me options"
2. 困难时："take your time", "comprehensive", "read all code that could be related"
3. 队列任务：Codex 支持消息队列
4. 停止时机：比预期长就按 Escape 问 status

### 测试策略
1. 在同一个 context 中写测试
2. 让 agent 自己发现 bug
3. 对 UI 调整可能不需要测试
4. 保持 context 连续性

### Agents.md 管理
1. 让 agent 自己维护这个文件
2. 出问题时让它加笔记
3. 不要用全大写命令（GPT-5 不吃这套）
4. 包含：产品解释、命名规范、技术栈笔记、pattern 偏好

---

*分析完成于 2026年1月*
