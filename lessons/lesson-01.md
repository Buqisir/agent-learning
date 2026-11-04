# 第 1 课：agent 到底是什么

对应 CS329Z L01「Foundations & Landscape」和 L02 后半「context engineering」。

## 1. 读

先读这些，按顺序：

0. CS329Z 讲义 [lecture01.pdf](http://web.stanford.edu/class/cs329z/slides/lecture01.pdf)（71 页，翻得很快）和 [lecture02.pdf](http://web.stanford.edu/class/cs329z/slides/lecture02.pdf) 最后一节「LLMs for Agents」（从 Structured I/O 开始）。
1. Anthropic《Building Effective Agents》全文。约 15 分钟。整门课的骨架都在这篇里。
2. OpenAI《A Practical Guide to Building Agents》，读到 "Agent design foundations" 为止。
3. Google《Introduction to Agents》，读前三节：定义、5 步循环、Level 0–4。

选读，都是 CS329Z L01 指定的：

- BAIR《The Shift from Models to Compound AI Systems》
- Lilian Weng《LLM Powered Autonomous Agents》
- Kapoor 等《AI Agents That Matter》，讲评测为什么必须算成本

## 2. 懂：核心概念

### 2.0 先说学法（L01 讲义里的一张片子）

L01 专门放了一张「技能会退化」：Anthropic 2026 年做了一个随机对照实验，用 AI 辅助学一个新 Python 库的工程师，事后测验低了 17%，差距最大的是调试能力。课程的态度是**用 AI 来学会怎么学**，而不是让 AI 替你写。

所以我们的规矩是：代码你自己写，我讲、出题、看你的代码。HW1 的课程组也一样，会用口头抽查确认你真的懂自己交的东西。

### 2.1 复合 AI 系统 ⊃ workflow 和 agent

CS329Z 第一讲的视角是：今天的好结果，靠的不是单个模型，而是**多个组件拼成的系统**：模型、检索器、工具、代码。

这类系统按「谁决定下一步」可以分成两种：

| | 谁决定下一步 | 例子 |
|---|---|---|
| **Workflow（工作流）** | 你写的代码 | 先分类 → 再按类别调不同 prompt → 汇总 |
| **Agent（智能体）** | 模型自己 | 模型看完结果，决定再搜一次还是直接回答 |

Anthropic、OpenAI、Google 三家对这条分界线的说法基本一致。Google 补了一句：这不是二选一，而是一条**光谱**，中间有各种混合形态。

### 2.2 一句话定义

> **Agent = 模型在一个循环里使用工具，每一步拿环境的真实反馈来决定下一步。**

写成代码，就是 12-Factor 和 HW1 都给出的那个骨架：

```python
messages = [system_prompt_with_tools, user_message]
for turn in range(MAX_TURNS):            # 停止条件：最大步数
    reply = lm(messages)                  # 模型：messages 进，字符串出
    call = parse(reply)                   # 你的代码：从文本里解析出工具调用
    if call is None:
        return reply                      # 没有工具调用 = 最终答案
    result = registry.execute(call)       # 你的代码：执行
    messages += [reply, result]           # 把真实结果喂回去
```

注意：**模型只负责"说"要做什么，真正执行的永远是你的代码。** 这就是 12-Factor 第 4 条「工具就是结构化输出」。

### 2.2.1 agent 的五个部件（L01 讲义）

L01 用 ReAct 的一条轨迹把 agent 拆成五块：

| ReAct 轨迹里的 | 对应部件 | 在上面那段循环里是 |
|---|---|---|
| 模型本身 | LLM 核心 | `lm(...)` |
| `Thought: ...` | 规划与推理 | 模型回复里的文字部分 |
| 到目前为止的轨迹 | 记忆 | `messages` 列表 |
| `Act: Search[...]` | 工具 | `parse` + `registry.execute` |
| `Obs: ...` | 环境 | 工具返回的 `result` |

讲义里的 Apple Remote 例子值得细看：

- 只推理（CoT）的模型凭记忆答成了「iPhone、iPad」，是幻觉。
- ReAct 先搜 Apple Remote，得知它原本控制的是 Front Row；再搜 Front Row，没找到；换成 Front Row (software) 再搜，才查到正确答案「键盘功能键」。

关键在于**第 2 步搜索失败后，它根据观察换了查询词**。这就是「每一步拿环境的真实反馈」的意思。

L01 还拆了一次工具调用的四步，HW1 的 P5、P6 就是在实现这四步：

**选工具**（从上下文里的 schema 挑）→ **填参数**（按类型从任务里填）→ **校验**（schema 检查）→ **执行**（在沙箱里跑，结果回到上下文）

### 2.2.2 agent 工程的三层（L01 讲义）

不管部件多复杂，agent 工程只分三层：**系统**（架构）、**数据**（示例、示范、上下文）、**评测**（含指标）。后面每一课都要问自己：这一课在改哪一层？

### 2.3 agent 本质上是"上下文策展"

Google 白皮书有一句很准的话：agent 就是一个反复「组装上下文 → 调模型 → 看结果 → 再组装上下文」的循环。

模型是无状态的，每一次调用它只知道你这一次塞进去的东西。所以造 agent 的大部分功夫，其实花在**每一步往上下文里放什么、不放什么**。后面第 5 课的压缩和记忆都是在解决这个问题。

L02 讲义最后一节给了四条实用建议，你在 HW1 里都会用到：

1. **别臃肿**。Pi 这个编码 harness 只有 read、write、edit、bash 四个工具。
2. **用检索**找回最相关的内容，而不是全塞进去（上下文越长，注意力越差，叫 context rot）。
3. **只往后追加，不要改前面的内容**。前面一改，提供方的提示缓存就失效了，又贵又慢。HW1 也提醒：系统提示保持不变，缓存才能生效。
4. **用评测来比较**不同的上下文方案，不要凭感觉。

L02 还讲了「结构化 I/O」：系统里的组件之间必须有一份共同的契约。HW1 的 LM 只返回字符串，所以这份契约（输出格式、怎么解析）得你自己设计。这正是 P1 要练的东西。

### 2.4 什么时候才值得用 agent

三家的建议都是：**先用最简单的方案，只有确实变好了才加复杂度。**

OpenAI 列了值得建 agent 的三种情况：

- 需要细腻判断的复杂决策
- 规则多到难以维护
- 大量非结构化数据（自然语言、文档）

不属于这三种的，用确定性代码就够了。Anthropic 还提醒了代价：agent 用延迟和成本换效果，而且错误会一步步累积。

### 2.5 能力分级（Google）

| 级别 | 是什么 | 我们会不会做 |
|---|---|---|
| L0 | 纯模型，没有工具 | 第 1 课 |
| L1 | 接上工具，能查实时信息 | 第 3–4 课 |
| L2 | 会规划多步，会管理上下文 | 第 4–5 课 |
| L3 | 多个 agent 协作 | 只读不做 |
| L4 | 自己造工具、自己进化 | 不做 |

## 3. 想：思考题

把答案写在本文件末尾，或者直接发给我。

**题 1（workflow 还是 agent）**：EQB 组卷任务是「给高二出一套阅读理解练习，中等难度，4 篇」。把它拆成步骤，标出每一步应该是**写死的代码**，还是**交给模型判断**，并说明理由。

**题 2（找反例）**：EQB 里哪些功能**不应该**做成 agent？用 OpenAI 那三条标准说明。

**题 3（读代码）**：上面 2.2 那段循环，如果模型连续 3 次输出的工具调用格式都不对，会发生什么？应该怎么改？提示：HW1 的 P7 有一个 `malformed` 状态。

**题 4（L01 的挑战）**：L01 列了五大挑战：可靠性、训练、长程、安全、评测。其中「可靠性」又拆成三点：同一任务跑两次结果是否一致、小扰动下是否稳定、失败时是否失败得清楚、能恢复。EQB 组卷 agent 最怕哪一种？举一个具体的失败场景。

## 4. 做：HW1 环境 + P1

### 4.1 准备（第 0 课）

1. 申请一个 [OpenRouter](https://openrouter.ai) 的 API key。HW1 的 LM 包装就是走 OpenRouter 的。先充一点钱，按 HW1 自己的估计，所有实时测试跑一遍大约 $0.25。
2. 在你的电脑上装 [uv](https://docs.astral.sh/uv/getting-started/installation/)。
3. 克隆 HW1：

```bash
git clone https://github.com/cs329z/assignment1-harness.git
```

```bash
cd assignment1-harness && cp .env.example .env
```

4. 编辑 `.env`，填 `OPENROUTER_API_KEY`，并把 `CS329Z_BUDGET_USD` 设成 `2`（自己给自己加一道花钱上限）。
5. 解压数据，跑一遍测试。这时候应该全部失败，报 `NotImplementedError`：

```bash
uv run python data/download.py
```

```bash
uv run pytest
```

### 4.2 P1：给邮件判优先级

先读这几个文件：

- `cs329z_hw1/adapters.py` 里的 `run_priority`
- `cs329z_hw1/llm.py`，重点看 `LM` 和 `ScriptedLM`
- `data/priority_rubric.md`
- `tests/test_priority.py`

要求：一次模型调用，返回 `{"category": "urgent"|"normal"|"ignore"|"unknown", "reason": str}`。

你要做的三个设计决定：

1. **prompt 怎么让模型按固定格式回答？** 比如要求最后一行写 `LABEL: urgent`。
2. **解析器怎么做到稳？** 测试会喂各种乱七八糟的回复。找不到标签就返回 `unknown`，不能抛异常。
3. **rubric 放多少进 prompt？** 实时测试用的是另外 60 封邮件，只贴合公开样例的 prompt 过不了。

顺序：先让确定性测试变绿（免费），再跑实时测试：

```bash
uv run pytest tests/test_priority.py
```

```bash
CS329Z_EVAL_SLICE=5 uv run pytest tests/test_priority.py -m live -s
```

做完把你的 `run_priority` 代码贴给我，我帮你看。

## 5. 迁移到 EQB

P1 的模式是「一次调用 + 结构化解析 + 兜底」，在 EQB 里有直接对应：给一道题，一次调用判出题型和难度档。

这一课先不写 EQB 的代码，只需要想清楚：如果要做，**`unknown` 在 EQB 里对应什么？** 判不出来的时候，应该交给老师，而不是瞎猜。

## 6. 记

做完以后，在 `notes/` 里新建 `design-decisions.md`，记下 P1 的三个设计决定，以及为什么这么选。这就是 HW1 最后那份设计备忘录（P17）的草稿。
