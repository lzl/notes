# 采访分析：He Built a SUPER App to Control AI Agents

**视频链接**: https://www.youtube.com/watch?v=fu7th5HiADo  
**时长**: 29分10秒  
**主持人**: Mayank Gupta  
**说明**: 这是两部分采访的第一部分

---

## 一、AI Agent 是程序员的"老虎机"和"猫薄荷"

Peter 在采访开头就用了两个生动的比喻来描述 AI agent：

> [00:01:10] "I rarely actually call them agents, but many times I just call them slot machines because I feel they are the ultimate drug for programmers. Or my friends call them catnip for programmers."

为什么是"老虎机"？因为：
- 你输入 prompt，可能会出来令你惊叹的东西
- 也可能 agent 陷入"幻觉之旅"（vision quest），把一切都搞砸
- 这是一个 0-1 的方程——要么大获全胜，要么一塌糊涂

这种不确定性带来的多巴胺刺激让人上瘾。Peter 承认他睡眠不足，但又停不下来。他举了一个例子：昨晚他在做一个虚拟键盘功能，因为 prompt 写得有点懒，agent 误解了他的意图，把键盘加到了登录屏幕上。

> [00:01:52] "It edited the keyboard to the login screen. So then you had your image, you had the password and you had the keyboard underneath and I'm like you stupid..."

但有趣的是，agent 总是很礼貌地回应——"哦，用户很沮丧，我需要认识到这一点"，然后道歉。

**值得关注的行动项**：
- 认识到 AI 编程的成瘾性，设置合理的工作边界
- 接受"老虎机"的本质——有时候赢，有时候输
- 当 agent 搞砸时，直接 revert 而不是试图修复

---

## 二、Prompt 工程：从多角度解释你的意图

这是采访中最实用的部分。Peter 的核心观点是：**如果你只给 agent 一个简短的 prompt，就会得到垃圾输出**。

> [00:02:33] "If you just give them a tiny prompt... shit will come out. If you put shit in, shit will come out."

他的解决方案是使用 **Whisper Flow**（一个语音转文字工具），用口述的方式写 prompt。这个工具的特别之处在于：

1. **自动修正**：如果你说"啊，这样做...不，那样做"，它只会输出"那样做"
2. **允许思考过程**：你可以边想边说，改变想法，工具会帮你整理

> [00:03:02] "Really what it also does, it paraphrases what I say. So if I say 'ah do it this way, or no do it that way', it will just write 'do it that way'."

关键技巧是**从多个角度解释同一件事**：

> [00:03:20] "So if I talk about 'oh like we want to make the layout better and the padding', then I say a few samples. 'You see like this button here it's too far on the right side and you know our goal is to have it right aligned.' So you repeat yourself from a few different angles and then usually you will get your perfect one-shot response."

**为什么这样做有效？** 因为你说得越多，agent 能够幻觉的空间就越小。它会更好地理解你的真实意图和背后的推理。

> [00:07:50] "The more you talk, the less they'll be able to hallucinate because they'll understand better what you actually mean and what your reasoning behind is."

**值得关注的行动项**：
- 使用语音转文字工具（如 Whisper Flow）来写 prompt
- 从多个角度解释你想要的东西
- 不要害怕"啰嗦"——详细的 prompt 带来更好的结果
- 可以在 prompt 中表达情绪（比如"这看起来像垃圾"），agent 能接受

---

## 三、并行工作：同时操作 2-4 个 Agent

Peter 的工作模式是同时处理多个项目/任务：

> [00:04:03] "The way I do it is I work on usually two to four different things based on how much I actually slept and how complex it is."

他的工作流程：
1. 给一个 agent 下 prompt
2. Agent 开始工作（通常需要 5-15 分钟）
3. 转到下一个项目，给另一个 agent 下 prompt
4. 如此循环

关键点是**所有项目都在同一个目录**，但他选择不同的区域来工作，以减少 agent 之间相互干扰的可能性。

> [00:04:35] "Everything goes into the same directory. I just pick different areas of my products so that the chances that they interfere are low."

然后用 Git 工具（如 Tower）来查看所有的 diff，要么接受，要么 revert。

**关于 Revert**：
> [00:05:01] "If they make shit, then just revert it."

但他也承认，Claude Code 的 revert 不如 Cursor 方便。如果你不经常 commit，可能会陷入混乱。

---

## 四、Claude Code vs Cursor 的深度对比

Peter 明确表示他更喜欢 Claude Code 的工作流：

> [00:05:33] "I can stop using Cursor. I don't... I like the Claude Code workflow much better."

**Cursor 的问题**：
1. 切换 agent 需要 3 秒，很卡顿
2. 经常断线或有问题
3. **Agent 不主动读取文件**——这是 Cursor 为了省钱的设计

> [00:05:42] "The agents in Cursor, they are not very eager to fetch in a lot of files. Their system prompt is built in a way where it's optimized that you have to think what they see."

**为什么 Cursor 这样设计？** Peter 认为这是商业利益驱动的：

> [00:06:12] "Ultimately this is part of the way Cursor charges you money because they charge per request and not per token. So they need every request to not eat too many tokens."

**Claude Code/Anthropic 的优势**：
> [00:06:26] "Anthropic is just like 'om nom nom' eating tokens like crazy. That's also why they are better. They paper people."

这段话很有趣——Anthropic 因为按 token 计费而不是按请求计费，所以 Claude Code 可以"大口吃"token，读取更多文件，从而产生更好的结果。

**值得关注的行动项**：
- 理解不同工具背后的商业模式如何影响其行为
- Claude Code 的"吃 token"特性意味着更好的上下文理解
- 如果用 Cursor，需要更主动地提供文件上下文

---

## 五、软件工程的未来：20 倍生产力提升

当被问到 AI 对软件工程师就业的影响时，Peter 给出了务实的回答：

> [00:08:53] "That's society ultimately, right? We had a lot more farmer jobs 100 years ago, we had a lot more industrial jobs 50 years ago. You can't be mad at progress."

他的核心观点是：**工具让你的生产力提升 20 倍**。

> [00:09:08] "If you work them and if you learn them, you'll be no exaggeration 20 times as productive."

**具体例子**：一年前，你必须非常挑剔地选择要构建什么，因为每个项目都需要几周时间。现在，他在 4 小时内构建了一个完整的网站（带 Postgres 数据库、实时数据同步）。

> [00:10:05] "In just like four hours I built a whole website backed by Postgres, already hosted and it's already working. This would have taken me at least a few days back in the day."

而且这不是 100% 的精力——只是他 30% 的注意力，因为他同时在做其他事情。

**现在软件工程师的核心工作是什么？**

> [00:11:00] "What changed is now my task is to pick the right technologies."

选择正确的技术栈变成了最重要的决策。他举例说用 Supabase，因为它：
- 存在足够长时间，有大量"世界知识"
- 不需要额外提供文档，agent 自己就知道
- 有实时同步等现成功能

**值得关注的行动项**：
- 学习如何使用 AI 工具是当务之急
- 选择"世界知识"丰富的技术栈（流行、文档完善、存在时间长）
- 接受生产力范式的转变

---

## 六、被困时的策略：升级到更聪明的 Agent

Peter 分享了一个重要的工作哲学：当你被困住时，agent 可以帮你；当 agent 被困住时，你可以使用更聪明的 agent。

> [00:18:56] "Whenever you're stuck, they'll help you out. When they are stuck, you can use smarter agents."

他举了一个虚拟键盘的例子。前 20 分钟什么都不工作，于是他做了一些研究，找到了一个解释新 API 的长 markdown 文档，然后直接把 URL 给 Claude：

> [00:14:29] "I found a document, a very long markdown that explains how the new API works. I just copied the URL into Claude, 'read this and now build it right'."

**核心理念**：
> [00:19:12] "Does it defy the laws of gravity or physics? No. Then it can be done."

如果不违反物理定律，就一定能做到。

**值得关注的行动项**：
- 当 agent 卡住时，做一些研究，提供更多上下文
- 可以直接给 agent 文档 URL
- 保持"一切皆可能"的心态

---

## 七、开源哲学与"Startup Slop"争议

Peter 的原则是**一切开源**：

> [00:15:40] "That's my mantra. Everything I do is open source. There's literally maybe some old projects from before my burnout. But now everything you can look up on GitHub."

他提到了一个有趣的争议——他的网站被一个类似 Hacker News 的平台封禁，理由是"startup slop"（创业垃圾）。他对此感到困惑和被侮辱。

他的反驳是：虽然他使用 agent 来帮助写作，但他仍然在每篇博客文章上花费 3-4 小时，反复迭代，压缩内容：

> [00:16:23] "Even if I use agents to do most of the writing, I iterate so much on what I want to say. And then at some point my blog post is always like 20-25 minutes and then the hard work is to compress it into like 5 minutes."

他区分了两种内容：
1. **博客文章**：可以用 AI 辅助，因为目的是传递知识
2. **Newsletter**：100% 手写，因为那是更私人的声音

> [00:17:22] "My newsletter gives a different side of me, like the more personal side, and there I want to have 100% my voice."

**值得关注的行动项**：
- 使用 AI 辅助写作不等于"slop"，关键是投入的思考和迭代
- 为不同目的的内容设置不同的标准
- 文档既是为了帮助他人，也是为了帮助未来的自己

---

## 八、VibeTunnel：从手机控制所有 Agent 的"超级应用"

采访的后半部分演示了 VibeTunnel——一个让你从手机上查看和控制所有终端 agent 的工具。

**起源故事**（Malcolm in the Middle 修灯泡 meme）：
> [00:21:22] "I had this idea of something I want to build... and then I noticed that the build tools suck. So I started building things so I could build better MCPs for the thing that I use to build Cursor. And now I'm one way deeper... I'm building something that helps me use Claude Code which helps me build MCPs which helps me build this which ultimately would help me build this. So I think I'm in level four inception."

**为什么需要这个工具？**

Agent 通常需要运行 10-20 分钟，你需要：
1. 检查它们是否还在正常运行
2. 看它们是否陷入"幻觉之旅"
3. 在它们过早停止时能够发现

> [00:23:40] "Claude or agent in general, they run 10-20 minutes, but sometimes you want to check up on them because they like to go on vision quests if you don't steer them."

**技术实现亮点**：
- 使用 AppleScript 打开新窗口
- 使用 Accessibility API 输入命令
- 注入二进制文件来控制终端
- 混合了"非常老旧和手工的技术"与现代前端

> [00:27:50] "It was really cool to build because you had to use this mix of really old and crafty technology and then you have a cool Mac UI, you have frontend that uses really modern stuff."

**他为什么不用云端解决方案？**
> [00:27:07] "All those companies want their cloud-based solution... But that's not a workflow that works for small developers because I want to work on my computer. I want to have my windows. I don't want to change."

**值得关注的行动项**：
- 当现有工具不满足需求时，考虑自己构建
- "Inception 式"的工具链是正常的——构建工具来构建工具
- 有时候最好的解决方案是混合新旧技术

---

## 九、关键原话与时间戳索引

| 时间戳 | 原话 | 主题 |
|--------|------|------|
| 00:01:14 | "I call them slot machines... the ultimate drug for programmers" | Agent 的本质 |
| 00:02:33 | "If you put shit in, shit will come out" | Prompt 质量 |
| 00:03:34 | "Repeat yourself from a few different angles and then usually you will get your perfect one-shot response" | Prompt 技巧 |
| 00:06:12 | "This is part of the way Cursor charges you money" | 商业模式分析 |
| 00:09:08 | "You'll be no exaggeration 20 times as productive" | 生产力提升 |
| 00:10:05 | "In just like four hours I built a whole website backed by Postgres" | 具体案例 |
| 00:11:00 | "My task is to pick the right technologies" | 工程师角色变化 |
| 00:19:15 | "Does it defy the laws of gravity or physics? No. Then it can be done." | 心态 |
| 00:22:25 | "I'm in level four inception" | 工具链 |
| 00:27:07 | "I want to work on my computer. I don't want to change." | 工作流哲学 |

---

## 十、可采取行动的清单

1. **立即行动**：
   - 尝试使用 Whisper Flow 或类似的语音转文字工具来写 prompt
   - 在 prompt 中加入更多上下文和解释

2. **短期实验**：
   - 尝试并行运行多个 agent，处理项目的不同部分
   - 测试 Claude Code 和 Cursor 的区别

3. **长期策略**：
   - 重新评估你的技术栈选择——优先选择"世界知识"丰富的技术
   - 建立"被困时升级"的心智模型
