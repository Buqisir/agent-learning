# 课程表：以 CS329Z 为主线，造一个 EQB agent

## 总思路

CS329Z 的 HW1（[cs329z/assignment1-harness](https://github.com/cs329z/assignment1-harness)，MIT 协议，公开）几乎就是我们要的东西：

- 不用任何 agent 框架，自己从零写一个 harness
- 先写 LLM 流水线（workflow），再写 agent 循环
- 依次加上工具、检索、终端、压缩、记忆、审批、护栏和评测

它自带一整套测试：

- **确定性测试**：用假模型回放固定回复，不花钱，告诉你代码"对不对"
- **实时测试**：调真模型，跑一遍大约几毛钱

所以分两个阶段：

1. **阶段一（第 1–7 课）**：跟着 CS329Z 讲义和 HW1，把整个 harness 亲手做一遍。文章穿插着读。
2. **阶段二（第 8 课起）**：同一套 harness，把工具和数据换成 EQB 的，造出 EQB agent。先用假数据，再接 EQB 真 API，最后包成 MCP。

每课的固定结构：**读**（讲义 + 文章）→ **懂**（概念讲解 + 思考题）→ **做**（HW1 的题，测试变绿）→ **迁移**（这一课在 EQB agent 里对应什么）→ **记**（设计决定写进笔记）。

## 课表

| 课 | CS329Z 讲次 | 配套阅读 | 动手（HW1） | EQB 迁移 |
|---|---|---|---|---|
| 0 | — | — | 环境：uv、Python 3.11、OpenRouter key、预算上限 | — |
| 1 | L01 基础全景、L02 后半（上下文工程） | Anthropic《Building Effective Agents》；OpenAI 指南前半；Google《Introduction to Agents》 | P1 priority：一次调用 + 解析 + unknown 兜底 | 一次调用给一道题判题型 / 难度 |
| 2 | L03 RAG | （讲义论文：RAG、DPR、ColBERT） | P3 BM25 从零写；P4 多跳 RAG | 用 BM25 搜题库里的文章和题干 |
| 3 | L04 工具调用与 MCP | Google《Agent Tools & MCP》；12-Factor F1、F4；BEA 附录 2 | P5 工具注册表；P6 工具调用协议 | 设计 `search_questions`、`get_question` 的 schema |
| 4 | L05 框架、L06 设计模式（ReAct） | 12-Factor F2、F3、F5、F8、F9、F13；OpenAI 指南编排部分 | P2 每日摘要（workflow）；P7 agent 循环；P8 工具组装 | 组卷：哪些写死、哪些交给模型 |
| 5 | L06 上下文管理、L07 记忆 | Google《Context Engineering》；Anthropic《Effective harnesses》 | P9 检索当工具；P10 终端；P11 压缩；P12 记忆 | 老师偏好档案（年级、难度、题型），不存学生数据 |
| 6 | L08 多智能体（只读）、L18 安全与护栏 | Anthropic《Multi-Agent Research System》；OpenAI 指南护栏部分；Google MCP 篇安全部分 | P13 审批 / 插话 / ask_user；P14 deny 规则与提示注入 | 保存、发布试卷前必须老师确认 |
| 7 | L13 评测基础、L15 LLM-as-judge | Google《Agent Quality》《Prototype to Production》；Anthropic《C compiler》 | P15 模拟用户评测；P16 使用笔记；P17 设计备忘录 | 20 个组卷请求的评测集 |
| 8+ | L19 编码 agent（参考） | 12-Factor 全部回顾 | — | EQB agent：假数据 → EQB HTTP API → MCP |

## 完成标准（沿用 README）

- 固定问题集能稳定通过
- 失败了能定位到哪一步出的错
- 成本有账本可查
- 有最大步数和超时上限
- 不可逆操作要人确认
- 换个模型还能跑

## 文件

- `notes/readings.md`：所有文章的读书笔记，每篇末尾写"对 EQB 的启发"
- `lessons/`：每课的讲义和作业
- `CURRICULUM.md`：本文件

## 已知缺口

- CS329Z 讲义 PDF 放在 `web.stanford.edu/class/cs329z/slides/` 下，这个云端环境访问不了（503，证书链问题）。课程主页、课表和 HW1 都读过了。讲义需要你在自己电脑上打开。
- 每讲的论文（ReAct、MemGPT 等）还没读，讲到对应那课时再读。
