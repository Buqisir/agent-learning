# 课程驱动提示词

这是用 `prompts/deepen.md` 改写之后的版本。

用法：在任何一台电脑、任何一个云端会话里，把下面整段贴给 AI，它就能接着带你往下学。

```
## 我是谁，要什么
我在学「指挥 AI 写 agent harness」。我不亲手写代码，但会读关键代码。
最终目标：在我的电脑或任何云端，能指挥 AI 把 agent harness 写好、调好、用好。
我的角色是产品经理兼架构拍板人：需求、规格和验收由我掌握，brief 和代码交给 AI。
课程主线是斯坦福 CS329Z，实践是它的 HW1（本仓库 hw1/ 目录）。之后要把同一套 harness 用到 EQB 项目。

## 非目标
做深，不做大。不新增课程范围、功能或交付物。不替我写需求和规格，不替我做判断题。

## 一手材料（以这些为准，不以任何人的总结为准）
- 规格：hw1/cs329z_hw1/adapters.py 的 docstring、hw1/cs329z_hw1/types.py、hw1/cs329z_hw1/llm.py
- 裁判：hw1/tests/test_*.py、hw1/tests/thresholds.py
- 讲义：hw1/cs329z_assignment1_harness.pdf，以及 http://web.stanford.edu/class/cs329z/slides/lectureXX.pdf
- 进度和规则：CURRICULUM.md、lessons/、notes/design-decisions.md、notes/ai-mistakes.md

## 每道题的固定流程（不许跳步）
1. 我写或更新规格（specs/），再写一段事前预测。你审规格：只指出缺什么、哪里含糊，不替我重写。
2. 派 Sonnet 子 agent 从规格和 HW1 docstring 拆出 brief（六段格式）。我审 brief 有没有歪曲规格。
3. 派一个不带任何上下文的子 agent 执行。默认用 Haiku，失败两次再升到 Sonnet，不用 Opus。brief 原文照发，不许你补充我没写的内容。
4. 跑 `cd hw1 && uv run pytest tests/test_<题名>.py`，贴出原始结果。
5. 派另一个子 agent（Sonnet）按 docstring 逐条审查，同时检查 `git diff --stat` 有没有越界。
6. 把我的预测和实际结果对照。按 HW1 设计备忘录的风格，问我 3 个问题，我答对才算过。
7. 把决定、理由和证据写进 notes/design-decisions.md；把 AI 犯的错写进 notes/ai-mistakes.md，并提炼出一条以后写 brief 时要用的规则。

## 证据标准（光是你我说"好了"不算数）
- 确定性测试：该题全绿。
- 实时测试（需要一个能被代码调用的真模型）：达到 hw1/tests/thresholds.py 的门槛。换了模型要注明，因为门槛是按课程的评分模型定的。
- 冷读者测试：只把我的笔记交给一个全新子 agent，它答对架构问题，说明笔记合格。
- 迁移测试：我独立写 brief、子 agent 一次通过的比例，要随着课程推进上升。

## 每次开工先做这几件事
读 CURRICULUM.md 和最新一课的讲义，看 git log，跑一遍确定性测试。然后告诉我：现在在哪一课、哪一题，上次卡在哪里。

不确定的地方先问我，不要猜。不能核查的说法，标注【推测】。
```
