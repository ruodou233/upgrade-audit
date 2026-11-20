# AI/Agent 使用知识库

> 供用户阅读的 AI/Agent 方法与实证整理，Agent/skill 无需加载。
> **收录条件**：内容必须会改变模型、Agent、开发工具或工作方法的实际选择；社交内容需对象自身原生点赞不少于 10，官方文档、论文和正式演讲按内容价值判断。推送时说明它在说什么、为什么值得看，不只发标题。
> **分工**：本文收录来源、数据、模型/工具特定技巧与行业实证。
> **认知篇已拆分**：原"AI 认知与思考"（AI 本质、AGI 光谱、行业判断、组织与人）已整合为 `ai-cognition-perspectives.md`，仅供用户阅读，Agent/skill 无需加载。
> **时效说明**：快变段为 §2.2（模型族）和 §6（基准类结论）；产品功能不在这里维护静态清单。稳定段（工作流、心智模型、MCP 原则）不需反复核验。
> 整理自 2024–2026.9 的公开分享、官方研究、竞赛复盘与团队文档；只收会改变实际判断的方法或证据。

---

## 0. 心智模型校准

决定"何时信任输出、何时必须验证"的五个模型：

- **过度自信的实习生**（Willison）：LLM 像"读过世界上所有教科书但对你的代码库零实战经验、容易忘记最近一小时之外任何事的初级开发者"。"Over-confident"不能省——它不会说"我不确定"。
- **参差的能力边界**（jagged frontier）：AI 的能力不是均匀前沿线——某些任务超预期强，某些出人意料弱，不用就不知道边界在哪。实证（Dell'Acqua、Mollick 等人的 758 名 BCG 顾问实验[^bcg]）：用 AI 的顾问多完成 12.2% 任务、快 25.1%，但在"frontier 之外"的任务上给出正确解的比例比对照组**低约 19 个百分点**。含义：没确认 AI 擅长的任务，先小样本测。
- **插值器，不是创新器**（Howard）：LLM 擅长"在已知之间填空"，弱于分布外的概念创新。代理脚手架是把更多问题"挪进分布内"来 work 的。分布外任务降低期望。
- **幻觉两分类**（Lilian Weng）：in-context（输出与上下文源不一致）与 extrinsic（输出无世界知识支撑）。核验策略要分开：前者对照源文本，后者对照外部事实。
- **迎合机制**（Schulman）："模型只是想生成一条让人满意的消息。"它会揣摩你想听什么——所以不信口头断言、要求证据、用独立上下文的 Reviewer 对抗确认偏误。

配套使用原则：

- **Mollick 四原则**：①凡合法合规都用 AI 试一遍（边界只能试出来）；②关键决策点保留人类检查；③给 AI 人设但记住它是机器；④假设这是你用过最差的 AI。
- **指令遵从 > 偶发峰值**：工程场景中，可预测、稳定遵从指令的模型往往比偶尔超水平但时常自作主张的模型更好用。选型时把是否按边界完成任务当一等指标。

---

## 1. 工作流总框架

2025–2026 年多个独立来源（Cursor、Anthropic、宝玉、Shumer、Howard）收敛到同一循环：**研究 → 规划 → 执行 → 审查**（部分来源在此后另加交付/发布环节）：

- **先规划后编码**：Jeremy Howard 的 Dialogue Engineering 把这件事推到极致——"在生成代码前先打磨思维过程"，精心构造一段对话，对话的产出才是最终工件。
- **写测试让代理自验证**：Matt Shumer 的原话最硬——"先建测试框架再开始实现，每个测试通过前别完工——这些不是建议，是停止条件。"
- **人做审查者，不做打字员**：要求代理展示证据（测试输出、命令返回、截图），而非信它口头断言。Cursor："代理越快，你的审查越重要。"Fiona Fung（Anthropic 工程文化）：**shift verification left**——吞吐量上升后维持质量的唯一杠杆是把验证提前。
- **技术辩论中，代码赢**（Fiona Fung）：让 agent 把两个候选方案都原型化，比实际产物而非空谈设计文档。
- **会话卫生**：一个会话一类任务，避免"厨房水槽式会话"（Anthropic 语）。
- **人定目标，Agent 做实现**：Anthropic 对约 40 万次 Claude Code 会话的观察性分析发现，使用者承担更多规划决策，Claude 承担更多执行决策；领域经验与更有效的指令和更高成功率相关。它支持的做法是让人负责目标、路线和验收，让 Agent 承担具体实现，而不是把两者混成逐步遥控。[来源](https://www.anthropic.com/research/claude-code-expertise)

---

## 2. 提示词策略

### 2.1 结构化 Prompt（复杂/可复用任务）

李继刚的方法论：像写文章一样编写提示词，用标识符（`#`、`<>`）控制层级。常用模块：Role/Profile（角色背景）、Constraint（"帮模型剪枝，减少不必要分支计算"）、Definition（"减轻幻觉的小技巧，把关键概念清晰定义"）、Example（few-shot）、Goal/Workflow。

云中江树的 LangGPT 把这套方法模板化："归根结底还是结构化、模板化 Prompt 的性能好。"注意两类一致性：格式语义一致性（标识符功能前后统一）和内容语义一致性。

李继刚指出局限："在创意性要求较高的场景，结构化思想不适用"——创意的本质是输出的随机性。

宝玉的判断：简单任务不需要精心提示词；复杂任务把专业提示词当"解难题的数学公式"，充当"工作流经理"串联多步骤。

### 2.2 不同模型族的差异化策略（快变段，截至 2026-09）

换模型或模型升级后，先删掉为旧模型准备的提示补丁，再用代表性任务重测。对更强、指令更敏感的模型，短句说明自主推进、何时提问、是否使用子代理和验证强度；频繁停顿或过量测试时，先清理冲突和重复规则，不继续堆免责条款。OpenAI 在 2026-09-03 的 GPT-6 Astra 指南中明确列出这些适配点；具体模型行为仍按当期官方指南读取。[来源](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-6-astra)

### 2.3 进阶提示技术

- **Ultra-deep thinking**（Shumer）：引导模型做远超平常的验证——"独立用替代方法验证你的推理，把每个事实、推断、结论对照外部数据交叉检查……一旦你完全确信，刻意停下，强迫自己从头再审视一遍。"
- **Composition 原则**（Shumer，"最被低估"）："格式很重要，因为格式改变思维"——指定确切的输出形状。
- **给指令编号优先级**（AMBIG-SWE，ICLR 2026）：agent 默认非交互行为，不主动问澄清就直接执行，导致 resolve 率从 48.8% 降到 28%。修复：给指令编号优先级，明确告诉 agent 何时应暂停询问。

---

## 3. Coding Agent 实操

### 3.1 Cursor 十条原则（官方博客）

先用 plan mode → 困惑时开新会话 → 让代理自己获取上下文（别什么都 @）→ 与其无望地修不如回退并改指令 → 对重复出现的错误才加规则 → 先写测试让它能迭代 → 跑多个模型择优 → 仔细审查 diff → 给可验证目标（类型语言、linter、测试）→ 把代理当有能力的协作者。

YOLO 模式：让代理自己跑测试/build 直到验证正确。Cursor 2.0 支持最多 8 个代理用 git worktrees 并行隔离。

### 3.2 Anthropic 极简脚手架哲学

SWE-bench 上只给一个 prompt、一个 Bash 工具、一个 Edit 工具，"把尽可能多的控制权交给模型本身"。两个工程洞察：

- "应该像为人类设计工具界面那样为模型精心设计工具界面"
- "错误防护"工具——如强制绝对路径，防模型切目录后搞错相对路径

成本教训："很多成功的 run 要上百轮、>100k token……模型很顽强但很贵。"

### 3.3 Advisor Strategy 与模型成本路由（Code with Claude 2026，Brad Abrams）

**小执行模型负责大多数任务，只在难 case 调用大 advisor 模型。**"通过对 advisor 实际发送的 token 极度保守，以低得多的价格逼近旗舰级智能。"GitHub 已在其 agent 系统采用。

两条互补的成本判断：

- **路由按难度，不是一律省钱**：罗福莉团队实测重要研究任务降档不可行（"想切便宜的模型省钱，但发现真不行"，张小珺 #138）——难任务省模型是假节约。
- **比较模型成本按任务总成本，不按单 token 价格**（Freda Duan）：同一任务不同模型 token 消耗可差 10–100 倍，单价便宜的模型跑完可能更贵。

新模型上线时，同时比较模型档位与 reasoning effort；以任务成功率、总成本、时延和返工量选择，不沿用上一代的默认组合。

### 3.4 产品能力按当前环境读取

产品功能、模型名和 effort 档位变化太快，不在知识库维护功能清单。任务发生时直接读取当前宿主工具、官方文档或产品界面；长期知识只保留会跨版本成立的方法。

### 3.5 社区验证的高效模式

- **Writer/Reviewer 双 Agent**：一个会话写、另一个独立会话只读审查，显著优于自审。同逻辑适用于测试：一个写测试，另一个写代码让测试通过。
- **先访谈再执行**：复杂或模糊任务，让 Claude 先访谈你输出 SPEC.md，新会话基于 SPEC 执行。社区反馈：发现大量"自己都没意识到自己不知道的问题"。
- **大规模并行迁移**：先列出所有待处理文件，每个文件独立会话并行（`claude -p "Migrate $file..." &`）。
- **并行假设探索**：多个 idea 分给不同 subagent 同时做、交叉验证（罗福莉团队："十个 idea 交给不同 subagent 同时做"）。与并行迁移的区别：前者探索假设空间，后者机械分片。

### 3.6 LLM Wiki 模式（Karpathy）

让 agent 把杂乱源文档**增量编译成持久 Markdown wiki**（摘要、实体页、概念页、矛盾、交叉链接），而非每次 RAG 重新检索。这是"过去不可能、现在自然"的信息变换方式，也适合维护分层文档体系。

### 3.7 复杂任务拆分与状态文件（Anthropic 官方建议）

将复杂任务拆为子任务，用状态文件（progress.md、todos.md）维持跨会话上下文，每完成一个子任务即 git commit 并更新进度。关键设计：状态文件用结构化格式（JSON/JSONL），模型对结构化文件的覆盖更保守。可落地为 Initializer/Implementer 或 Commander/Worker 模式。
落地做法见开源的 [agent-orchestration](https://github.com/ruodou233/agent-orchestration)：长任务的目标与进度写进任务自己的交付物。

来源：https://code.claude.com/docs/en/best-practices（截至 2026-07）

---

## 4. 上下文与范式

### 4.1 从 Prompt Engineering 到 Context Engineering

Karpathy（2025-06）："在每个工业级 LLM 应用里，上下文工程才是用恰当信息填充上下文窗口的精妙艺术与科学。"

Goodside 早在 2023 年初就区分过 context engineering（为任务挑选和准备相关上下文）与 prompt programming（写清晰指令）；2025-11 他给出更精确的表述："提示词工程是操心用户该写什么，上下文工程是操心模型该读什么。这两者过去是同一回事……只有从后者中剥离出来，我们才能认真考虑前者，看看还剩下什么。"


### 4.2 自动化你能验证的，而非你能指定的（Karpathy，Sequoia Ascent 2026）

**传统软件自动化你能"指定"的东西；LLM 和 RL 自动化你能"验证"的东西。** 这解释了为什么 math/coding/tests 进步最快——有清晰可验证信号。姚顺宇的互补判断："95% 的白领工作其实比写代码简单，但大模型做不掉的关键在于没有一个好的信号来评判好坏。"

推论：**要让 AI 做得好，先想清楚"什么叫做好"**。有清晰验证标准，AI 才能自我迭代；没有验证标准的任务，AI 只能靠一次猜对。

EdgeBench 在 134 个真实任务、约 3.8 万小时交互中观察到：当环境持续返回可读的分数、错误和提示时，Agent 可以在多轮尝试中继续改进。这里的关键不是“多跑一会儿”，而是反馈能被下一轮利用；没有外部反馈的长运行不会自动变成学习。[来源](https://seed.bytedance.com/en/blog/edgebench-measuring-real-world-environment-learning-and-discovering-a-new-scaling-law)

### 4.3 品味（taste）是 agentic engineering 的核心竞争力（Karpathy）

代码能跑不等于设计对了——Karpathy 举例：让 Claude 做账户匹配，代码能跑但设计错了，"这是品味缺失，不是技术缺失"。人仍然对质量负责；瓶颈从写代码转移到判断什么值得做、哪个方案更好。

### 4.4 Harness 假设会腐化（戴雨森，2026）

Harness 对模型能力的假设随模型迭代变旧。常见症状：**过时的 few-shot 样板、为旧模型设计的防御性约束、基于旧模型弱点的 workaround**。定期审计 harness 假设是否仍成立，是维护 harness 价值的必要成本——这正是升级审计的核验项之一。

Boris Cherny 在 2026 年的正式演讲中提到，Claude Code 团队随着模型能力提升删掉了约 80% 的系统提示。可执行的结论是：模型升级时先做删减实验，只有移除后实测退化的约束才加回。[演讲](https://www.youtube.com/watch?v=qyPCVqFUyDo)

---

## 5. MCP 心智模型与用户侧安全

MCP 已成事实标准（主流各家原生支持）。心智模型：**不是"给 AI 加工具"，而是 AI 和外部世界之间的标准化合约**——类比 USB-C，切换底层模型不需重写工具集成。区分：MCP 连接工具和数据源（给 agent 装手）；A2A 连接其他 agent（让 agents 协作），两者互补。

用户侧安全：只连接可信 MCP server，按最小权限授权；按敏感级别划分工具区域，私密数据优先本地 server。

---

## 6. Eval 与基准（基准类数字为快变段，截至 2026-06）

### 6.1 Test-time Compute（跨赛事最强规律）

ARC（TTT+多候选投票）、AIMO（SC-TIR 多数投票）、SWE-bench（多 pass ensembling）、Kaggle（TTA）**全部独立验证**：test-time 阶段投入更多算力几乎总比换更大模型划算。

- **ARChitects**（ARC Prize 2024 冠军，私有集 53.5%[^arc]）：同一 LLM 既当生成器又当打分器，候选解施加数据增强后用负对数似然乘积聚合。
- **Augment**（SWE-bench 65.4%）：Claude Sonnet 3.7 做 solver 生成多候选 diff，o1 做多数投票选择器，带来 3–8% 增益（60.6%→65.4%）。（历史实验，模型代际已过时，多候选+投票的方法论仍适用）
- **Ryan Greenblatt**：GPT-4o 每题生成约 8000 个程序，解释器确定性验证——搜索量与准确率对数线性。

### 6.2 简单 > 复杂

**Agentless**：刻意禁止 LLM 自主决策，只用三步——分层定位、修复（采样多补丁）、验证（生成复现测试重排序），以 $0.34/题的成本，被 OpenAI 和 DeepSeek 采纳为标准评测方法。

**Harness 可以显著改变同一模型的表现。** OpenAI 在 ARC-AGI-3 的同模型对照中，同时保留推理历史并以 compaction 代替滚动截断，得分从 13.3% 提升到 38.3%，输出 token 约降六倍。它证明的是这两个设置在该任务上的联合效果，不能拆分归因或外推到所有任务。[来源](https://openai.com/es-419/index/how-two-settings-tripled-our-arc-agi-3-scores/)

### 6.3 Eval-Driven Development

Anthropic 黑客松获胜者方法论：先定评估标准再实现，每次改动用 eval 分数而非直觉 justify。Jake Heinrich（CoCounsel）流程：先问真实专业人士什么是好表现 → 每个 prompt 先做十几个 eval 迭代到完美 → 再加 50 个 → 发布前 100 eval/prompt → 每个用户投诉都变成新 eval。Kaggle 铁律"相信你的 CV"。

### 6.4 Agent Eval 体系

- **pass@k**（k 次至少一次通过）衡量能力上限；**pass^k**（k 次全部通过）衡量可靠性。生产环境更关心 pass^k——用户不会给你多次机会。
- **Agent demo 可用 ≠ 生产可靠**：从 90% demo 到生产，每提升一个可靠性"9"都要独立的 eval/回归验证支撑（Karpathy "march of nines"的操作面）。
- Grader 三类：确定性验证器（测试、类型检查）> LLM-as-judge（开放式输出，需校准）> 人工（贵但最可靠，用于 eval 的 meta-evaluation）。
- **eval 数据集本身会过时**，需定期审计 eval 质量——这是升级审计的固有职责。

### 6.5 基准怀疑主义（快变段，截至 2026-06）

- **报告分数必须说明 scaffold 条件**：同一基准的"最好成绩"常有多个数字同时流通（scaffold 与数据子集不同），引用任何分数不带条件即失真。
- **基准分是能力上界，不是真实工程能力**：METR 分析发现许多通过 SWE-bench 的 PR 过不了人工 review，模型在长任务上急剧退化；Augment 报告"改善生产 agent 的改动往往不动 SWE-bench 分数"——基准与真实产品体验脱节。
- **公开榜单只做初筛**：用榜单缩小候选范围，最终用自己的隐藏任务或盲测决定。Google DeepMind 2026-08 的双盲评测试点进一步说明，题目泄露和针对性优化本身也是 benchmark 风险。[来源](https://deepmind.google/blog/piloting-the-worlds-first-double-blind-ai-evaluations/)

---

## 7. 反直觉清单

> 均为特定环境实测结论，外推前先在自己场景验证。

### 有效到意外

| 方法 | 来源 | 为什么意外 |
|------|------|-----------|
| 暴力采样数千程序 + 确定性验证 | Greenblatt / ARC | 准确率随搜索量对数线性增长 |
| grep/find 替代 embedding 检索 | Augment / SWE-bench | embedding 在代码定位场景并非瓶颈 |
| 强制绝对路径的"错误防护" | Anthropic / SWE-bench | 一个小约束消除一类系统性错误 |
| Markdown 替代 JSON 作为 tool 输出 | Code with Claude 2026 企业案例 | 降 66% token、效果更好 |
| 给指令编号优先级 | AMBIG-SWE / ICLR 2026 | agent 默认不问澄清，编号让它知道何时停 |
| Writer/Reviewer 双 agent 分离 | Claude Code 社区 | 新鲜上下文消除确认偏误，优于自审 |
| "先访谈再执行" | Claude Code 社区 | 发现大量"自己不知道自己不知道"的问题 |

### 看起来该有效但实测没用

| 方法 | 来源 | 发生了什么 |
|------|------|-----------|
| Claude Sonnet 3.7 思考模式 | Augment / SWE-bench | 在该 scaffold 与该模型代际下无效，勿外推 |
| 单独的"修回归" agent | Augment / SWE-bench | 修一些回归也引入新 bug，净增益为零 |
| Embedding 检索 | Augment / SWE-bench | grep/find 就够了 |
| LLM 自动生成 AGENTS.md | 多项研究 2026 | 5/8 设置成功率下降，但总体差异未达统计显著；成本显著增加 |
| 把所有规范堆进 AGENTS.md | Daisy Hollman / Anthropic | 每轮付全部内容的费用，效果变差变贵 |

---

## 8. 分阶段行动建议

本文特有项：

**立即可做：**
1. 先做极简脚手架 baseline（一个 prompt + bash + edit），别一上来堆多 agent。
2. 复杂需求用"先访谈再执行"；代码审查试 Writer/Reviewer 双 agent 分离。

**有算力预算时：**
3. 投入 test-time compute：多候选采样 + 确定性验证/多数投票（Augment 实测 +3–8%，幅度因任务而异，先算成本）。
4. 检索先用 grep/find，确认检索是瓶颈再上 embedding。
5. 按模型族切换提示策略，并比较模型档位与 effort（§2.2、§3.3）。
6. 用 Advisor Strategy 做难度路由：高频低难度用小模型，难 case 调旗舰；成本按任务总成本比较（§3.3）。

**进阶：**
7. 多模型并行择优（多代理、worktrees 隔离）。
8. 定期审计 harness 假设是否随模型迭代腐化（§4.4）——纳入升级审计例行项。

---

[^bcg]: Dell'Acqua et al., "Navigating the Jagged Technological Frontier"（SSRN 4573321），2026-03 发表于 Organization Science。
[^arc]: ARC Prize 2024 Technical Report, arXiv:2412.04604。
*来源覆盖（仅保留内容实际引用）：Karpathy、Willison、Dell'Acqua & Mollick、Schulman、Lilian Weng、Howard、Goodside、宝玉、李继刚、云中江树、罗福莉、姚顺宇、Freda Duan、戴雨森、Cursor、OpenAI、Anthropic、Google DeepMind、ByteDance Seed、ARC Prize、SWE-bench / Augment / METR、AMBIG-SWE、Kaggle。时间范围 2024–2026.9。*
