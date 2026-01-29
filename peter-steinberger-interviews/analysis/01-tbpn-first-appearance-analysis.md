# TBPN 采访分析：Peter Steinberger 首次公开亮相

**视频链接**: https://youtube.com/watch?v=qyjTpzIAEkA  
**时长**: 36分30秒  
**日期**: 2026年1月  
**主持人**: John Coogan, Jordi Hays (TBPN)

---

## 一、Peter 的背景与回归

Peter Steinberger 在采访中分享了他的职业历程。他经营了 13 年的软件公司（PSPDFKit，一个 PDF 框架），大约四年前卖掉了公司。卖掉公司后，他完全筋疲力尽（burned out）。

> [00:00:29] "I worked for my own software company for 13 years and then I sold it about four years ago. Then I was completely burned out."

他用了一个有趣的比喻来描述那段休息时光："blackjack and hookers"（当然是开玩笑）。他引用了一个经验法则：每工作 4 年需要休息 1 年，而他连续工作了 13 年，所以需要 3 年左右的休息时间。

在 2025 年 4 月，他的"火花"（spark）回来了。他用《王牌大贱谍》里的比喻来形容之前的状态——感觉像是有人把他的 Mojo 吸走了。

> [00:01:37] "It felt like someone sucked my Mojo out... but yeah, I had time to recover. I came back in April and I wanted to do something new."

他之前的背景是 Apple 和 iOS 开发，但他想尝试新东西，又不想因为缺乏经验而感到像个白痴。所以他开始研究 AI，发现它"不错，虽然不算很棒，但挺好的"。而且他正好在 Claude Code 2月份发布 beta 版本的时候回归，这是他第一次体验 AI 编程。

**值得关注的行动项**：
- 长期高强度工作后需要彻底的休息来恢复
- 在合适的时机进入新领域（AI），可以获得先发优势
- 保持好奇心和玩乐心态是持续创新的关键

---

## 二、Clawdbot/Moltbot 的诞生：偶然发现的"顿悟时刻"

Peter 在 2025 年 5 月就有了个人 AI 助手的想法，当时 GPT-4 刚出来，但模型还不够好。他当时想："大公司肯定会在几个月内做出这个东西，我干嘛要自己做？"

直到 11 月，他发现市场上仍然没有他想要的产品。

> [00:05:58] "In November I looked and still there was nothing. Like where is where is my agent?"

最初，他只是想在厨房做饭的时候能检查自己正在运行的 AI agent 的状态，于是花了大约 1 小时搭建了一个 WhatsApp 集成——收到消息后调用 Claude Code，然后返回结果。

**顿悟时刻**发生在一次摩洛哥马拉喀什的周末旅行中。他不经意间给自己的 agent 发了一条语音消息，但他并没有实现语音消息功能。神奇的是，agent 自己解决了这个问题：

> [00:09:07] "Yeah, you sent me a message, but there was only a link to a file. There's no file ending. So I looked at the file header. I found out that it's Opus. So I used ffmpeg on your Mac to convert it to wave. And then I wanted to use Whisper, but didn't have it installed, and there was an installation error. But then I looked around and found the OpenAI key in your environment. So I sent it via curl to OpenAI, got the translation back and then I responded."

> [00:09:47] "That was the moment where... these things are damn smart, resourceful beasts if you actually give them the power."

这个时刻让 Peter 意识到，当你给 AI agent 足够的权限和工具时，它们展现出惊人的"足智多谋"（resourceful）。

**值得关注的行动项**：
- 不要低估 AI agent 的解决问题能力
- 给 agent 足够的权限（如文件系统、shell 命令、API 密钥），它们会找到创造性的解决方案
- 从小的个人需求出发，往往能发现大的市场机会

---

## 三、CLI 优于 MCP 的哲学

Peter 在采访中明确表达了他对 MCP（Model Context Protocol）的看法：

> [00:12:11] "My premise: MCPs are crap. Doesn't really scale."

他的替代方案是 CLI（命令行工具）。他的论点是：

1. **Agent 天生理解 Unix**：它们可以在你的电脑上调用成千上万个小程序
2. **自动发现**：Agent 只需要知道工具的名称，调用 help 菜单后就能加载所需的信息，然后就知道如何使用
3. **为模型而非人类设计**：如果你足够聪明，你会按照模型的期望来构建 CLI。比如模型期望 `--log` 参数，你就实现 `--log`

> [00:12:42] "Don't build it for humans, build it for models. So if they call --log, you build --log... it's like agentic driven... built how they think and everything works better. It's a new kind of software in a way."

在这种理念下，他构建了大量 CLI 工具：Google Places、Sonos 音响、摄像头、家居自动化系统等。每添加一个 CLI 和 skill，他的 agent 就获得更多能力，也变得更有趣。

**值得关注的行动项**：
- 考虑将你的 API 或服务包装成 CLI 而非仅仅是 MCP
- CLI 的 help 菜单是给 agent 的"说明书"
- 按照 agent 的思维方式（标准 Unix 约定）来设计接口

---

## 四、模型选择：Opus 的"人格"vs Codex 的可靠性

Peter 分享了他对不同模型的看法：

**Claude Opus**：
- "Character-wise... I don't know what they trained their model on, how much of Reddit is in there or whatever, but it behaves so good in a Discord."
- 它在 Discord 中表现得像个人类——不会回复每条消息，而是听着对话，然后偶尔来一个精彩的发言（banger）
- 它的笑话比其他 AI 好笑，这很难得

**OpenAI Codex**：
- 更可靠、更像一个可靠的工人
- 对于编程，Peter 更喜欢 Codex，因为它能导航大型代码库
- "I literally prompt and then push to main and I have like 95% certainty that it actually works"
- 但需要更少的"手把手"指导

> [00:18:24] "With Claude Code you need more tricks to get the same... you need more charade I sometimes say... both are good but I can parallelize faster by Codex because it requires less handholding."

**值得关注的行动项**：
- 根据任务类型选择模型：社交/创意用 Opus，编码用 Codex
- 不同模型有不同的"性格"，找到与你工作风格匹配的
- 考虑任务的并行化能力——需要更少监督的模型可以更好地并行

---

## 五、被迫改名：从 Clawdbot 到 Moltbot

Peter 透露他收到了 Anthropic 的邮件，要求他重命名项目（因为 Clawdbot 和 Claude 太相似）。

> [00:19:48] "Kudos, they were really nice. They didn't send their lawyers. They sent someone internally. But the timeline was a bit rough."

他形容当天"所有可能出错的事情都出错了"。在重命名过程中，他同时打开两个 Twitter 窗口，一个用来重命名账号，另一个用来创建新账号。但在他完成之前，crypto 投机者已经抢注了新名字（他们有脚本在监控）。

> [00:21:15] "We do it live." [笑]

幸运的是，X（Twitter）团队帮他迅速解决了这个问题。主持人认为从长远来看，拥有独立的品牌对 Moltbot 是好事。

---

## 六、数据解放与"APP 消亡论"

Peter 描述了一个有趣的未来愿景：很多 APP 将会"融化消失"。

> [00:26:04] "A lot of apps will just melt away. Why do I still need MyFitnessPal? I just make a picture of my food. My agent already knows I'm at McDonald's making bad decisions."

他的逻辑是：
- Agent 结合已有信息，可以精确知道你在吃什么
- 然后自动调整你的健身计划
- 你不再需要健身 APP，agent 会确保你达到目标

> [00:26:54] "Most apps will be reduced to API and then the question is do you still need the API if I can just save it somewhere else?"

关于数据访问，他也承认他的一些 CLI 工具是通过指向网站让 Codex 逆向工程 API 来构建的，有时候可能违反服务条款。但他不太在意。

**值得关注的行动项**：
- 思考你的产品/服务在 AI agent 世界中的定位
- 如果你的产品只是在收集和展示数据，可能会被 agent 替代
- 考虑提供 API 或 CLI 接口，让 agent 可以集成你的服务

---

## 七、开源哲学与拒绝 VC

Peter 在采访中多次强调他的开源理念：

> [00:14:14] "This is open source. My motivation is have fun, inspire people, not make a whole bunch of money. I already have a whole bunch of money."

当被问到是否会成立公司时，他的回答让"一万个 VC 气得捶墙"：

> [00:32:26] "I think instead of a company, I would much rather consider a foundation or something that is nonprofit."

关于 MIT 许可证和别人可能会拿代码去卖：

> [00:33:29] "It doesn't even matter that much. You know, code is not worth that much anymore. You could just delete that and then build it again in months. It's much more the idea and the eyeballs and maybe the brand that actually has value."

这反映了他对代码价值的重新评估——在 AI 时代，代码本身的价值降低了，想法、用户关注度和品牌才是真正有价值的。

**值得关注的行动项**：
- 在 AI 时代重新评估"代码"的价值
- 开源可以是一种品牌策略，而非放弃价值
- 考虑非营利/基金会模式作为可持续发展的替代方案

---

## 八、安全挑战：Prompt Injection 尚未解决

Peter 坦诚地谈到了安全问题：

> [00:29:24] "The thing is I built this for fun, for me to use one-on-one on WhatsApp or Telegram... the model was that you trust the people that are in there."

但现在人们在不受信任的环境中使用它（比如把调试用的 web app 暴露在公网上），导致他面临大量安全报告。

> [00:31:06] "Prompt injection is not solved... there is absolute risk."

他认为这种"需求"会加速相关研究，推动行业找到解决方案。

**值得关注的行动项**：
- 在给 AI agent 权限时，始终考虑威胁模型
- Prompt injection 是当前 AI 系统的普遍挑战
- 个人使用 vs 公开部署需要不同的安全考量

---

## 九、关键原话与时间戳索引

| 时间戳 | 原话 | 主题 |
|--------|------|------|
| 00:04:25 | "My main mantra is I want to have fun" | 创作动机 |
| 00:04:34 | "I do agentic engineering and then when it starts hitting 3am I switch to vibe coding and next I have regrets" | Agentic vs Vibe Coding |
| 00:05:38 | "You have to close the loop. That's always the secret." | Agent 设计原则 |
| 00:09:47 | "These things are damn smart, resourceful beasts if you actually give them the power" | Agent 能力 |
| 00:12:11 | "MCPs are crap. Doesn't really scale." | 技术选型 |
| 00:12:42 | "Don't build it for humans, build it for models" | 设计哲学 |
| 00:16:45 | "You can now ship as much as a company could a year ago" | 生产力提升 |
| 00:26:57 | "Most apps will be reduced to API" | 未来预测 |
| 00:33:37 | "Code is not worth that much anymore" | 价值重估 |

---

## 十、可采取行动的清单

1. **立即行动**：
   - 尝试给现有的 shell 命令和脚本添加良好的 `--help` 输出
   - 考虑用 CLI 而非 MCP 来集成服务

2. **短期计划**：
   - 评估你的产品/工作流中哪些可以被 AI agent 替代或增强
   - 尝试将一些重复性任务委托给 AI agent

3. **长期思考**：
   - 在 AI 时代，什么才是真正有价值的？（想法、品牌、用户关注）
   - 如何设计"为 agent 而生"的软件接口？
