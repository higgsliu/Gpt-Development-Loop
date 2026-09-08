# GPT Development Loop

一套面向 **GPT + Codex + Git + Runtime** 的轻量 AI 开发闭环规范。

目标不是增加流程，而是解决一个核心问题：

> **Codex 说“完成了”，不等于任务真的完成。**

本规范把设计、实施、源码事实、独立审查和真实运行验证拆开，让 AI 开发过程可追踪、可审查、可验证，同时尽量保持小团队所需要的低复杂度。

---

## 一、核心角色

| 角色 | 主要职责 |
|---|---|
| **GPT** | 理解目标、设计方案、判断风险、生成 Codex 指令、独立读取 Git 代码做 Review、给出最终验收判断 |
| **Codex** | 检查真实环境、修改代码、测试、形成 Review Snapshot、Git 交接、必要时执行 Integration / Runtime Acceptance |
| **Git** | 正式源码 Source of Truth（事实源） |
| **Runtime** | 当前真实运行事实 |

辅助信息：

- `SYSTEM_STATE.md`：可选的系统能力状态总账，不是 Runtime Truth。
- Codex Handoff：审查导航，不是完成证据。
- 测试结果：技术证据，不等于最终 Acceptance。

---

## 二、完整开发闭环

```text
用户需求
  ↓
GPT：目标分析 / 当前状态 / 风险判断
  ↓
GPT：生成 Codex Task
  ↓
Codex：真实环境检查 / 最小修改 / 测试
  ↓
Git：固定 Review Snapshot
  ↓
Codex：Push Review Branch + 输出 Handoff
  ↓
GPT：实际读取 REVIEW_BASE..REVIEW_HEAD
  ↓
GPT Delta Review
  ├─ ACCEPTED
  ├─ REVISION_REQUIRED → 返回 Codex 修订
  └─ BLOCKED
  ↓
必要时 Integration
  ↓
必要时 Runtime Acceptance
  ↓
必要时 Outcome Acceptance
  ↓
main
  ↓
STOP
```

最重要的关系：

```text
Codex Handoff ≠ Acceptance
Test PASS ≠ GPT Review ACCEPTED
GPT Review ACCEPTED ≠ Runtime Accepted
Runtime Accepted ≠ Outcome Accepted
Review Push ≠ main Push
SYSTEM_STATE ≠ Runtime Truth
```

---

# 三、5 分钟开始使用

本仓库**不需要安装**，也不需要 Skill、服务、数据库或工作流引擎。

## 第 1 步：创建 ChatGPT Project

在 ChatGPT 中创建一个用于开发的项目，例如：

```text
公司开发
AI 基础设施
CRM 开发
内部工具
```

进入项目后：

```text
项目右上角「…」
→ 项目设置
→ 项目指令
```

> ChatGPT 界面名称可能随版本调整；核心目标是在该 Project 的“项目指令”中放入本仓库的正式规范。

## 第 2 步：复制 `PROJECT_INSTRUCTIONS.md`

打开：

[`PROJECT_INSTRUCTIONS.md`](./PROJECT_INSTRUCTIONS.md)

把完整内容复制到 ChatGPT Project 的项目指令中。

复制前只需要修改顶部的“项目配置区”。

推荐至少填写：

```text
PROJECT_NAME
REPOSITORY
CANONICAL_BRANCH
RUNTIME
SYSTEM_STATE
INTEGRATION
PRODUCTION_SAFETY
```

第一次使用时，除“项目配置区”外，**不要修改下面的核心规范**。

## 第 3 步：正常向 GPT 提开发需求

例如：

> 帮我修复登录接口偶发 500 的问题。

GPT 应自动完成：

```text
真实目标
→ 当前事实
→ 风险 L1 / L2 / L3
→ Changed Surface
→ Acceptance Surface
→ Codex Task
```

用户不需要自己先判断风险等级。

## 第 4 步：把 GPT 任务交给 Codex

GPT 会生成一份简洁任务，格式参考：

[`templates/CODEX_TASK.md`](./templates/CODEX_TASK.md)

Codex 在**你自己的源码仓库**工作，而不是修改本规范仓库。

## 第 5 步：Codex 完成后，把 Handoff 原样发回 GPT

L2/L3 任务通常需要固定：

```text
REVIEW_BASE
REVIEW_HEAD
REVIEW_REF
```

并 Push 到远程 Review Branch。

Codex 交接格式参考：

[`templates/CODEX_HANDOFF.md`](./templates/CODEX_HANDOFF.md)

## 第 6 步：GPT 独立读取 Git 做 Review

用户只需要把 Codex Handoff 原样贴回 GPT，并说：

> 请按项目规范做 GPT Delta Review。

GPT 不应只相信 Codex 摘要，而应实际读取固定 Git SHA 与 Diff，再输出：

```text
ACCEPTED
REVISION_REQUIRED
BLOCKED
```

Review 格式参考：

[`templates/GPT_REVIEW.md`](./templates/GPT_REVIEW.md)

---

# 四、风险分级

## L1｜低风险局部修改

适合：

- 文档、注释
- 测试
- 单模块 Bug
- 局部确定性逻辑
- 内部小工具
- 不改变公共 Contract
- 不影响真实生产 Runtime

默认流程：

```text
目标
→ 最小修改
→ T0 / 最小 T1
→ Git Diff
→ 必要提交
→ STOP
```

## L2｜共享基础能力修改

适合：

- Gateway / Runtime / Adapter / Provider
- 公共 API / Schema / Contract
- MCP / 公共工具
- 多 Agent 共用逻辑
- Launcher / Deploy
- 公共基础组件

默认流程：

```text
结果 Claim
→ Changed Surface
→ Acceptance Surface
→ T0 / T1 / 必要 T2
→ Review Snapshot
→ GPT Review
→ Integration
→ 必要 Runtime Acceptance
→ STOP
```

## L3｜生产关键修改

适合：

- 财务、库存、订单、发货等关键链路
- 自动生产写入
- 权限、安全门禁、Secret / Token
- 数据迁移、不可逆操作
- 多系统关键联动
- 失败可能造成明显业务损失

需要完整治理：

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

详细示例：

- [`examples/L1.md`](./examples/L1.md)
- [`examples/L2.md`](./examples/L2.md)
- [`examples/L3.md`](./examples/L3.md)

---

# 五、项目配置建议

`PROJECT_INSTRUCTIONS.md` 顶部只开放少数项目专属配置：

| 配置 | 说明 |
|---|---|
| `PROJECT_NAME` | 项目名称 |
| `REPOSITORY` | 正式源码仓库 |
| `CANONICAL_BRANCH` | 一般为 `main` |
| `RUNTIME` | 正式运行环境；没有可写 `NONE` |
| `SYSTEM_STATE` | `ENABLED` / `DISABLED` |
| `INTEGRATION` | Review 通过后如何正式进入 main |
| `PRODUCTION_SAFETY` | 哪些高风险操作必须人工授权 |

不建议每个团队重新设计一套风险模型；默认直接使用 L1 / L2 / L3。

---

# 六、个人与团队的两种使用方式

## 个人 / 小项目

最简单：

```text
复制 PROJECT_INSTRUCTIONS.md
→ 放进自己的 ChatGPT Project
→ 修改项目配置
→ 开始开发
```

## 团队 / 长期项目

可以 Fork 本仓库：

```text
Gpt-Development-Loop
→ Fork
→ company-development-loop
```

由团队维护自己的 `PROJECT_INSTRUCTIONS.md`。

此时团队 Fork 后的仓库就是自己的 Workflow Source of Truth。

---

# 七、SYSTEM_STATE 是可选能力

`SYSTEM_STATE.md` 适合：

- 多系统
- 多 Codex
- 公共基础设施
- 长期持续开发
- 需要快速回答“系统现在开发到什么程度”的项目

简单单体项目可以直接：

```text
SYSTEM_STATE=DISABLED
```

参考：

[`reference/SYSTEM_STATE.md`](./reference/SYSTEM_STATE.md)

---

# 八、Integration 不绑定某个平台

本规范不要求必须使用某个脚本、操作系统或 Git 平台。

核心 Integration Contract 是：

```text
1. 基于最新 canonical branch
2. Canonical working tree clean
3. Review Branch ≠ main
4. 只允许明确、可验证的集成路径
5. 禁止 force push
6. 发生冲突时返回开发分支修复，不在正式集成阶段偷偷扩大 Changed Surface
```

参考：

[`reference/INTEGRATION.md`](./reference/INTEGRATION.md)

---

# 九、设计原则

1. **治理成本必须与真实风险匹配。**
2. **能复用现有能力，就不要增加新系统。**
3. **测试范围由 Changed Surface 决定。**
4. **Review 必须绑定固定源码，而不是聊天摘要。**
5. **真实 Runtime 高于文档状态。**
6. **Acceptance Evidence 足够后立即停止。**
7. **发现无关问题只记录 FOLLOW_UP，不顺手扩大任务。**

最终目标：

> **足够正确、足够可靠、尽可能简单。**

---

# 十、这个仓库不是什么

本仓库不是：

- Skill
- Agent 平台
- CI/CD 平台
- 工作流引擎
- 状态数据库
- 文件锁系统
- 自动审批系统

它只是一个可复制的、Git 版本化的 **AI 开发闭环规范**。

真正的价值不是增加更多工具，而是让：

> **GPT 负责设计和独立审查，Codex 负责实施，Git 固定源码事实，Runtime 验证真实结果。**
