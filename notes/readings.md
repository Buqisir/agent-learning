# 阅读笔记

按 CS329Z 课表顺序组织。每篇：核心观点 → 对我们这个 EQB agent 的启发。

## Anthropic《Building Effective Agents》（CS329Z L06 指定阅读）
- 统称 agentic systems：workflow（代码写死路径）vs agent（模型自己决定流程和工具）
- 基本积木：augmented LLM = 模型 + 检索 + 工具 + 记忆
- 5 个 workflow 模式：prompt chaining（加 gate）、routing、parallelization（sectioning / voting）、orchestrator-workers、evaluator-optimizer
- agent = 模型在循环里用工具，每步从环境拿真实反馈；要有停止条件（最大步数）
- 原则：先用最简单的；先直接调 API 再考虑框架；设计简单、规划透明、认真设计 ACI（工具文档）
- 附录：工具要像 prompt 一样打磨；poka-yoke（防呆，例：强制绝对路径）
- 对 EQB：组卷大部分是 workflow，只有"挑哪些题、凑够难度分布"需要模型判断

## OpenAI《A Practical Guide to Building Agents》
- agent 定义：用 LLM 控制流程执行、知道何时完成、失败时交还给用户；动态选工具，在护栏内
- 何时建：复杂判断、难维护的规则、大量非结构化数据；否则确定性方案就够
- 三件套：model / tools / instructions
- 选模型：先用最强的建基线（要有 eval），再试着换小的省钱
- 工具三类：data（读）、action（写）、orchestration（agent 当工具）
- 指令：用现成文档改写、拆小步、每步对应明确动作、写边界情况
- 编排：先单 agent 加工具；run = 循环直到退出条件（最终输出 / 无工具调用 / 错误 / 最大轮数）
- 拆多 agent 的信号：prompt 里 if-else 太多；工具相似重叠（不是数量）
- 多 agent：manager（agent 当工具）vs decentralized（handoff）
- 护栏分层：相关性、安全分类、PII、moderation、工具风险分级（读/写、可逆、金额）、规则（长度/正则/黑名单）、输出校验
- 人工介入两个触发：超过失败阈值；高风险/不可逆动作
- 对 EQB：工具风险分级直接可用——查题=低，保存试卷=中，发布/删除=高需确认

## Google《Introduction to Agents》（Kaggle 5 篇系列第 1 篇，2026-05 版）
- agent = 模型 + 工具 + 编排层 + 部署服务，"LM 在循环里用工具完成目标"
- 一句话最好：agent 是"上下文窗口策展"系统——组装上下文 → 调模型 → 观察 → 再组装
- 5 步循环：拿任务 → 扫描环境 → 想 → 做 → 观察并迭代（Think-Act-Observe）
- 能力分级：L0 纯模型；L1 接工具；L2 战略规划 + 上下文工程；L3 多 agent；L4 自我进化（自己造工具/agent）
- 选模型看你自己的业务指标，不看通用榜单；可多模型路由
- 工具：检索（RAG / NL2SQL）、动作、HITL 工具（ask_for_confirmation）
- 编排层：自主度是光谱；框架要开放、可控（硬规则管住模型）、可观测（trace）
- 短期记忆 = 本次的 (动作, 观察) 序列；长期记忆 ≈ 另一个检索工具
- Agent Ops：先定 KPI；LM-as-judge + golden dataset；指标驱动发布；OpenTelemetry trace 查"为什么"；用户差评 → 变成新测试用例
- 安全：效用 vs 风险；双层防御 = 确定性护栏（代码硬规则）+ 推理型防御（guard model）；agent 要有自己的身份和最小权限
- 对 EQB：我们目标先到 L1→L2；L3/L4 不做

## Google《Agent Tools & Interoperability with MCP》（Kaggle 第 2 篇，配 CS329Z L04）
- 工具两类：让模型"知道"（检索）/"做"（动作）。工具定义 = 模型与工具之间的契约：名字、参数、自然语言描述
- 三种工具：function tool（自己写）、built-in（模型服务自带）、agent tool（agent 当工具，主 agent 保持控制）
- 工具文档最佳实践：名字清楚具体；输入输出都描述；参数少；加例子；写默认值
- 指令里描述"做什么"而不是"用哪个工具"；不要重复工具文档；不要规定死流程；但要说明工具之间的副作用
- **发布任务，不是 API**：工具 = agent 要完成的一个具体任务，不是后端 API 的薄封装
- 工具要细粒度、职责单一；输出要精简（大结果存外部，返回引用）；用 schema 校验输入输出
- **错误信息也是给模型的指令**："没找到 X，请让用户确认名字后再按名字查 ID"
- MCP：解决 N×M 集成问题；Host / Client / Server；JSON-RPC 2.0；stdio（本地）/ Streamable HTTP（远程）
- 原语：server 端 Tools / Resources / Prompts，client 端 Sampling / Elicitation / Roots；实际上只有 Tools 被广泛支持（99%）
- 工具定义字段：name、title、description、inputSchema、outputSchema、annotations（readOnlyHint / destructiveHint / idempotentHint / openWorldHint——只是提示，别信不可信的 server）
- 两种错误：JSON-RPC 协议错误 vs 结果里 isError: true（业务错误）
- 风险：工具太多撑爆上下文 → 未来可能"先检索工具再加载"；动态能力注入、工具影子（恶意描述抢调用）、恶意工具定义/返回内容（prompt injection）、敏感信息泄漏、权限粒度粗、confused deputy
- 缓解：工具白名单、钉版本、输入校验、输出清洗、HITL 管高风险 sink、最小权限、凭证不进上下文
- 对 EQB：工具设计成 search_questions / get_question / draft_paper 这种"任务"，不是把 /api/question-groups 原样暴露；EQB 已有 ops-mcp（Streamable HTTP、只读）可当范例

## Google《Context Engineering: Sessions & Memory》（Kaggle 第 3 篇，配 CS329Z L02 末尾 + L07）
- LLM 无状态；context engineering = 每一轮动态组装整个上下文（prompt engineering 只管静态系统指令）
- 目标："不多不少，刚好最相关"
- 上下文三类：引导推理（系统指令、工具定义、few-shot）；证据（长期记忆、RAG、工具输出、子 agent 输出、文件）；当前对话（历史、scratchpad 状态、用户问题）
- context rot：上下文越长注意力越差 → 需要压缩
- 每轮循环：取上下文 → 组装 → 调模型/工具 → 上传新信息（后台）
- **Session = 工作台**（本次对话的 events + state），**Memory = 档案柜**（跨会话、提炼过的）
- session history（完整原始记录）≠ context（本轮发给模型的那份）
- 多 agent：共享历史 vs 各自私有历史（agent-as-tool / A2A）
- 压缩策略：保留最近 N 轮；按 token 截断；递归摘要（后台做、持久化）；触发：按数量 / 按空闲时间 / 按任务完成
- 记忆 vs RAG：RAG 让 agent 成为"事实专家"（共享、静态），记忆让它成为"用户专家"（按用户隔离、动态）
- 记忆类型：陈述性（knowing what）vs 程序性（knowing how，playbook）
- 组织：collections / 结构化用户档案 / 滚动摘要
- 生成 = LLM 驱动的 ETL：摄入 → 抽取过滤 → 合并（UPDATE / CREATE / DELETE）→ 存储
- 来源可信度（provenance）：预置数据 > 用户显式输入 > 对话隐式推断；工具输出不建议做记忆（容易过期）
- 取记忆：相关性 + 新近度 + 重要性 混合打分；主动预载 vs memory-as-a-tool
- 放哪：稳定的（用户档案）放系统提示；临时的放对话里或用工具取
- 评测：生成质量（precision / recall）、检索（Recall@K、延迟）、端到端任务成功
- 安全：按用户隔离、PII 先脱敏再存、防记忆投毒
- 对 EQB：我们的 v3 记忆先做最简单的"一个 markdown 文件 = 老师偏好档案"（如：年级、常用难度、题型偏好），放系统提示；不存学生数据

## Google《Agent Quality》（Kaggle 第 4 篇，配 CS329Z L13 / L15 评测）
- 质量是架构支柱，不是最后一道测试；三句话：**轨迹即真相**、可观测是地基、评测是持续循环
- agent 失败不是崩溃，而是"200 OK 但答错了"：幻觉、偏见、漂移、钻规则空子
- 传统测试问"做对了吗"（verification），agent 评测问"做的是对的东西吗"（validation）
- 四支柱：Effectiveness（达成目标）、Efficiency（token / 时间 / 步数）、Robustness（API 超时、数据缺失、模糊请求时优雅失败）、Safety
- 由外向内：先黑盒（任务成功率、用户满意度），失败了再开玻璃盒看轨迹
- 轨迹逐项查：规划（幻觉、循环）→ 选工具/填参数 → 理解工具返回（**没认出错误还当成功**）→ RAG → 效率 → 多 agent
- 评审者：自动指标（ROUGE / BERTScore，只看趋势）；LLM-as-judge（**优先用两两对比**算胜率，而不是 1-5 打分）；agent-as-judge（评整条轨迹）；人工（定 rubric、做 golden set）
- 安全评测：红队、过滤 + 人工复核
- 可观测三支柱：日志（结构化 JSON；**动作前记意图、动作后记结果**）、trace（span + 属性 + trace_id 串因果链）、指标（系统指标：延迟、错误率、token、成本；质量指标：正确性、轨迹符合度、帮助度）
- 采样：生产环境成功请求抽 10%，错误 100% 记录；日志入库前去 PII
- 质量飞轮：定义质量 → 埋点 → 评过程 → 失败回流进 golden set
- 对 EQB：v5 的评测集 = 10~20 个固定组卷请求 + 期望（题型、数量、难度区间）；先用代码检查硬约束，再用 LLM 两两对比软质量

## Google《Prototype to Production》（Kaggle 第 5 篇）
- "最后一公里"：约 80% 的工作量在基础设施、安全、验证，不在 agent 的"聪明"本身
- agent 特有难点：轨迹每次不同、状态（session/记忆）、成本和延迟不可预测
- 三支柱：自动评测、自动部署（CI/CD）、全面可观测
- **评测门控发布**：任何版本没过评测不上线；起步可以手动（PR 里附评测报告），成熟后进 CI 自动拦
- CI/CD 三段：合并前（快检 + 评测集）→ 合并后预发布（压测、内部试用）→ 生产（人签字）
- agent 是复合体：代码 + prompt + 工具定义 + 配置，**全部要版本化**（含评测集），才能一键回滚
- 灰度：canary、蓝绿、A/B、feature flag
- 安全三层：策略/系统指令（宪法）→ 护栏（输入过滤、输出过滤、HITL 升级）→ 持续评测与红队
- 生产循环 Observe → Act → Evolve；事故处置：熔断（flag 关掉某个工具）→ 人工排查 → 修复走 CI
- 可靠性：工具要能安全重试（幂等），防重复写入
- 生产失败 → 变成评测集里的新用例
- MCP 是"做这件具体的事"，A2A 是"完成这个目标"（agent 对 agent）；注册中心等规模大了再建
- 对 EQB：EQB 自己已经有 Ship 流程和 CI；我们 agent 的版本化 = git 管 prompt + 工具 schema + 评测集

## HumanLayer《12-Factor Agents》
- 核心判断：好的生产级 agent 大部分是普通软件，LLM 只在关键点上；用框架常卡在 70-80% 质量，想再往上就得拆框架重写
- 基本循环：LLM 决定下一步（输出结构化 JSON）→ 确定性代码执行 → 结果追加到上下文 → 重复直到 done
- F1 自然语言 → 工具调用（结构化对象）
- F2 自己掌握 prompt（当代码管，能测、能改、能看见）
- F3 自己掌握上下文窗口（可以不用标准 messages 格式，自定义 XML/YAML 事件流，信息密度更高；已解决的错误可以从上下文里删掉）
- F4 工具就是结构化输出：模型决定"做什么"，你的代码决定"怎么做"
- F5 执行状态和业务状态合一：整个 thread（事件列表）就是唯一状态 → 好序列化、好恢复、好分叉、好调试
- F6 简单 API 启动 / 暂停 / 恢复；**尤其要能在"选好工具"和"执行工具"之间暂停**（审批）
- F7 用工具调用联系人类（request_human_input 是一种 intent），配合 webhook 恢复
- F8 自己掌握控制流：有的 intent 同步继续，有的跳出循环等人（澄清、高风险审批）；在这里加摘要、缓存、judge、限流、日志
- F9 把错误压缩进上下文让模型自愈，但要有连续错误计数（约 3 次）就停/升级
- F10 小而专的 agent：3-10 步，最多 20 步
- F11 从任何地方触发（Slack、邮件、cron）
- F12 agent = 无状态 reducer：(state, event) → new state
- F13 预取：如果几乎肯定要调某工具，就直接确定性地调用，把结果塞进上下文，省一轮
- 对 EQB：我们的 v1-v2 就照 F4/F5/F8/F9 写——thread = 事件列表，switch 分发，错误计数，高风险 intent 跳出循环等老师确认；F13：组卷前直接预取"可用题型和数量"

## Anthropic《Effective harnesses for long-running agents》（CS329Z L18 附加阅读）
- 问题：agent 按会话工作，每个新会话没有记忆；压缩（compaction）不够
- 两种失败：一次想做完（做到一半上下文没了，留下半成品）；过早宣布完成
- 解法：初始化 agent（只跑一次：写 feature list JSON（全部 failing）、init.sh、progress 文件、首次 commit）+ 编码 agent（每次只做一个 feature，测完才改 passes，commit + 写进度）
- feature list 用 JSON 不用 Markdown：模型不太会乱改 JSON；只允许改 passes 字段
- 每次开工：pwd → 读 git log 和进度文件 → 挑最高优先级未完成项 → 跑 init.sh 和一次端到端冒烟测试，先修旧 bug 再做新功能
- 要求像真实用户一样端到端测试（浏览器自动化），单元测试/curl 不够
- 对我们：这正是 EQB 仓库 AGENTS.md / spec / ticket 那套的来源；我们自己学习也用 progress.md 记录每课进度

## Anthropic《How we built our multi-agent research system》（CS329Z L08 多智能体）
- 研究类任务路径不可预测 → 适合 agent；搜索的本质是压缩，子 agent 各有上下文并行探索再浓缩
- 多 agent（Opus 主 + Sonnet 子）比单 agent 好 90.2%；token 用量解释 80% 的性能差异
- 代价：agent ≈ 4× 聊天 token，多 agent ≈ 15×；需要共享上下文、依赖多的任务（多数编码）不适合
- 架构：orchestrator-worker；主 agent 把计划存进记忆（防上下文超 200k 被截）；最后 CitationAgent 加引用
- prompt 原则：像 agent 一样思考（模拟看它一步步做）；教会委派（目标、输出格式、工具、边界）；按复杂度定投入（简单 1 个 agent 3-10 次调用）；工具描述要清楚；让模型自己改工具描述（完成时间 -40%）；先宽后窄；用 thinking 做规划；并行
- 评测：**马上用约 20 个真实查询开始**，早期效果大不需要大样本；单个 LLM judge 输出 0-1 分 + 通过/不通过 最一致；人工测试发现 SEO 内容农场偏好
- 生产：错误会累积 → 断点续跑 + 重试；全链路 trace；彩虹部署；同步执行是瓶颈
- 附录：会改状态的 agent 用终态评测；长对话：阶段性总结存外部记忆；子 agent 输出写文件只传引用
- 对 EQB：我们不做多 agent。只记住"20 个查询就能开始评测"

## Anthropic《Building a C compiler with a team of parallel Claudes》
- 16 个 Claude 并行、约 2000 次会话、$20k，写出 10 万行能编 Linux 的 Rust C 编译器
- harness 就是一个 bash `while true` 循环；协调只靠 git + current_tasks/ 下的锁文件，没有 orchestrator
- **验证器必须近乎完美**，否则 agent 会解决错的问题；后期新功能老是破坏旧功能 → 加 CI 拦回归
- 为模型设计测试输出：输出少、细节写日志、错误行带 ERROR 方便 grep、预算统计；--fast 随机抽 1%/10%（按 agent 固定种子）
- 能并行的前提是任务可拆；Linux 内核是一个大任务 → 用 GCC 当 oracle 拆成多文件子任务
- 角色专门化：去重、性能、生成代码质量、设计评审、文档
- 风险：测试通过 ≠ 完成；作者对"没人亲自验证过的代码上线"感到不安
- 对 EQB：EQB 那边的 CI/Ship 流程就是这个思路；我们的评测集就是 agent 的"验证器"，要先把它做对

## CS329Z HW1《Building an Agentic Harness》（课程的核心实践，MIT 协议，公开）
- 不许用 agent 框架；LM 包装只给一个原语：messages 进、字符串出（没有原生 tool calling）→ 工具协议自己设计
- LM 包装：磁盘缓存（同样调用免费）、成本账本、预算上限（BudgetExceeded）、OutputTruncated
- 两类测试：确定性测试（ScriptedLM 回放固定回复，不花钱，占 48/88 分，定义了必须处理的错误）+ 实时测试（真模型，有阈值）
- Part 1 流水线：P1 单次调用分类（unknown 兜底）；P2 多次调用（并发分类 → 代码聚合 → 一次调用写摘要，≤300 词）；P3 从零写 BM25（倒排索引，Lucene IDF，k1=1.5，b=0.75，20ms/查询）；P4 多跳 RAG（代码掌控循环，最多 3 次搜索）
- Part 2 Cardinal Agent：
  - P5 工具注册表：schemas / validate / execute；**validate 和 execute 分开**（先拒坏调用，再问审批）；状态 ok / unknown_tool / invalid_args / tool_error（后加 denied）
  - P6 工具调用协议：render_tools / parse_response / format_tool_call；每次最多一个调用；截断时要么没调用、要么完全正确
  - P7 循环：done / max_turns / token_budget / malformed（连续 3 次格式错）；预算在每次调用前检查；工具结果原样出现在下一次调用；transcript 只追加
  - P8 组装 10 个工具；懒加载索引
  - P9 检索当工具：返回指针 + 片段（≤150 token），按窗口读（≤1500 token）；有被后来公告取代的旧政策
  - P10 终端：沙箱、白名单、超时；大输出的观察策略（截断头尾 / 写文件告诉路径）
  - P11 压缩：每次调用都不超 context_budget；先丢旧工具结果，不够再让 aux_lm 写摘要；用户说过的约束必须保留
  - P12 记忆：remember / recall 两个工具，按目录隔离
  - P13 交互：confirm / auto 模式；非只读工具要审批；每次调用前检查用户插话（steering）；ask_user 工具
  - P14 护栏：deny 正则；**工具输出不能改模式/规则**；文档和邮件里埋了针对 AI 的注入指令
  - P15 模拟用户评测：20 个 persona，程序检查 + judge 都过才算成功
- 设计备忘录 10 问（协议、终止规则、检索窗口、观察策略、压缩、记忆、审批、沙箱、局限）= 我们每课的"设计决定"
- 对 EQB：这份作业的结构就是我们 EQB agent 的蓝图——先完整做一遍 HW1，再把同一套 harness 换成 EQB 的工具和数据

## CS329Z L01 讲义《Intro to Agentic Systems》（71 页）
- 课程定位：同时需要"模型视角"（训练、prompt、对齐、评测）和"系统视角"（工具、API、环境、基础设施）
- 一张值得记住的片子："技能会退化"——Anthropic 2026 年的随机对照实验，用 AI 辅助学新库的工程师，后测低 17%，调试能力差距最大。课程的态度是"用 AI 学会怎么学"
- 历史：agens（行动者）→ 1973 Actor → 1990s Maes / Wooldridge & Jennings（自主、社交、反应、主动）/ Russell & Norvig（通过传感器感知、通过执行器行动）→ RL 时代（Sutton & Barto 的 agent-环境循环、DQN）→ LLM 时代（GPT-3 上下文学习、MRKL、ReAct、ChatGPT、AutoGPT）
- **agent 解剖（以 ReAct 为准）**：模型 → LLM 核心；Think → 规划与推理；到目前为止的轨迹 → 记忆；Act → 工具；Obs → 环境
- 推理对 agent 的作用：观察到行动的映射很难，推理能帮助泛化和对齐（"盐用完了 → 推理 → 去右边柜子找酱油"）
- ReAct vs CoT：只推理会幻觉（Apple Remote 的例子答成 iPhone/iPad），推理+行动能查证纠错；还讲了 Self-Consistency、Reflexion（把失败轨迹提炼成自我提示）、多 agent 辩论、Magentic-One 编排
- 记忆三类（按内容）：情景（经历，追加事件流、启发式打分取回，Generative Agents）、语义（知识，LLM 对事件推理后写入）、程序（技能，代码形式，Voyager）
- 工具：Toolformer（模型自学何时调什么 API）、Gorilla（海量 API）
- **工具调用解剖**：选择（从上下文里的 schema 挑工具）→ 填参数（按类型从任务里填）→ 校验（schema 检查 / 约束解码）→ 执行（沙箱，结果回到上下文）
- **agent 工程三层**：系统（架构）、数据（训练样例、示范、上下文）、评测（含指标）
- workflow vs agent：都是复合 AI 系统；workflow 由代码定步骤（Agentless、AlphaCode 2），agent 由 LLM 定步骤（SWE-agent）
- 五大挑战：可靠性（错误累积）、训练（稀疏奖励、长程、rollout 贵）、长程（上下文增长和漂移）、安全（任务成功 ≠ 安全）、评测
- 可靠性 ≠ 能力：一致性（同一任务两次结果一样吗）、鲁棒性（小扰动）、可预测的失败方式（失败得清楚、能恢复）——HAL 可靠性面板
- 安全案例：弹窗攻击成功率 87%、PrivacyLens、谄媚 / reward tampering、多 agent 合谋（2026 年 OpenAI agent 攻击 Hugging Face 事件）

## CS329Z L02 讲义《LLMs for Builders》（176 页）
- 课程在 LLM 技术栈的最上层（agent），但要懂一点底层
- LM = 逐 token 条件概率的乘积；生成式 → 可以采样；采样策略：贪心、top-p、top-k、温度、beam search；数学/代码用低温，创作用高温
- 架构：Transformer 注意力 O(n²)、KV cache 随长度增长；线性注意力 O(n)、状态恒定但长上下文差；实际是混合（如 Kimi K3）；上下文长度上限来自训练时的位置编码
- 对 agent 的意义：推理效率是设计约束；了解模型长短处，按它来搭 harness
- 训练：预训练（数据清洗、去重、配比）→ 中训练（加重目标领域）→ 后训练（指令微调、RLHF、RLVR 可验证奖励——单测、Lean 证明、精确匹配、系统状态；**验证器设计要小心**）→ agent 专项训练（SWE-smith）
- 推理：prefill（算力瓶颈）/ decode（带宽瓶颈）；投机解码；推理时扩展（CoT、推理模型、推理强度 low/medium/high、重复采样、没有验证器时用 self-consistency）
- **LLM 用于 agent**：
  - 结构化 I/O：组件之间需要共同的契约；约束解码（SGLang 有限状态机）；DSPy Signature（用 `[[ ## 字段 ## ]]` 标记）；分类头
  - 上下文里放：系统提示、消息历史、记忆、工具结果、附件
  - 为什么不全放：context rot
  - 实用建议：**避免臃肿**（Pi 只有 read / write / edit / bash 四个工具）；用检索找回最相关的；**按顺序追加，不要改前面的内容**（否则 KV cache / 提示缓存失效）；用评测实证地比较上下文方案；压缩时的取舍——留多了很快又要压，丢多了任务做不完
  - 另一种思路：递归 LM（RLM）
  - "没有唯一正确的上下文构建方式，HW1 里你会自己实验"
