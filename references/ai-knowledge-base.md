# AI/Agent 使用技巧参考库

> 作者整理自全球顶尖大模型厂商与从业者公开分享的经验，随 upgrade-audit 包作为可选阅读附录。
> 本文收录来源、数据、模型/工具特定技巧与行业实证；通用文档工程与工作流规范见 `agent-doc-engineering.md`，本文不重复其操作规程，只交叉引用。
> 认知篇见 `ai-cognition-perspectives.md`，仅供使用者阅读，Agent/skill 无需默认加载。
> **时效说明（截至 2026-07）**：快变段为 §2.2（模型族）、§3.4（Claude Code 功能）、§3.8（Managed Agents）、§5 的 NSA 指南条目、§6 中的基准类结论。稳定段（工作流、心智模型、MCP 原则）不需反复核验；快变段应以官方当期文档和本地实测为准。
> 整理自 2024–2026-07 顶尖从业者公开分享、竞赛复盘与团队文档；尽量只收录可复用、有来源支撑的方法。
> **核实约定**：文中标注〔需核实〕/〔待核实〕的条目为未能核实一手来源、或随平台快速变化的转述内容，保留是取其现象与量级含义；引用或据此决策前请自行验证当期事实。

---

## 0. 心智模型校准

决定"何时信任输出、何时必须验证"的五个模型：

- **过度自信的实习生**（Willison）：LLM 像"读过世界上所有教科书但对你的代码库零实战经验、容易忘记最近一小时之外任何事的初级开发者"。"Over-confident"不能省——它不会说"我不确定"。
- **参差的能力边界**（jagged frontier）：AI 的能力不是均匀前沿线——某些任务超预期强，某些出人意料弱，不用就不知道边界在哪。实证（Dell'Acqua、Mollick 等人的 758 名 BCG 顾问实验[^bcg]）：用 AI 的顾问多完成 12.2% 任务、快 25.1%，但在"frontier 之外"的任务上给出正确解的比例比对照组**低约 19 个百分点**。含义：没确认 AI 擅长的任务，先小样本测。
- **插值器，不是创新器**（Howard）：LLM 擅长"在已知之间填空"，弱于分布外的概念创新。代理脚手架是把更多问题"挪进分布内"来 work 的。分布外任务降低期望。
- **幻觉两分类**（Lilian Weng）：in-context（输出与上下文源不一致）与 extrinsic（输出无世界知识支撑）。核验策略要分开：前者对照源文本，后者对照外部事实。
- **迎合机制**（Schulman）："模型只是想生成一条让人满意的消息。"它会揣摩你想听什么——所以不信口头断言、要求证据、用独立上下文的 Reviewer 对抗确认偏误（`agent-doc-engineering.md` §3.2）。

配套使用原则：

- **Mollick 四原则**：①凡合法合规都用 AI 试一遍（边界只能试出来）；②关键决策点保留人类检查；③给 AI 人设但记住它是机器；④假设这是你用过最差的 AI。
- **指令遵从 > 原始能力**（开发者社区共识）：工程场景中，可预测、稳定遵从指令的模型优于偶尔超水平但时常自作主张的模型。一个团队切换到指令遵从更稳的模型后，"AI 做了奇怪的事"类工单降了 70%。选型时把"是否听话"当一等指标。

---

## 1. 工作流总框架

2025–2026 年多个独立来源（Cursor、Anthropic、宝玉、Shumer、Howard）收敛到同一循环：**研究 → 规划 → 执行 → 审查 → 交付**（通用原则版见 `agent-doc-engineering.md` §3.1，此处只记来源与特色做法）：

- **先规划后编码**：Jeremy Howard 的 Dialogue Engineering 把这件事推到极致——"在生成代码前先打磨思维过程"，精心构造一段对话，对话的产出才是最终工件。
- **写测试让代理自验证**：Matt Shumer 的原话最硬——"先建测试框架再开始实现，每个测试通过前别完工——这些不是建议，是停止条件。"
- **人做审查者，不做打字员**：要求代理展示证据（测试输出、命令返回、截图），而非信它口头断言。Cursor："代理越快，你的审查越重要。"Fiona Fung（Anthropic 工程文化）：**shift verification left**——吞吐量上升后维持质量的唯一杠杆是把验证提前。
- **技术辩论中，代码赢**（Fiona Fung）：让 agent 把两个候选方案都原型化，比实际产物而非空谈设计文档。
- **会话卫生**：一个会话一类任务，避免"厨房水槽式会话"（Anthropic 语，规则见 `agent-doc-engineering.md` §1.8）。

---

## 2. 提示词策略

### 2.1 结构化 Prompt（复杂/可复用任务）

李继刚的方法论：像写文章一样编写提示词，用标识符（`#`、`<>`）控制层级。常用模块：Role/Profile（角色背景）、Constraint（"帮模型剪枝，减少不必要分支计算"）、Definition（"减轻幻觉的小技巧，把关键概念清晰定义"）、Example（few-shot）、Goal/Workflow。

云中江树的 LangGPT 把这套方法模板化："归根结底还是结构化、模板化 Prompt 的性能好。"注意两类一致性：格式语义一致性（标识符功能前后统一）和内容语义一致性。

李继刚指出局限："在创意性要求较高的场景，结构化思想不适用"——创意的本质是输出的随机性。

宝玉的判断：简单任务不需要精心提示词；复杂任务把专业提示词当"解难题的数学公式"，充当"工作流经理"串联多步骤。

### 2.2 不同模型族的差异化策略（快变段，截至 2026-07）

**用同一套提示词喂不同模型族，效果可能天差地别。** 换模型或模型升级后，先查该家最新官方提示指南，不要沿用旧策略。可迁移的教训（来自 DeepSeek R1 一代的验证）：**推理模型不要外加"think step by step"**——它自动生成思考块，外部要求导致冗余或循环；few-shot 对推理模型常不如清晰的直接指令。具体参数（温度、system prompt 取舍）随模型代际失效，一律以官方当期文档为准。

### 2.3 进阶提示技术

- **Ultra-deep thinking**（Shumer）：引导模型做远超平常的验证——"独立用替代方法验证你的推理，把每个事实、推断、结论对照外部数据交叉检查……一旦你完全确信，刻意停下，强迫自己从头再审视一遍。"
- **Composition 原则**（Shumer，"最被低估"）："格式很重要，因为格式改变思维"——指定确切的输出形状。
- **给指令编号优先级**（AMBIG-SWE，ICLR 2026）：agent 默认非交互行为，不主动问澄清就直接执行，导致 resolve 率从 48.8% 降到 28%。修复：给指令编号优先级，明确告诉 agent 何时应暂停询问。

---

## 3. Coding Agent 实操

### 3.1 Cursor 十条原则（官方博客，通用版见 `agent-doc-engineering.md` §3）

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

### 3.4 Claude Code 实用功能（快变段，逐条核验后使用）

- **Routines**（截至 2026-07）：prompt + 仓库 + 触发器（cron/webhook/GitHub 事件）在云端自主运行。适合晨间 PR review、夜间 CI 分析、依赖审计。注意配额与 GitHub 身份权限范围，细节见官方文档。
- **auto-fix**（截至 2026-07）：自动跟踪 PR、修 CI 失败、回应 review comment。`/autofix-pr` 触发。
- **Channels + Remote Control**（截至 2026-07）：session 跨设备接续；通过 Telegram/Discord 给运行中 session 发消息。
- **Code Review 插件**（截至 2026-07）：并行多 review agent 从不同角度独立审查，confidence scoring 过滤误报。
- **worktrees**（截至 2026-07）：enter/exit 工具自主开隔离分支，多任务并行。
- **Auto Mode**（截至 2026-07）：权限决策交给 classifier 筛破坏性操作与 prompt injection；有漏报率，适合有版本控制保底的场景；破坏性操作需权限升级——"让 agent 整晚跑"的安全前提（具体实现未公开，以官方当期文档为准）。
- **effort 参数**（截至 2026-07，已核验[^effort]）：API 档位 `low` / `medium` / `high`（默认）/ `xhigh` / `max`。`xhigh` 适用于最难的 coding/agentic 任务，`max` 不限 token 预算。指挥官可为不同工人子任务指定不同档位。
- **Task Budgets**（截至 2026-07）：单任务 token 与时间上限，防 agent 死胡同无限消耗。与 effort 配合：低价值任务低 effort + 紧 budget，高价值任务高 effort + 宽 budget。

### 3.5 社区验证的高效模式

- **Writer/Reviewer 双 Agent**：一个会话写、另一个独立会话只读审查，显著优于自审（原则版见 `agent-doc-engineering.md` §3.2）。同逻辑适用于测试：一个写测试，另一个写代码让测试通过。
- **先访谈再执行**：复杂或模糊任务，让 Claude 先访谈你输出 SPEC.md，新会话基于 SPEC 执行。社区反馈：发现大量"自己都没意识到自己不知道的问题"。
- **大规模并行迁移**：先列出所有待处理文件，每个文件独立会话并行（`claude -p "Migrate $file..." &`）。
- **并行假设探索**：多个 idea 分给不同 subagent 同时做、交叉验证（罗福莉团队："十个 idea 交给不同 subagent 同时做"）。与并行迁移的区别：前者探索假设空间，后者机械分片。
- **Compact 策略配置**：显式告诉 agent 压缩时必须保留什么（原则版见 `agent-doc-engineering.md` §1.4）。

### 3.6 LLM Wiki 模式（Karpathy）

让 agent 把杂乱源文档**增量编译成持久 Markdown wiki**（摘要、实体页、概念页、矛盾、交叉链接），而非每次 RAG 重新检索。这是"过去不可能、现在自然"的信息变换方式，也适合维护分层文档体系。

### 3.7 复杂任务拆分与状态文件（Anthropic 官方建议）

将复杂任务拆为子任务，用状态文件（progress.md、todos.md）维持跨会话上下文，每完成一个子任务即 git commit 并更新进度。关键设计：状态文件用结构化格式（JSON/JSONL），模型对结构化文件的覆盖更保守。可落地为 Initializer/Implementer 或 Commander/Worker 模式。

来源：<https://code.claude.com/docs/en/best-practices>（截至 2026-07）

### 3.8 Managed Agents 架构（Anthropic 2026，截至 2026-07〔需核实〕）

据官方文档，平台将 agent 系统解耦为 Agent（推理与工具调用）、Environment（沙箱执行）、Session（持久状态，崩溃可恢复）三层；凭据通过 Vaults 管理——agent 不接触密钥明文。细节见 https://platform.claude.com/docs/en/managed-agents/overview

---

## 4. 上下文与范式

### 4.1 从 Prompt Engineering 到 Context Engineering

Karpathy（2025-06）："在每个工业级 LLM 应用里，上下文工程才是用恰当信息填充上下文窗口的精妙艺术与科学。"

Goodside 早在 2023 年初就区分过 context engineering（为任务挑选和准备相关上下文）与 prompt programming（写清晰指令）；2025-11 他给出更精确的表述："提示词工程是操心用户该写什么，上下文工程是操心模型该读什么。这两者过去是同一回事……只有从后者中剥离出来，我们才能认真考虑前者，看看还剩下什么。"

操作规范（上下文比例、工具集最小化、压缩策略）见 `agent-doc-engineering.md` §1。

### 4.2 自动化你能验证的，而非你能指定的（Karpathy，Sequoia Ascent 2026）

**传统软件自动化你能"指定"的东西；LLM 和 RL 自动化你能"验证"的东西。** 这解释了为什么 math/coding/tests 进步最快——有清晰可验证信号。姚顺宇的互补判断："95% 的白领工作其实比写代码简单，但大模型做不掉的关键在于没有一个好的信号来评判好坏。"

推论：**要让 AI 做得好，先想清楚"什么叫做好"**。有清晰验证标准，AI 才能自我迭代；没有验证标准的任务，AI 只能靠一次猜对。（与 `agent-doc-engineering.md` §3.3 eval-driven 同源。）

### 4.3 品味（taste）是 agentic engineering 的核心竞争力（Karpathy）

代码能跑不等于设计对了——Karpathy 举例：让 Claude 做账户匹配，代码能跑但设计错了，"这是品味缺失，不是技术缺失"。人仍然对质量负责；瓶颈从写代码转移到判断什么值得做、哪个方案更好。

### 4.4 Harness 假设会腐化（戴雨森，2026）

Harness 对模型能力的假设随模型迭代变旧。常见症状：**过时的 few-shot 样板、为旧模型设计的防御性约束、基于旧模型弱点的 workaround**。定期审计 harness 假设是否仍成立，是维护 harness 价值的必要成本——这正是升级审计的核验项之一。

---

## 5. MCP 生产环境实战

MCP 已成事实标准（主流各家原生支持）。心智模型：**不是"给 AI 加工具"，而是 AI 和外部世界之间的标准化合约**——类比 USB-C，切换底层模型不需重写工具集成。区分：MCP 连接工具和数据源（给 agent 装手）；A2A 连接其他 agent（让 agents 协作），两者互补。

### 最常见的四个生产 bug

- **STDIO 陷阱**：STDIO 传输时 stdout 专传协议消息，代码里任何 `print()`/`console.log()` 写 stdout 会污染协议、让客户端崩溃。**所有日志发 stderr 或专用 sink**。HTTP 传输无此问题。
- **全局变量污染**：工具被不同用户并发调用，全局变量存 session 状态会导致用户数据互相泄露。为每个 session 维护显式状态对象。
- **Schema 验证缺失**：所有工具输入来自 LLM——**必须视为不可信**。强制严格 JSON Schema（含 `additionalProperties: false`），校验失败直接拒绝，不猜测意图。
- **Tool Description 写得差**：描述差 = 模型调错工具 = 看起来像模型 bug 实际是工具定义 bug。**把工具描述当 prompt 优化**，明确写出副作用。

### 安全要点（含 NSA 2026-05 指南〔需核实〕）

- **工具投毒/rug pull**：恶意 server 在获批后更改工具定义。缓解：连接后锁定定义，检测运行时变更。
- **Scoped Token 替代永久 API key**：15–30 分钟有效期最小权限 token，放 `CLAUDE.local.md`（不提交 git）或环境变量。
- **数据分区**：按敏感级别划分工具区域；私密数据优先本地 MCP server。

---

## 6. Eval 与基准（基准类数字为快变段，截至 2026-07）

### 6.1 Test-time Compute（跨赛事最强规律）

ARC（TTT+多候选投票）、AIMO（SC-TIR 多数投票）、SWE-bench（多 pass ensembling）、Kaggle（TTA）**全部独立验证**：test-time 阶段投入更多算力几乎总比换更大模型划算。

- **ARChitects**（ARC Prize 2024 冠军，隐藏测试集 53.5%[^arc]）：同一 LLM 既当生成器又当打分器，候选解施加数据增强后用负对数似然乘积聚合。
- **Augment**（SWE-bench 65.4%）：Claude 3.7 做 solver 生成多候选 diff，o1 做多数投票选择器，带来 3–8% 增益（60.6%→65.4%）。
- **Ryan Greenblatt**：GPT-4o 每题生成约 8000 个程序，解释器确定性验证——搜索量与准确率对数线性。

### 6.2 简单 > 复杂

**Agentless**：刻意禁止 LLM 自主决策，只用三步——分层定位、修复（采样多补丁）、验证（生成复现测试重排序），以 $0.34/题登顶开源榜，被 OpenAI 和 DeepSeek 采纳为标准评测方法。

**Scaffold 是比模型更大的变量（限饱和基准语境）**：六个前沿模型在 SWE-bench Verified 上相差 <0.8 分，但同一模型在三个不同 agent 系统下产生 5.2 分差距——纯粹来自上下文与工具调用管理。注意：这说明在该饱和基准上 scaffold 是主要区分点，**不等于模型在真实任务上无差别**（Arena 顶层与中层差距稳定，见 6.5；重要任务的降档实测见 §3.3）。

### 6.3 Eval-Driven Development

Anthropic 黑客松获胜者方法论：先定评估标准再实现，每次改动用 eval 分数而非直觉 justify。Jake Heinrich（CoCounsel）流程：先问真实专业人士什么是好表现 → 每个 prompt 先做十几个 eval 迭代到完美 → 再加 50 个 → 发布前 100 eval/prompt → 每个用户投诉都变成新 eval。Kaggle 铁律"相信你的 CV"（通用版见 `agent-doc-engineering.md` §3.3）。

### 6.4 Agent Eval 体系

- **pass@k**（k 次至少一次通过）衡量能力上限；**pass^k**（k 次全部通过）衡量可靠性。生产环境更关心 pass^k——用户不会给你多次机会。
- **Agent demo 可用 ≠ 生产可靠**：从 90% demo 到生产，每提升一个可靠性"9"都要独立的 eval/回归验证支撑（Karpathy "march of nines"的操作面）。
- Grader 三类：确定性验证器（测试、类型检查）> LLM-as-judge（开放式输出，需校准，偏差清单见 `agent-doc-engineering.md` §4.1）> 人工（贵但最可靠，用于 eval 的 meta-evaluation）。
- **eval 数据集本身会过时**，需定期审计 eval 质量——这是升级审计的固有职责。

### 6.5 基准怀疑主义（快变段，截至 2026-07）

- **报告分数必须说明 scaffold 条件**：同一基准的"最好成绩"常有多个数字同时流通（scaffold 与数据子集不同），引用任何分数不带条件即失真。
- **基准分是能力上界，不是真实工程能力**：METR 分析发现许多通过 SWE-bench 的 PR 过不了人工 review，模型在长任务上急剧退化；Augment 报告"改善生产 agent 的改动往往不动 SWE-bench 分数"——基准与真实产品体验脱节。
- **Chatbot Arena 用法**：有英语偏差、用户群偏技术、不区分"显著更好"vs"略好"。正确用法：用 Arena 排名缩小候选范围，再用任务特定 eval 做最终决定。顶层（Elo~1300+）与中层（~1100）差距显著且稳定，重要应用用顶层模型。

---

## 7. 反直觉清单

> 均为特定环境实测结论，外推前先在自己场景验证。

### 有效到意外

| 方法 | 来源 | 为什么意外 |
|------|------|-----------|
| 暴力采样数千程序 + 确定性验证 | Greenblatt / ARC | 准确率随搜索量对数线性增长 |
| grep/find 替代 embedding 检索 | Augment / SWE-bench | embedding 在代码定位场景并非瓶颈 |
| 强制绝对路径的"错误防护" | Anthropic / SWE-bench | 一个小约束消除一类系统性错误 |
| Markdown 替代 JSON 作为 tool 输出 | Code with Claude 2026 企业案例 | 降 66% token、效果更好（规则版见 `agent-doc-engineering.md` §1.6） |
| 给指令编号优先级 | AMBIG-SWE / ICLR 2026 | agent 默认不问澄清，编号让它知道何时停 |
| Writer/Reviewer 双 agent 分离 | Claude Code 社区 | 新鲜上下文消除确认偏误，优于自审 |
| "先访谈再执行" | Claude Code 社区 | 发现大量"自己不知道自己不知道"的问题 |

### 看起来该有效但实测没用

| 方法 | 来源 | 发生了什么 |
|------|------|-----------|
| Sonnet 3.7 思考模式 | Augment / SWE-bench | 在该 scaffold 与该模型代际下无效，勿外推 |
| 单独的"修回归" agent | Augment / SWE-bench | 修一些回归也引入新 bug，净增益为零 |
| Embedding 检索 | Augment / SWE-bench | grep/find 就够了 |
| LLM 自动生成 AGENTS.md | 多项研究 2026 | 5/8 场景降低成功率（原则版见 `agent-doc-engineering.md` §4.3） |
| 把所有规范堆进 AGENTS.md | Daisy Hollman / Anthropic | 每轮付全部内容的费用，效果变差变贵 |

---

## 8. 分阶段行动建议

通用工作流与文档工程操作（规划循环、可验证目标、会话卫生、cache 监控与清理规则、入口指令行数控制）见 `agent-doc-engineering.md` §1–§3，不在此重复。本文特有项：

**立即可做：**
1. 先做极简脚手架 baseline（一个 prompt + bash + edit），别一上来堆多 agent。
2. 复杂需求用"先访谈再执行"；代码审查试 Writer/Reviewer 双 agent 分离。

**有算力预算时：**
3. 投入 test-time compute：多候选采样 + 确定性验证/多数投票（Augment 实测 +3–8%，幅度因任务而异，先算成本）。
4. 检索先用 grep/find，确认检索是瓶颈再上 embedding。
5. 按模型族切换提示策略（§2.2）；Claude 用 effort 档位控制思考深度（§3.4）。
6. 用 Advisor Strategy 做难度路由：高频低难度用小模型，难 case 调旗舰；成本按任务总成本比较（§3.3）。

**进阶：**
7. 多模型并行择优（多代理、worktrees 隔离）。
8. 试点 Routines 做重复运维（夜间 CI 修复、依赖审计），注意权限范围。
9. MCP 生产：日志发 stderr、输入严格 Schema 验证、Scoped Token 替代永久 key（§5）。
10. 定期审计 harness 假设是否随模型迭代腐化（§4.4）——纳入升级审计例行项。

---

[^bcg]: Dell'Acqua et al., "Navigating the Jagged Technological Frontier"（SSRN 4573321），2026-03 发表于 Organization Science。
[^arc]: ARC Prize 2024 Technical Report, arXiv:2412.04604。
[^effort]: https://platform.claude.com/docs/en/build-with-claude/effort （2026-07-02 核验：档位为 low/medium/high/xhigh/max，API 默认 high）。

*来源覆盖（仅保留内容实际引用）：Karpathy（Dwarkesh / Sequoia Ascent 2026）、Willison、Dell'Acqua & Mollick（SSRN/Organization Science）、Schulman（Dwarkesh）、Lilian Weng、Howard（Solveit / Dialogue Engineering）、Goodside（X / Simon Willison 博客 2023-01）、宝玉、李继刚、云中江树（LangGPT）、罗福莉（张小珺 #138）、姚顺宇（张小珺 #140）、Freda Duan（张小珺 #141）、戴雨森（张小珺 #142）、Cursor 官方博客、Anthropic（含 Code with Claude 2026：Daisy Hollman、Brad Abrams、Fiona Fung）、DeepSeek 官方文档、ARC Prize 2024（arXiv:2412.04604）、SWE-bench 系列 / Augment / METR、AMBIG-SWE（ICLR 2026）、AGENTS.md 研究（2026）、Kaggle LLM 赛道、Anthropic/YC 黑客松复盘（Jake Heinrich）、Claude Code 社区（Reddit / GitHub）、MCP 生产实践（Nearform / NSA 2026-05 指南 / WorkOS）。时间范围 2024–2026-07。*
