# 综合实训Ⅱ阶段 - 个人结题技术报告

## 一、项目与团队基本信息

- **本人姓名：杨金鑫**
- **本人学号：20216424**
- 项目名称：StateWeaver：状态驱动的本地工具调用型 Agent 系统
- 实际完成目标：参与完成包含状态管理、动态工具调用、验证恢复、长期记忆和 Web 展示的本地 Agent 系统，重点围绕 B3 动态 Tool Schema 与执行层 开展测试、验收和文档工作。
- **个人工作量：10%**
- **个人角色：B3 动态 Tool Schema 与执行层的测试验证、Schema 核对、参数异常用例和执行日志检查**
- 小组其他成员：梁少天、刘广阔、王渲淏

### 成员最终分工与交付核对表

| 角色 | 姓名 | 学号 | 实际负责或参与的内容 | 工作量占比 | 代码库链接 |
| :---: | :---: | :---: | :--- | :---: | :--- |
| 组长 | 梁少天 | 20236533 | 总体架构、B1-B5 核心实现、Verification、Recovery、Web Console 与集成 | 70% | [团队代码库](https://github.com/liangshaotian/stateweaver_agent) |
| 组员 | 刘广阔 | 20236507 | 参与 B2 Skill 工具系统的测试验证、接口检查、工具用例整理和文档编写 | 10% | [b2_skills](https://github.com/liangshaotian/stateweaver_agent/tree/main/b2_skills) |
| **组员** | **杨金鑫（本人）** | **20216424** | **参与 B3 动态 Tool Schema 与执行层的测试验证、Schema 核对、参数异常用例和执行日志检查** | **10%** | **[b3_tools](https://github.com/yangjinxin20020716/stateweaver_agent_b3_tools)** |
| 组员 | 王渲淏 | 20236512 | 参与 B5 可信分层记忆系统的测试验证、记忆样例整理、检索写回检查和文档编写 | 10% | [b5_memory](https://github.com/liangshaotian/stateweaver_agent/tree/main/b5_memory) |

> 说明：四份报告中的成员职责、工作量占比和代码库指向完全一致；仅按照模板要求，将当前报告提交人的所在行整体加粗。

---

## 二、整体系统架构与最终成果展示

### 2.1 最终系统总体架构图

![StateWeaver 系统总体架构](images/stateweaver_system_architecture.png)

图 1 StateWeaver 总体系统架构

StateWeaver 面向本地文档和结构化数据分析任务，由 B1 Runtime、B2 Skills、B3 Tool Layer、B4 Decision 和 B5 Memory 五个核心模块组成，并通过 Verification、Recovery Controller、Trace、Checkpoint 和 Web Console 构成完整执行闭环。

系统主流程为：

用户任务与本地数据  
→ B1 初始化 AgentState  
→ B5 检索可信记忆  
→ B4 生成结构化计划  
→ B3 编译 Tool Schema、筛选工具并校验参数  
→ B2 执行真实 Skill  
→ Verification 核验关键数值、文本和输出文件  
→ 失败时进入 Recovery / Replan  
→ 生成 Markdown、JSON、Trace 和 Checkpoint  
→ B5 写回经过验证的经验

五个模块之间通过 AgentState、PlanStep、SkillSpec、Tool Schema、SkillResult、EvidenceRecord、VerificationResult 和 MemoryRecord 等结构化对象协作。组长梁少天完成核心代码与总体集成；本人围绕 B3 动态 Tool Schema 与执行层 参与测试、接口核对、运行验收和技术文档整理。

### 2.2 系统整体运行流程与集成说明

系统启动后读取任务目标、允许文件、输出格式和预算配置。B1 创建 run_id 和独立运行目录，B5 检索与当前任务相关的历史经验，B4 将任务拆分为结构化步骤，B3 根据步骤选择最小必要工具并校验参数，B2 执行具体文件与数据操作。

工具结果进入 B1 后，会同步更新步骤状态、Tool History、Evidence 和预算。Verification 检查 required 步骤、输出文件、数值证据和文本证据；验证失败时，Recovery Controller 选择重试、参数修复、替代工具、重新规划、跳过可选步骤或优雅失败。

本人参与模块的系统位置和接口关系如下：

B3 位于 B4 Decision 与 B2 Skills 之间，是内部能力与模型调用接口之间的桥梁。B4 需要看到可调用 Tool Schema，B2 保存的则是真实 Python Skill。B3 负责将 SkillSpec 编译成 JSON Schema，根据任务筛选工具，对 Tool Call 参数进行验证和修复，并通过 Tool Executor 执行真实 Skill。本人承担 B3 方向的 Schema 核对、参数异常测试、路由结果检查、执行日志验收和文档整理。

主要跨模块接口包括：

| 接口对象 | 产生位置 | 使用位置 | 主要内容 |
|---|---|---|---|
| AgentState | B1 Runtime | B1 / B4 / B5 / Web | 阶段、步骤、工具、错误、证据与预算 |
| PlanStep | B4 Decision | B1 / B3 | 目标、工具、输入和依赖 |
| SkillSpec | B2 Skills | B3 / Web | 工具能力、参数与约束 |
| Tool Schema | B3 Tool Layer | B4 Decision | 模型可调用的标准接口 |
| SkillResult / ToolResult | B2 / B3 | B1 / B4 | 执行状态、输出、错误与证据 |
| EvidenceRecord | B2 / Verification | B1 / B4 | 来源、位置、Claim 与验证状态 |
| MemoryRecord | B5 Memory | B1 / B4 | 长期经验、状态与相关性 |
| VerificationResult | Verification | B1 / Recovery | 完成度、失败项和恢复建议 |

### 2.3 最终产品展示（Demo）

![StateWeaver Web 控制台](images/stateweaver_web_console.png)

图 2 StateWeaver Web Console

Web Console 分为任务配置、结果查看和运行监控三个区域。用户可以设置任务、允许读取文件、输出格式和预算；查看 Markdown 报告、JSON 摘要、Trace、Checkpoint、Evidence 和 Memory；监控当前阶段、工具调用、预算和 B1-B5 状态。

默认任务读取需求文档、会议记录、预算表和人员表，得到预算总额 4800.0、人员工时 122.0 h 和中等风险等级，并生成 report.md、summary.json、result.json、trace.jsonl、checkpoint.json 和 runtime.log。

### 2.4 团队系统代码库

- 团队完整合并代码库：[stateweaver_agent](https://github.com/liangshaotian/stateweaver_agent)
- 本人协作模块目录：[b3_tools](https://github.com/yangjinxin20020716/stateweaver_agent_b3_tools)

---

## 三、个人核心模块技术报告（个人成绩给定的核心依据）

### 3.1 模块定位与系统融合方式

#### 3.1.1 模块在系统中的角色

B3 位于 B4 Decision 与 B2 Skills 之间，是内部能力与模型调用接口之间的桥梁。B4 需要看到可调用 Tool Schema，B2 保存的则是真实 Python Skill。B3 负责将 SkillSpec 编译成 JSON Schema，根据任务筛选工具，对 Tool Call 参数进行验证和修复，并通过 Tool Executor 执行真实 Skill。本人承担 B3 方向的 Schema 核对、参数异常测试、路由结果检查、执行日志验收和文档整理。

#### 3.1.2 个人参与内容与工作边界

1、核对 SkillSpec、函数签名、类型注解和 Docstring 到 Tool Schema 的字段映射。  
2、检查 Schema Compiler 生成的工具名称、描述、properties、required 和类型定义。  
3、设计文档、表格、计算、输出和证据任务，检查 Tool Router 的动态工具集合。  
4、验证最小必要工具集是否排除与当前任务无关的 Skill，减少本地模型的选择空间。  
5、构造缺少参数、类型错误、路径错误、未知字段和未知工具等 Tool Call。  
6、检查数字字符串、单值列表、默认值和相对路径等确定性错误能否自动修复。  
7、验证 Tool Executor 的超时、重试、缓存、并行执行、异常捕获和耗时记录。  
8、检查 Tool Call、ToolResult、Trace 和 result.json 中的字段是否一致。  

本人的工作重点是测试验证、接口核对、运行验收和文档整理，不将组长完成的核心编码工作计入个人成果。通过这些工作，能够发现模块在真实集成环境中的字段遗漏、错误处理和状态同步问题，并协助完成最终验收。

#### 3.1.3 与上下游模块的协同

本人围绕 B3 动态 Tool Schema 与执行层 检查以下协同关系：

1、与 B1 Runtime  
模块结果是否能够写入 AgentState；错误是否能够触发恢复；Trace 是否完整记录调用和结果。

2、与 B2 Skills  
工具说明、参数和返回结构是否一致；Evidence 是否能够继续进入验证流程。

3、与 B3 Tool Layer  
Tool Schema、Tool Call 和 ToolResult 是否使用统一字段；参数错误是否在执行前被发现。

4、与 B4 Decision  
Planner 是否能够看到正确候选工具或记忆上下文；Verifier 是否能够理解模块输出。

5、与 B5 Memory  
任务前的记忆检索和任务后的记忆写回是否由 B1 统一触发，避免模块绕过 Runtime 修改全局状态。

### 3.2 核心技术理解与协作实施路径

![B3 动态 Tool Schema 与执行层 架构](images/b3_tool_layer_architecture.png)

图 3 B3 动态 Tool Schema 与执行层

#### 3.2.1 模块设计理解

B3 的完整流程为：

Skill Registry  
→ Schema Compiler  
→ Tool Router  
→ Parameter Validator / Repair  
→ Tool Executor  
→ ToolResult

Schema Compiler 读取 SkillSpec、Python 函数签名、类型注解和配置，输出模型可理解的 JSON Tool Schema。Tool Router 根据任务关键词、当前阶段、输入文件类型、工具标签、风险、成本和剩余预算筛选工具。

参数校验层负责检查 required 字段、参数类型、枚举、范围和路径安全。系统只对可确定的错误进行修复，例如把 `"12"` 转换为数字、把单值转为单元素列表或补充明确默认值。无法可靠推断的参数会返回 validation_error，由 B1 决定重试、修复或 Replan。

Tool Executor 统一处理调用、异常、超时、缓存和并行。执行结果被封装为 ToolResult，并写入 AgentState 和 Trace。

#### 3.2.2 关键机制与验收重点

B3 验收时特别关注以下边界：

1、工具可见性  
任务只需要表格分析时，候选集合应包含 table_analyzer 和必要的文件工具，不应把全部 Skill 都交给模型。

2、Schema 一致性  
模型看到的参数名、类型和 required 字段必须与真实函数一致，避免调用时发生接口漂移。

3、修复边界  
参数修复用于确定性转换，不能凭空生成用户没有提供的路径或业务值。

4、执行隔离  
工具超时后应返回 timeout 状态，并尽可能终止独立进程，防止后台任务继续占用资源。

5、缓存正确性  
相同工具、相同参数和相同输入文件可以复用缓存；文件哈希变化后旧缓存应失效。

6、可观测性  
每次调用必须记录工具名、参数摘要、开始时间、耗时、状态、错误、缓存命中和结果摘要。

#### 3.2.3 测试实施过程

本人的测试过程分为四步：

1、阅读接口与配置  
先核对模块公开字段、允许值和与上游的依赖，避免仅根据页面结果判断。

2、构造测试输入  
为正常、边界和错误场景准备可复现输入，保证每个测试都能对应明确的预期结果。

3、检查多处产物  
同时检查函数返回、AgentState、Trace、result.json 和 Web Console，避免只看单一输出。

4、回归验证  
发现问题并完成修改后，重新运行默认任务与自动化测试，确认没有破坏其他模块。

代表性验收逻辑如下：

```python
def validate_tool_record(record: dict) -> None:
    required = {
        "tool", "arguments", "status",
        "elapsed_sec", "cached", "error"
    }
    assert required.issubset(record)
    assert record["status"] in {"ok", "error", "timeout"}

def test_router(router) -> None:
    tools = router.select_tools(
        task="读取 CSV 预算表并计算总额",
        stage="EXECUTE",
        file_types=["csv"],
    )
    assert "table_analyzer" in tools
    assert "file_reader" in tools
    assert len(tools) < router.total_skill_count

```


#### 3.2.4 真实 Tool Call 链路检查

本人在验收时按照“Schema 是否正确、工具是否选对、参数是否安全、执行是否真实、日志是否完整”的顺序检查 B3。

第一步，检查 Schema Compiler 输出。工具名称、描述、properties、required 和类型必须与真实 SkillSpec 对应。例如 table_analyzer 的 path 必须是字符串，不能在 Tool Schema 中误写为列表。

第二步，检查动态工具集合。针对“读取 Markdown”“分析 CSV”“生成报告”和“验证证据”等不同步骤，本人记录候选 Toolset，确认系统只暴露相关工具。若当前任务没有计算需求，calculator 不应无条件占用上下文。

第三步，检查 Tool Call 参数。本人构造 path 为列表、数值为字符串、缺少 required 字段、包含未知字段和越界路径等情况，观察系统是否先校验再执行。可确定的格式错误可以修复，业务信息缺失则必须返回错误。

第四步，检查执行记录。每次 Tool Call 应在 Trace 中包含 tool、arguments 摘要、status、elapsed_sec、cached 和 error。执行成功后 ToolResult 必须被 B1 合并到 AgentState；执行失败后必须形成明确 failure_type。

第五步，检查缓存和文件变化。相同文件和相同参数重复调用可以命中缓存；修改输入文件后，文件哈希变化，旧缓存必须失效。否则系统可能使用过期结果。

第六步，检查并行与依赖。无依赖的文档读取和表格分析可以并行，依赖前序结果的 format_converter 与 evidence_checker 必须在前序步骤完成后执行。

#### 3.2.5 B3 方向发现的问题与核对结论

1、动态路由与实际执行脱节  
若 Tool Router 只生成候选集合，但 Runtime 仍直接调用硬编码工具，则动态工具机制只是展示。验收必须确认 selected_tool 真正决定执行路径。

2、tools.yaml 与 Registry 不一致  
配置文件、SkillSpec 和函数签名如果维护三套信息，容易出现参数漂移。Schema Compiler 应以统一来源为准，并进行一致性检查。

3、参数修复范围过大  
自动修复不能猜测文件、列名和业务值。可确定的类型转换与默认值补全可以执行，语义缺失应交给 Replan。

4、线程超时不能真正停止任务  
普通线程等待超时后，后台函数可能继续运行。对可能阻塞的工具需要独立进程或可取消执行方式。

5、缓存键不包含文件状态  
仅根据工具名和路径缓存会在文件更新后返回旧结果。缓存键需要包含输入文件哈希或修改时间。

6、调用记录不足  
若只记录“成功/失败”，无法定位参数、耗时、缓存和恢复问题。Trace 必须保存足够但不过度泄露敏感内容的执行摘要。

通过上述检查，本人确认 B3 的验收标准应覆盖编译、路由、验证、执行和可观测五个环节。

### 3.3 最终结果与性能评估

#### 3.3.1 测试矩阵

| 测试类别 | 测试内容 | 预期结果 |
|---|---|---|
| Schema 生成 | 函数签名与类型注解 | 生成正确 JSON Schema |
| 必填字段 | 缺少 path | 返回 validation_error |
| 类型修复 | 数字字符串、单值列表 | 完成确定性转换 |
| 非法路径 | `../secret` | 在执行前拒绝 |
| 动态路由 | 表格分析任务 | 选择相关最小工具集 |
| 未知工具 | 不存在的 tool name | 返回 unknown_tool |
| 工具异常 | Skill 主动抛出异常 | 统一封装 error |
| 工具超时 | 阻塞操作 | 返回 timeout 并记录耗时 |
| 缓存 | 同输入重复调用 | 第二次命中缓存 |
| 文件变化 | 修改输入文件 | 旧缓存失效 |
| 并行调用 | 无依赖文档读取 | 并发执行并统一提交 |
| Trace | 正常、错误和超时调用 | 日志字段完整 |

#### 3.3.2 集成测试与端到端验证

除模块测试外，本人还关注该模块能否进入真实主流程。端到端验证步骤包括：

1、从 Web Console 或配置文件提交默认任务。  
2、确认 B1 进入对应阶段并调用相关模块。  
3、在 Trace 中检查输入、输出、状态和耗时。  
4、检查 result.json 是否保存结构化结果。  
5、检查 report.md 和 summary.json 是否使用相关结果。  
6、在异常场景下确认 Recovery Controller 能够收到错误。  
7、重复运行检查缓存、去重或历史状态是否正确。  

项目整体完成 48 项自动化测试，覆盖状态、预算、动态计划、工具执行、路径安全、表格分析、证据验证、Checkpoint、Memory、Web API 和 UI 流程。

#### 3.3.3 结果分析

验收结果表明，B3 可以从 B2 Skill Registry 获取工具说明并生成 Tool Schema。默认任务中，Tool Router 根据计划阶段提供 file_reader、local_file_search、table_analyzer、format_converter 和 evidence_checker 等相关工具，没有把全部能力无差别注入本地模型。

错误参数能够在执行前被发现，部分明确错误可以修复；未知工具、非法路径和不可推断参数会返回结构化错误。Tool Executor 的工具名、参数、状态、耗时和错误可以写入 Trace，工具超时或异常不会直接中断整个 Agent。

从验收结果看，本人负责的协作方向能够在完整 StateWeaver 闭环中正常工作，测试和接口核对也帮助明确了模块的输入输出边界。个人工作量与最终分工表保持一致，为 10%。

#### 3.3.4 模块价值与可扩展性

B3 动态 Tool Schema 与执行层 的价值不仅体现在默认 Demo。通过统一接口和测试规范，系统后续可以继续增加新的 Skill、替换 Tool Router、接入更强本地模型或扩展记忆检索，同时保持 B1 Runtime 的主流程稳定。

本人参与的测试与文档工作为后续扩展提供了以下基础：

1、明确字段和状态含义。  
2、建立正常与异常用例。  
3、形成可复用验收清单。  
4、保证 Trace 和 Web 展示能够反映真实运行。  
5、记录当前局限和可继续优化的位置。  


#### 3.3.5 后续优化建议

1、将 SkillSpec、Tool Schema 和函数签名的一致性检查加入启动阶段。  
2、记录 Tool Router 的候选集合、过滤原因和最终选择，提升决策可解释性。  
3、为参数修复建立明确规则表，区分可自动修复与必须 Replan 的错误。  
4、为高风险和长时工具提供进程级隔离、资源限制和终止能力。  
5、增加 Tool Call 成功率、平均耗时、缓存命中率和重试次数统计。  
6、为并行执行增加更严格的 DAG 检查和状态提交机制，防止重复提交或依赖未满足。  
7、在 Web Console 中展示 Tool Schema、参数修复前后对比和执行日志。  

这些优化能够进一步提升本地小模型的工具调用稳定性，也使 B3 更适合扩展到更多 Skill 和更复杂任务。

### 3.4 个人交付物清单

- **个人工作量：10%**
- **个人协作方向：B3 动态 Tool Schema 与执行层**
- 模块目录：[b3_tools](https://github.com/yangjinxin20020716/stateweaver_agent_b3_tools)
- 一致的成员分工与工作量核对表
- 模块测试与接口核对清单
- 正常、边界和异常测试用例说明
- 系统总体架构图、模块架构图和 Web Console 截图
- report.md、summary.json、result.json、trace.jsonl、checkpoint.json 和 runtime.log 运行产物核对
- 本 Markdown 报告及对应 PDF

---

## 四、实训总结与心得体会

### 4.1 遇到的最大挑战

B3 的难点是同时满足模型友好和工程安全。Schema 过于简单会导致模型缺少参数信息，Schema 过长又会增加本地模型的上下文负担。参数修复能够提高成功率，但修复范围过大会引入错误业务值。

本人通过逐项核对 Schema、设计错误参数矩阵、检查路由集合和分析 ToolResult 日志，协助确认系统只暴露当前步骤需要的工具，只修复可以确定的格式问题，并把其余问题交给 Runtime 的恢复策略。

### 4.2 如何克服

1、先梳理模块职责与接口对象，明确哪些字段属于本模块，哪些由上游或下游提供。  
2、将测试划分为正常、边界、异常和端到端四类，逐项记录预期结果。  
3、结合 Trace、result.json 和 Web Console 交叉检查，避免单看页面造成误判。  
4、发现问题后将复现条件、输入、预期和实际结果整理清楚，再交给组长统一修正。  
5、修改后重新执行默认任务和自动化测试，确认相关模块与其他模块仍能协同。  

### 4.3 心得体会

通过本次协作，我理解了 Skill、Tool、Tool Schema 和 Tool Call 的区别，也认识到大模型工具调用必须经过程序化验证。测试过程中涉及 JSON Schema、函数签名、类型检查、路径安全、缓存键、超时和并发等工程问题，提升了我对 Agent 执行层和异常链路的认识。

本次实训也让我认识到，协作测试和文档并非附属工作。Agent 系统包含模型、状态机、工具、验证和记忆，任何一个字段或错误状态不一致都可能导致整条链路失败。清晰的接口、可复现的测试和准确的技术说明，是系统能够稳定交付的重要保障。
