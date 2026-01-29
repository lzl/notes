# GitHub Open Source Friday 采访分析

**视频链接**: https://www.youtube.com/watch?v=1iCcUjnAIOM  
**时长**: 约1小时13分钟  
**日期**: 2026年1月24日  
**主持人**: Andrea Griffith (GitHub)

---

## 一、从燃尽到回归：Peter 的背景故事

Peter 在这次采访中再次分享了他的职业历程。他经营了 13 年的 B2B 公司，构建了"世界领先的 PDF 框架"，将公司发展到约 70 人规模，后来收到了一个"无法拒绝的收购要约"。

> [00:05:01] "I was running my own company for 13 years... it was B2B. We built the world's leading PDF framework. I built it up to around 70 people and yeah, at some point we got an offer I couldn't refuse basically."

卖掉公司后，他完全燃尽了。他再次使用了《王牌大贱谍》的比喻：

> [00:05:46] "You know the scene where they suck the mojo out? That was it."

他用了大约三年时间恢复，"补上生活的功课"（catch up on life）。2025 年初，当 AI 从"这很糟糕"变成"这很有趣"时，他的"火花"（spark）回来了。

**关键洞察**：Peter 强调这是他从事技术工作以来"最有趣的一年"——考虑到他曾经创建并发展了一家 70 人的公司，这个评价分量很重。

---

## 二、Clawdbot 的诞生：从语音消息开始的"顿悟时刻"

Peter 详细讲述了 Clawdbot（现名 Moltbot）的起源。他早在 2025 年 4 月就想要一个"生活助手"（life assistant），但当时的模型不够好。他想："大公司肯定会做这个。"

到了 11 月，市场上仍然没有他想要的产品：

> [00:08:54] "In November I looked and still there was nothing. Like where is my agent?"

他用一小时搭建了一个粗糙的原型——在 WhatsApp 上接收消息，发送给 Claude Code，返回结果。然后他添加了图片支持。

**顿悟时刻**发生在马拉喀什的一次朋友生日旅行中。他不经意间给 bot 发了一条语音消息，但他还没有实现语音功能：

> [00:10:16] "I just sent it a voice message... and then like the typing indicator appeared. I'm like, 'Huh, I'm curious what's going to happen now.' And then for like 5 seconds and then my bot answered as if nothing would have happened."

Agent 解释说它：
1. 发现文件没有扩展名
2. 检查了文件头，发现是 Opus 格式
3. 用 ffmpeg 转换
4. 想用 Whisper 但安装失败
5. 找到了 Peter 环境中的 OpenAI API key
6. 用 curl 发送到 OpenAI 获取转录

> [00:11:27] "I think that was the moment where it clicked for me really... it's like holy hell, these things are resourceful to a degree that I never thought of."

更疯狂的是，当 Peter 开玩笑说担心电脑在酒店被偷时，agent 发现了 Peter 电脑上的 Tailscale，找到了网络上的其他电脑，然后**自己迁移到了伦敦的另一台电脑上**。

> [00:12:11] "Basically migrated itself to like my computer in London. I know this is how Skynet starts, right?"

---

## 三、Agent 的概念：给电脑一个"幽灵共事者"

Peter 对 Clawdbot 的解释非常生动：

> [00:18:01] "Imagine you buy a new computer and you have this ghost entity and you give it a keyboard and a mouse and internet access and it's like a virtual coworker and then you can just talk to that and give it things to do. So anything you can do in a computer, that agent can do in a computer."

这个"幽灵共事者"的概念很好地解释了为什么 Clawdbot 如此强大——它不是一个功能受限的聊天机器人，而是一个有完整计算机访问权限的 agent。

---

## 四、开源哲学与 PR 作为"Prompt Request"

Peter 强调自从 2025 年 4 月回归后，他做的一切都是开源的：

> [00:14:42] "Ever since April everything I do is open source."

他分享了一个关于开源贡献的新观点。由于 agent 有计算机访问权限并且知道 GitHub 如何工作，很多从未学过编程或提交过 PR 的人现在也能贡献代码：

> [00:22:06] "I mostly see them as prompt requests instead of a pull request."

这种新的思维方式改变了他处理 PR 的方式：

> [00:24:39] "In the old ways, you would submit a PR and then you wait a few days and then somebody might say, 'This is wrong. This is wrong. This is wrong. You're a bad person.' Then you're like try again and then you go like a few rounds and then after a few weeks maybe the PR gets merged."

现在：

> [00:24:39] "I think in that was valid when code was hard and expensive. Now code is easy."

他的新方法是：
1. 把 PR 看作"这是一个问题，这是我尝试解决的方案"
2. 用 Codex 分析 PR，理解用户想解决什么问题
3. 大多数时候忽略具体代码，专注于理解痛点
4. 如果功能有价值，自己重新实现（因为新手通常不理解整体架构）

> [00:26:01] "I could never move as fast if I would try it that way."

**值得关注的行动项**：
- 重新思考开源贡献的本质——代码变便宜了，想法和反馈变得更有价值
- 考虑将 PR 看作"prompt request"，专注于理解意图而非审查代码细节

---

## 五、Agent 的"灵魂"：保密的核心配置

Peter 透露 Clawdbot 有一个叫做"soul"的文件，这是唯一不开源的部分：

> [00:34:55] "There's only one thing that I keep closed which is the soul. So in Claude there's a little system where they have the agents file, of course they have an identity file, they have files for memory, and they have one file which is like their soul where you kind of put in the values, how the agent should think and interact and what's important to you."

他把这个文件保密有两个原因：
1. 这是他仅有的 0.001% 的秘密
2. 这是一个渗透测试——如果有人能成功提取这个文件，说明 prompt injection 防护有问题

> [00:35:34] "So far nobody has exfiltrated Claude's soul and a lot of people tried... which does give me some confidence that at least all these labs working on getting prompt injection better and like having mitigation risks, it got a lot better."

他指出，如果使用非常小或旧的模型，重复询问最终可能会泄露一切。但最新一代的模型已经好很多了。

**值得关注的行动项**：
- 为你的 agent 创建一个"灵魂"文件——定义价值观、行为模式、重要原则
- 使用强大的模型来防止 prompt injection

---

## 六、心跳机制：Agent 的主动行为

Clawdbot 有一个独特的"心跳"（heartbeat）功能，让 agent 可以主动行动而不只是被动响应：

> [00:51:04] "Every 30 or 60 minutes depending on what model you use, the agent gets a ping. It's like 'is there something that needs you need to check, is there like a to-do you need to do.' So they automatically try to figure out if there's like any loose ends and will think about it and either ping you or not."

这使得 agent 可以：
- 早上问好
- 检查你的进度（"你今天去健身房了吗？"）
- 督促你实现目标
- 在深夜提醒你该睡觉了

> [00:52:04] "My Claude very unsuccessfully tries to get me to sleep after like it's 1 or 2 a.m. like 'Peter, I still see you awake. You need to sleep.'"

**值得关注的行动项**：
- 考虑为你的 agent 实现心跳机制
- 利用这个功能让 agent 帮助你保持健康习惯

---

## 七、社区创新用例

Peter 分享了用户们的创意用法：

1. **自动为图片添加字幕**
2. **控制 Tesla 汽车**
3. **伦敦公共交通查询**（告诉你是否需要跑着赶车）
4. **控制 Eight Sleep 床垫温度**
5. **清理 10,000 封邮件**
6. **控制 Sonos 音响、摄像头、灯光**
7. **查看食物订单配送状态**
8. **自动从 Tesco 购物**（告诉它买什么，几小时后送到门口）
9. **处理发票和报销**
10. **家庭成员 agent 之间的协调**

> [00:49:04] "What I find really cool is I saw some use cases where people set it up for families. So they're like I would get my agent and my wife would get the agent and then those agents can also talk to each other to synchronize if there's like a common to-do."

**关于健身追踪**：可以连接 Aura Ring、Garmin 手表等设备。Apple 是最难对付的，需要一个 iPhone 应用来同步数据。

---

## 八、模型推荐与技术选择

**推荐的模型**：
- **Opus（Anthropic）**：目前最好的"性格"（character）
- **OpenAI**：很可靠
- **MiniMax M2.1**：最好的开源选择，性价比高（$10/月 vs Anthropic $100/月）

**不推荐的模型**：
> [00:55:27] "Gemini is not a good model, it's not doing well."

Peter 对 Gemini 的评价很直接——它不擅长工具调用和 agentic 任务。可能可以用于写代码，但不适合 Clawdbot 的用途。

**关于技术选择**：

> [01:05:20] "Ever since AI, I don't really care so much about languages anymore. The language is not what matters so much. It's more the ecosystem that matters."

他选择 TypeScript 的原因：
1. 世界上最容易上手和可 hack 的语言是 JavaScript/TypeScript
2. 非常适合 web 相关工作（大量 JSON 处理）
3. 使更多人能够贡献

> [01:07:12] "This project is kind of insane because it is 100% written with AI. There's not a single line in there that I typed by hand."

但他强调，即使是 AI 写的代码，也需要人来提供"vision, taste and love"。

---

## 九、安全与沙箱

Peter 目前正在开发的重点是**沙箱和允许列表系统**：

> [00:45:27] "What I worked on this week is sandboxing... this week is kind of week where I'm working on allow lists. So ideally you can set up things that are safe and if the agent wants to run something that is unsafe, you get a nice little popup where you can say allow once, allow always."

关于安全风险的现实看法：

> [00:33:04] "Most people in Discord, they understand the risk that theoretically the model could do that. It just never happened before. Unless you even if you would prompt it, it would push back and say, 'Oh, this is a really bad idea.'"

他的观点是：如果你想搞坏自己的电脑，你也可以直接在终端输入命令。自从 5 月份开始用 agent 作为"智能终端"以来，他从未遇到过问题。

---

## 十、生产力对比：个人 vs 公司

这是采访中最令人印象深刻的对比之一：

> [00:21:27] "I ship more code every day than my company did in a month... and we had like 70 people."

这不是夸张——这是 AI 工具带来的真正变革。

关于发布频率：
> [01:03:22] "It's about 200-300 commits every day."

---

## 十一、未来优先事项

Peter 分享了他的短期目标：

1. **简化安装**：确保一行命令在任何地方都能工作
2. **移动应用**：iPhone、Android、Mac 应用（已存在但还不够好）
3. **沙箱系统**：让非技术用户也能安全使用
4. **构建社区**：找到更多维护者，让项目不只依赖他一个人

> [01:01:00] "I want to make sure it makes it work wherever you enter it... it should be simple to install. I want not just have this, but I also want to have apps for the iPhone and Android and for the Mac."

---

## 十二、关键原话与时间戳索引

| 时间戳 | 原话 | 主题 |
|--------|------|------|
| 00:06:20 | "May I say the most interesting year since I did tech" | 对 AI 年的评价 |
| 00:11:27 | "Holy hell, these things are resourceful to a degree that I never thought of" | Agent 能力 |
| 00:12:11 | "Basically migrated itself to like my computer in London. I know this is how Skynet starts" | Agent 自主性 |
| 00:18:01 | "Imagine you buy a new computer and you have this ghost entity" | Agent 概念 |
| 00:21:27 | "I ship more code every day than my company did in a month" | 生产力 |
| 00:22:06 | "I mostly see them as prompt requests instead of a pull request" | 开源新思维 |
| 00:24:39 | "That was valid when code was hard and expensive. Now code is easy" | 代码价值变化 |
| 00:34:55 | "There's only one thing that I keep closed which is the soul" | Agent 架构 |
| 00:51:04 | "The agent gets a ping... is there something that needs you need to check" | 心跳机制 |
| 00:55:27 | "Gemini is not a good model, it's not doing well" | 模型评价 |
| 01:05:20 | "The language is not what matters so much. It's more the ecosystem" | 技术选择 |
| 01:07:18 | "What those agents still miss is vision and taste and love" | 人的价值 |

---

## 十三、可采取行动的清单

1. **立即行动**：
   - 尝试设置 Clawdbot/Moltbot
   - 用旧电脑或 VPS（如 Hetzner）作为 agent 主机
   - 推荐 Telegram 作为消息平台（最好的 bot API）

2. **短期实践**：
   - 为你的 agent 创建一个"灵魂"文件
   - 实现心跳机制让 agent 主动帮助你

3. **长期思考**：
   - 重新思考"代码贡献"的意义——PR 可能更像是"需求描述"
   - Agent 可以帮助你完成你焦虑去做的事情（如打电话预约）
   - 未来可能是"让你的 agent 和我的 agent 谈"
