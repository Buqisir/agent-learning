# 第 1 课：agent 到底是什么

对应 CS329Z L01「Foundations & Landscape」和 L02 后半「context engineering」。

## 1. 读

先读这些，按顺序：

1. Anthropic《Building Effective Agents》全文。约 15 分钟。整门课的骨架都在这篇里。
2. OpenAI《A Practical Guide to Building Agents》，读到 "Agent design foundations" 为止。
3. Google《Introduction to Agents》，读前三节：定义、5 步循环、Level 0–4。

选读，都是 CS329Z L01 指定的：

- BAIR《The Shift from Models to Compound AI Systems》
- Lilian Weng《LLM Powered Autonomous Agents》
- Kapoor 等《AI Agents That Matter》，讲评测为什么必须算成本

## 2. 懂：五个核心概念

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

### 2.3 agent 本质上是"上下文策展"

Google 白皮书有一句很准的话：agent 就是一个反复「组装上下文 → 调模型 → 看结果 → 再组装上下文」的循环。

模型是无状态的，每一次调用它只知道你这一次塞进去的东西。所以造 agent 的大部分功夫，其实花在**每一步往上下文里放什么、不放什么**。后面第 5 课的压缩和记忆都是在解决这个问题。

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
