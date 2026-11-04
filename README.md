# agent-learning

阿伟的 Agent 自学仓库：跟着斯坦福 CS329Z 和几篇必读文章，从零搭一个能跑的 agent。

## 课程

- [斯坦福 CS329Z《Engineering AI Agents》](https://cs329z.stanford.edu/) — 主课。讲义、阅读材料、课表都在这儿，讲到哪放出到哪；官方录像不对公众开放，自学看讲义 + 阅读 + 公开的 HW1 starter 仓库
- [CS329Z 课程站（斯坦福镜像）](http://web.stanford.edu/class/cs329z)

## 必读文章

按顺序读：

1. [Anthropic《Building Effective Agents》](https://www.anthropic.com/engineering/building-effective-agents) — 先读这篇。workflow 和 agent 的区别，五个可组合的模式
2. [OpenAI《A Practical Guide to Building Agents》](https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/) — 模型 / 工具 / 指令三件套，什么时候该建 agent，怎么做 guardrails
3. [Google《Agents》白皮书](https://www.kaggle.com/whitepaper-introduction-to-agents) — 模型 / 工具 / 编排层，ReAct 循环，Level 0–4 能力分级
4. [Anthropic《How We Built Our Multi-Agent Research System》](https://www.anthropic.com/engineering/multi-agent-research-system) — 多智能体到底值不值，这篇给了数
5. [Anthropic《Effective harnesses for long-running agents》](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — 长时间任务怎么做 harness
6. [Anthropic《Building a C compiler with parallel Claudes》](https://www.anthropic.com/engineering/building-c-compiler) — 16 个 Claude 并行写编译器的大型实战复盘
7. [12-Factor Agents（HumanLayer）](https://github.com/humanlayer/12-factor-agents) — 从 demo 跨到能上线的 12 条生产原则

Google / Kaggle 5 天强化课系列（白皮书 3 的延伸）：

- [工具与 MCP](https://www.kaggle.com/whitepaper-agent-tools-and-interoperability-with-mcp)
- [上下文工程：会话与记忆](https://www.kaggle.com/whitepaper-context-engineering-sessions-and-memory)
- [Agent 质量](https://www.kaggle.com/whitepaper-agent-quality)
- [从原型到生产](https://www.kaggle.com/whitepaper-prototype-to-production)

## 学习路线

> 详细课表见 [CURRICULUM.md](CURRICULUM.md)（以 CS329Z + HW1 为主线），读书笔记在 [notes/readings.md](notes/readings.md)，每课讲义在 [lessons/](lessons/)。

1. 先做一个能跑的：Hugging Face Agents Course 或 Kaggle 5 天课
2. 读上面 1–3 篇文章
3. 只挑 CS329Z 关键几讲：基础全景、工具调用与 MCP、框架与编排、设计模式、记忆与多智能体、评测、安全与护栏
4. 手写 harness：v0 一次 chat 调用 → v1 工具调用 → v2 循环加最大步数 → v3 markdown 记忆文件 → v4 检索当工具（RAG）→ v5 评测集与护栏
5. 练手项目 HW1-mini：GenUI 找题 或 EQB 组卷（二选一，待定）

完成标准：固定问题集稳定通过、失败可定位、成本有账本、有步数 / 超时上限、不可逆操作有人确认、换模型还能跑。
