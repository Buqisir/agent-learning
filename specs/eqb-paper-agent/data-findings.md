# 数据摸底：B 篇主题标签覆盖率（2026-11-04）

## 能查到什么、查不到什么

- **真题库 `data/question-bank.sqlite` 不在 git 里**（被 gitignore），云端拿不到。主题标签存在这个库的 `question_group_topic_labels` 表里。
- 仓库里能查到的有三样：
  - `app/public/paper-assets/` 下 35 份已发布的真题卷
  - `docs/STATUS.md` 里的历史记录
  - 主题分类表 `shared/topics/topicTaxonomy.mjs`

## 发现

1. **历史记录写的是全覆盖。** `docs/STATUS.md` 写着"topic label 已实测 273/273 全覆盖"（2026-07 中旬）；2026-07-17 补上了最后一卷 paper-2026-i。之后新导入的卷子有没有补标签，**不知道**。
2. **标签是 AI 打的，带置信度，一组可以有多个。** 例：paper-2026-i 的 B 篇有两个标签，"乐于学习"0.85、"家庭生活"0.60。
3. **分类表里没有"代际关系"这个子话题。** 分类表按课标分三层：主题语境 → 主题群 → 子话题。离"代际"最近的是"家庭生活"（人与自我）和"良好的人际关系"（人与社会）。所以老师说"代际关系"时，**没有一个标签能直接对上**，需要做一次意思上的匹配。这正是 agent 的价值所在，也是一个产品决定。
4. **35 份已发布真题的情况：**
   - 35 篇 B 篇，每篇都有 AI 写的内容概述（`analysis.groups.*.overview`）。
   - 用关键词粗筛"代际相关"的阅读，所有篇位一共只有 4 篇：
     - paper-2020-ii B：作者用虚构的"奶奶的盒子"管理学生玩具，后来真有了孙子
     - paper-2023-jia B：26 岁女孩的 DIY 手艺来自已故的祖父（算不算代际关系？有争议）
     - paper-2020-bing **C**：英国多代同堂家庭的兴起（最贴题，但它是 C 篇）
     - paper-2022-i **C**：养老院让老人照看母鸡（算不算代际？有争议）
   - 这 35 份都是高考真题，**不含模拟题**。

## 对规格的影响

- Scenario 1"4 篇高三模拟题 B 篇"：在已发布的数据里，模拟题这个条件就凑不出来。这正好可以拿来当"凑不够"的场景。
- 新的产品决定：**篇位要卡死吗？** 最贴题的多代同堂那篇在 C 篇。
- 新的产品决定：**"代际"的边界在哪里？** 祖孙传承手艺算不算？养老院算不算？需要老师给定义，或者给几个正例、几个反例。
- 关键词筛选既会漏掉真正相关的，也会混进不相关的，用来做验收不靠谱。验收应该用老师标注的"标准答案"：哪几篇算代际。

## 要在本地跑的只读查询

在你本地的 EQB 仓库根目录运行。这条查询只读不写。

```bash
sqlite3 data/question-bank.sqlite "SELECT COUNT(*) AS b_groups, SUM(EXISTS(SELECT 1 FROM question_group_topic_labels t WHERE t.question_group_id = g.id)) AS labeled FROM question_groups g WHERE g.group_type = 'reading' AND (g.title = 'Reading B' OR g.id LIKE '%-reading-b');"
```

```bash
sqlite3 data/question-bank.sqlite "SELECT t.subtopic, COUNT(*) FROM question_group_topic_labels t JOIN question_groups g ON g.id = t.question_group_id WHERE g.group_type = 'reading' AND (g.title = 'Reading B' OR g.id LIKE '%-reading-b') GROUP BY t.subtopic ORDER BY 2 DESC;"
```
