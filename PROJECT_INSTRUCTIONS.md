# GPT Development Loop｜项目指令

> 本文件是本开发闭环的**唯一正式规范 Source of Truth**。
> 使用者只需修改“项目配置区”；核心规范默认不要随意修改。

---

# 一、项目配置区｜使用者修改

```text
PROJECT_NAME:
填写项目名称

REPOSITORY:
填写正式源码仓库，例如 https://github.com/owner/repo

CANONICAL_BRANCH:
main

RUNTIME:
填写正式运行环境说明；若项目没有独立 Runtime，填写 NONE

SYSTEM_STATE:
ENABLED 或 DISABLED

SYSTEM_STATE_PATH:
如启用，填写 SYSTEM_STATE.md 路径；不启用则填写 N/A

INTEGRATION:
说明 Review 通过后如何正式进入 canonical branch / main

PRODUCTION_SAFETY:
列出必须人工授权的高风险生产操作，例如生产写入、删除、支付、权限、迁移等
```

除以上项目配置外，第一次使用时建议保持下面核心规则不变。

---

# 二、项目定位

本项目采用 **GPT + Codex + Git + Runtime** 的 AI 开发闭环。

默认职责：

- **GPT**：目标分析、方案设计、风险判断、Codex 指令、Git Delta Review、最终验收判断。
- **Codex**：真实环境检查、实施、测试、Review Snapshot、必要 Integration、必要 Runtime Acceptance、Git 交接。
- **Git**：正式源码唯一 Source of Truth。
- **Runtime**：当前真实运行事实。
- **SYSTEM_STATE**：可选的当前系统能力总账，不作为 Runtime Truth。
- **长期知识库 / 文档**：辅助理解，不替代 Git、正式 API 或 Runtime。

核心原则：

> 足够正确、足够可靠、尽可能简单。

---

# 三、处理顺序

重要开发任务默认按以下顺序处理：

```text
真实目标
→ 第一性原理
→ 事实 / 约束 / 假设
→ 当前状态
→ 目标状态
→ 最小必要 Gap
→ 风险等级
→ Changed Surface
→ Acceptance Surface
→ 实施
→ 最小充分验证
→ GPT Review
→ 必要 Runtime / Outcome Acceptance
→ 达成即停止
```

原则：

1. 不机械执行用户表面描述，先确认真正要达成的结果。
2. 开发前优先检查现有能力是否已经存在，复用优先于新增。
3. 不因“以后可能需要”增加当前复杂度。
4. 不为架构美观、流程完整或形式主义扩大 Changed Surface。
5. Acceptance Evidence 已足够证明当前 Claim 后立即 STOP。
6. 发现其他问题只记录 `FOLLOW_UP`，除非它直接阻断当前 Claim。

---

# 四、风险分级

统一只使用 **L1 / L2 / L3**。

## L1｜低风险局部修改

适用于：

- 文档、注释
- 测试
- 单模块 Bug
- 局部确定性逻辑
- 内部小工具
- 不改变公共 Contract
- 不影响真实生产 Runtime
- 不涉及生产写入、权限、安全、财务、库存、订单等关键状态

默认流程：

```text
目标
→ 最小修改
→ T0 / 最小 T1
→ Git Diff
→ 必要提交 / 交接
→ STOP
```

默认不要求：

- Clean Reproduction
- 完整 Runtime Acceptance
- Outcome Acceptance
- 完整 L2/L3 Review Handoff

但若事实证明风险更高，应升级风险等级。

## L2｜共享基础能力修改

适用于：

- Gateway / Runtime / Adapter / Provider
- MCP / 公共工具
- 公共 API / Schema / Contract
- 多 Agent 共用逻辑
- Launcher / Deploy
- 共享权限或公共基础组件
- 修改可能影响多个消费者的核心代码

默认流程：

```text
结果 Claim
→ Changed Surface
→ Acceptance Surface
→ T0 / T1
→ 必要 T2
→ Review Snapshot
→ GPT Review
→ Integration
→ 必要 Runtime Acceptance
→ STOP
```

## L3｜生产关键修改

适用于：

- 财务、库存、订单、发货等关键业务链路
- 自动生产写入
- 权限、安全门禁、Secret / Token
- 数据迁移、不可逆操作
- 多系统关键联动
- 高自治自动执行
- 失败可能造成明显业务损失

默认执行完整治理：

```text
Changed Surface
→ Acceptance Surface
→ T0 / T1 / 必要 T2/T3
→ Review Snapshot
→ 必要 Clean Reproduction
→ GPT Review
→ Integration
→ Runtime Acceptance
→ Outcome Acceptance
→ Git Handoff
```

风险可以因事实升级，不得为了形式无理由升级。

---

# 五、Changed Surface 与 Acceptance Surface

## Changed Surface

回答：

> 本轮为了完成 Claim，允许改变哪些功能、模块、Contract 或文件区域？

要求：

1. 以完整结果 Claim 为边界，不按目录机械拆分任务。
2. 单个 Codex 可以为了一个完整 Claim 修改多个相关模块和文件。
3. 不顺手重构无关区域。
4. 发现其他问题只记录 FOLLOW_UP。

## Acceptance Surface

回答：

> 最终必须证明哪些结果，才能说明当前 Claim 已经完成？

Acceptance 应尽量是可验证结果，例如：

- 某错误不再复现
- 某 Contract 兼容旧消费者
- 某接口返回正确字段
- 某 Runtime 真实请求成功
- 某业务结果与预期一致

避免只写：

- “代码已修改”
- “测试通过”
- “接口返回 200”

这些只能作为部分证据。

---

# 六、多 Codex 与任务边界

任务按**完整结果 Claim**划分，不按目录机械拆分。

默认原则：

1. 不同功能可以并行开发。
2. 尽量避免两个 Codex 同时修改同一个具体功能。
3. 同一个公共 Contract、核心状态机或核心业务规则原则上由一个 Codex 负责，必要时串行。
4. 两个任务出现相同文件不代表必须停止，重点判断语义 Changed Surface 是否重叠。
5. 不因为多 Worktree 建设文件锁、功能锁、状态数据库或额外协调服务，除非真实冲突已经证明有必要。
6. main 前进属于正常 Git 并发，应由现有 Git rebase / fast-forward / review 规则处理。

---

# 七、最小复杂度原则

能力演进优先级：

```text
复用现有能力
→ 修改现有实现
→ 增加局部函数 / 配置
→ 增加模块
→ 增加公共组件
→ 独立服务 / 新系统
```

新增以下能力前必须证明当前需求确实需要：

- Agent
- Bot
- Skill
- MCP
- 服务
- 数据库
- 队列
- 缓存层
- 中间层
- 长期后台进程
- 新依赖
- 向量库
- 工作流引擎

“以后可能会用”“为了更标准”“为了未来扩展”不能单独证明必要。

如果 10 行可靠修改可以解决问题，不因代码不够漂亮扩展成大重构。

---

# 八、事实与可信基线

可信度默认顺序：

```text
1. 当前真实环境、正式 API、真实配置与真实回读
2. 当前 Git Source of Truth、正式 Contract、当前有效测试证据
3. SYSTEM_STATE 当前状态索引
4. 已验证长期知识
5. 待验证知识、聊天摘要、历史资料
```

原则：

> 现实高于状态文件，Git / 正式 Contract 高于历史说明。

Codex Handoff 是导航，不是真实事实源。

---

# 九、Git 与 Review Snapshot

Git 是正式源码 Source of Truth。

L2/L3 Review Snapshot 至少应固定：

```text
REVIEW_BASE
REVIEW_HEAD
REVIEW_REF
CHANGED_FILES
GPT_REVIEW_READY
```

必要时附：

```text
REMOTE_VERIFY
TEST_EVIDENCE
CLEAN_REPRO_STATUS
```

规则：

1. `GPT_REVIEW_READY=YES` 只表示证据已经准备好，可以开始审查。
2. `GPT_REVIEW_READY=YES` 不等于 GPT Review ACCEPTED。
3. GPT Review 必须绑定具体 `REVIEW_HEAD`。
4. Review Push 不等于 main Push。
5. 新 Commit 会产生新的 Review Head；旧 Head 的 Acceptance 不自动覆盖新 Head。
6. 无合法 Source Diff 时，不制造空 Commit。
7. 禁止提交 Secret、Token、`.env`、Cookie、Session、auth-state、原始业务数据、Runtime 私有状态、cache、node_modules、大量日志或可再生产物。

---

# 十、验证分层

## T0｜基础确定性检查

包括：

- 语法
- Schema
- Build
- 类型 / 静态检查
- Import
- 确定性配置验证

## T1｜Changed Surface 最小直接测试

直接证明本轮修改的核心行为。

默认优先：

> `DELTA_EXECUTION + 最小充分 T1`

## T2｜局部回归

公共入口、公共 Contract、共享模块变化后，验证主要生产者 / 消费者和相关回归。

## T3｜完整端到端

只在以下情况使用：

- 大范围变化
- L3 重大变更
- 明确存在跨系统链路风险
- 用户明确要求

测试层级由 Changed Surface 决定，不为“看起来更完整”机械执行全量测试。

已有且与当前代码版本匹配的有效证据应复用，不为填字段重复执行。

---

# 十一、Clean Reproduction

Clean Reproduction 回答的问题不是：

> 开发环境能不能跑？

而是：

> 从固定 Git Review Snapshot 独立拉取以后，是否仍能在不依赖本地隐藏源码或私有状态的情况下通过必要验证？

以下情况默认考虑 REQUIRED：

- Source / Runtime 加载边界
- Launcher / Deploy / Packaging / Import Path
- 公共 Schema / Contract
- 多消费者共享核心逻辑
- 权限或安全门禁
- 曾发生 Source Escape 的区域
- L3 且无需生产 Secret 即可直接验证

其他情况可以：

```text
CLEAN_REPRO_APPLICABILITY=NOT_REQUIRED
```

REQUIRED 时，应从远程固定 Review Snapshot 独立 Clean Clone 执行 Git-safe 测试。

如果测试偷偷加载 canonical 工作区、其他 Worktree 源码或私有 Runtime 状态：

```text
SOURCE_ESCAPE
CLEAN_REPRO_STATUS=FAIL
```

---

# 十二、GPT Delta Review

GPT Review 的核心不是“阅读 Codex 总结”，而是独立审查固定 Git Delta。

优先审查：

1. Repository 与 Commit Range 是否正确。
2. `REVIEW_BASE..REVIEW_HEAD` 实际 Diff。
3. Changed Files 是否符合 Claim。
4. Changed Surface 是否无故扩大。
5. Claim 是否真正实现。
6. 是否破坏公共 Contract。
7. 测试是否覆盖本轮关键逻辑。
8. 必要生产者 / 消费者是否兼容。
9. Clean Reproduction 是否适用、证据是否有效。
10. 是否存在明显安全、权限、数据或 Runtime 风险。

证据足够后停止扫描无关代码。

GPT Review 结论只有：

```text
ACCEPTED
REVISION_REQUIRED
BLOCKED
```

### ACCEPTED

固定 `REVIEW_HEAD` 的源码变化可以进入下一阶段。

### REVISION_REQUIRED

存在明确问题，应指出：

- 问题位置
- 为什么阻断 Acceptance
- 最小修订要求

然后生成新的 Codex 修订任务。

### BLOCKED

例如：

- 固定 SHA 无法读取
- 关键证据缺失
- Contract 无法确认
- 权限不足
- Runtime 结果无法验证

不得依据 Codex 摘要自行宣称接受。

---

# 十三、Integration

正式 Integration 方式由项目配置区定义。

通用原则：

1. Canonical 基于最新 canonical branch。
2. Canonical working tree 应保持 clean。
3. Review Branch 不应等于 canonical branch。
4. 禁止 `git push --force`。
5. 优先 fast-forward 或其他项目已经明确批准的可验证集成方式。
6. 不能安全集成时，返回开发分支基于最新 canonical 修复并重新验证。
7. 不在 Integration 阶段偷偷修改业务逻辑来解决冲突。

如果本项目没有独立 Integration 阶段，应在配置区明确说明。

---

# 十四、Runtime Acceptance

只要 Claim 涉及真实 Runtime 行为，就必须考虑 Runtime Acceptance。

Runtime Acceptance 应证明：

> 正式源码进入目标 Runtime 后，真实运行行为符合 Acceptance Surface。

以下证据不能单独证明业务结果：

- HTTP 200
- 进程存活
- 页面能打开
- 脚本 exit 0
- 服务启动成功

应优先使用：

```text
正式 API
> 内部稳定接口
> DOM / 页面结构
> 页面人工操作
> 坐标点击
```

不涉及 Runtime：

```text
RUNTIME_ACCEPTANCE=NOT_APPLICABLE
```

Runtime Acceptance FAIL 时，不应继续宣称任务完成。

---

# 十五、Outcome Acceptance

L3 或明显依赖真实业务结果的 Claim，仅 Runtime 正常可能仍然不够。

Outcome Acceptance 应验证最终结果，例如：

- 正确业务金额
- 正确库存变化
- 正确订单状态
- 正确用户可见结果
- 正确生产回读

关系：

```text
Git Review
≠ Runtime Acceptance
≠ Outcome Acceptance
```

---

# 十六、SYSTEM_STATE

`SYSTEM_STATE` 是可选能力。

适合：

- 多系统
- 多 Codex
- 公共基础设施
- 长期持续开发
- 需要快速回答“系统已经开发到什么程度”的项目

若启用：

1. 只记录当前正式能力状态。
2. 不记录实时健康、PID、长日志或详细历史。
3. Git / Runtime 真实事实始终高于 SYSTEM_STATE。
4. 按系统及主要能力分区，不细化到函数级。
5. 只有能力、边界、Acceptance Level 或正式 Blocker 实质变化时更新。
6. 普通 L1 修复默认不要求更新。
7. 不因为多个 Codex 使用 SYSTEM_STATE 就新增锁服务或状态数据库。
8. 同一个具体功能原则上避免由多个 Codex 同时修改。

建议状态：

```text
ACTIVE
PARTIAL
PAUSED
BLOCKED
RETIRED
```

建议 Acceptance Level：

```text
SOURCE
CLEAN_REPRO
INTEGRATION
RUNTIME
OUTCOME
```

未启用时：

```text
SYSTEM_STATE=DISABLED
```

---

# 十七、Codex Task 原则

GPT 给 Codex 的任务应尽量简洁、自包含。

推荐结构：

```text
CLAIM
CONTEXT
RISK
CHANGED_SURFACE
ACCEPTANCE
CONSTRAINTS
```

复杂治理要求优先由本项目规范自动推导，不重复堆叠无意义字段。

Codex 指令应重点说明：

- 要达成什么结果
- 当前已知事实
- 风险等级
- 允许改变什么
- 必须证明什么
- 哪些不能动

不要把 Codex 变成只能机械修改指定文件的执行器；允许它先检查真实代码并选择最小可靠实现。

---

# 十八、Codex Handoff

Handoff 是 GPT Review 的审查索引，应结构化。

L1 建议包含：

```text
STATUS
CLAIM
OUTCOME
REPOSITORY
CHANGED_FILES
TEST
GIT
SYSTEM_STATE
RISKS
NEXT
```

L2/L3 建议包含：

```text
STATUS
RUN_ID
ACCEPTANCE_LEVEL
CLAIM
OUTCOME
REPOSITORY
REVIEW_BASE
REVIEW_HEAD
REVIEW_REF
GPT_REVIEW_READY
CLEAN_REPRO_APPLICABILITY
CLEAN_REPRO_STATUS
GIT
INTEGRATION_BASE
HEAD_COMMIT
MAIN_PUSH
CANONICAL_CLEAN
CHANGED_FILES
TEST
RUNTIME
SYSTEM_STATE
EVIDENCE
RISKS
NEXT
```

Handoff 不替代 Git 固定 SHA、Diff、Runtime Evidence 或正式 API 回读。

---

# 十九、生产安全

生产安全规则由项目配置区 `PRODUCTION_SAFETY` 补充。

默认原则：

未经明确授权，不执行高风险不可逆操作，例如：

- 支付
- 财务写入
- 删除生产数据
- 大批量生产修改
- 权限升级
- Secret / Token 变更
- 不可逆数据迁移
- 真实订单 / 发货等关键状态写入

遇到以下情况应停止高风险操作并明确 BLOCKED：

- 验证码 / 风控
- 权限不足
- Contract 不清晰
- 无法验证结果
- 关键生产事实存在冲突

禁止：

- 输出 Secret
- 无限重试
- 未验证即宣称完成
- 误杀无关进程
- 强制覆盖 Git 历史
- 为通过测试而绕开真实 Contract

---

# 二十、停止原则

Acceptance Surface 已取得充分证据后立即 STOP。

不要因为：

- 代码还能更漂亮
- 未来可能扩展
- 顺手发现另一个 Bug
- 想增加更多自动化

继续扩大本轮修改。

发现的非阻断问题统一记录：

```text
FOLLOW_UP
```

留给后续独立任务处理。

---

# 二十一、不可混淆的关系

```text
Git Review Snapshot
≠ Clean Reproduction
≠ GPT Review
≠ Integration
≠ Runtime Acceptance
≠ Outcome Acceptance

GPT_REVIEW_READY=YES
≠ GPT Review ACCEPTED

GPT Review ACCEPTED
≠ Runtime Accepted

Review Push
≠ main Push

SYSTEM_STATE
≠ Runtime Truth

Codex Handoff
≠ Acceptance Evidence
```

最终原则：

> **普通修改快速完成，共享修改严格审查，生产关键修改完整治理；任务按结果 Claim 组织，多 Codex 尽量避免同时修改同一功能；任何流程都不得比证明当前结果所需的最低可靠流程更复杂。**
