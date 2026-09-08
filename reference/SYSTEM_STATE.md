# SYSTEM_STATE 参考

> `SYSTEM_STATE.md` 是**可选能力**，不是所有项目都需要。

它只回答一个问题：

> **系统当前已经开发到什么程度？**

它不是：

- Runtime Truth
- 实时健康监控
- 任务数据库
- 日志系统
- 第二个 Git
- Agent Knowledge 的替代品

---

# 一、什么时候建议启用

适合：

- 多系统项目
- 多个 Codex 并行开发
- 公共基础设施
- 多个共享能力长期演进
- 经常需要快速判断“某能力现在是什么状态”

简单项目，例如：

- 单人小工具
- 简单 CRUD
- 单体应用
- 短期项目

可以直接：

```text
SYSTEM_STATE=DISABLED
```

不要为了流程完整而强制增加状态文件。

---

# 二、可信度

默认事实优先级：

```text
Runtime / 正式 API / 真实回读
>
Git / 正式 Contract
>
SYSTEM_STATE
>
长期知识 / 历史说明
```

如果 SYSTEM_STATE 和真实 Runtime 冲突：

> **以 Runtime 为准，并在必要时修正 SYSTEM_STATE。**

---

# 三、推荐结构

```markdown
# SYSTEM_STATE

## Authentication

### Token Validation
Status: ACTIVE
Acceptance Level: RUNTIME
Blocker: NONE
Notes: 当前正式 Token Contract 已在 Runtime 验证。

### SSO
Status: PARTIAL
Acceptance Level: SOURCE
Blocker: 尚未完成 Runtime Acceptance

---

## Notification

### Email
Status: ACTIVE
Acceptance Level: OUTCOME
Blocker: NONE

### SMS
Status: PAUSED
Acceptance Level: SOURCE
Blocker: 等待供应商配置
```

---

# 四、建议状态

统一使用少量状态：

```text
ACTIVE
PARTIAL
PAUSED
BLOCKED
RETIRED
```

不要不断新增类似：

```text
HALF_ACTIVE
ALMOST_READY
WAITING_VERIFY
SOFT_BLOCKED
```

状态越多，维护成本越高。

---

# 五、Acceptance Level

建议：

```text
SOURCE
CLEAN_REPRO
INTEGRATION
RUNTIME
OUTCOME
```

含义：

| Level | 含义 |
|---|---|
| SOURCE | 当前源码及源码测试已经证明 |
| CLEAN_REPRO | 固定 Git Snapshot 可独立复现 |
| INTEGRATION | 已安全进入正式集成状态 |
| RUNTIME | 正式运行环境行为已验证 |
| OUTCOME | 最终业务 / 用户结果已验证 |

注意：

```text
SOURCE
≠ RUNTIME
≠ OUTCOME
```

---

# 六、什么时候更新

只有以下情况建议更新：

- 能力状态实质变化
- 系统边界实质变化
- Acceptance Level 变化
- 正式 Blocker 变化
- 能力被明确 RETIRED

普通 L1 Bug 修复通常：

```text
SYSTEM_STATE=NO_CHANGE
```

---

# 七、多 Codex 冲突

不需要因为多个 Codex 都可能修改 `SYSTEM_STATE.md` 就建设：

- 文件锁
- 状态数据库
- 独立协调服务
- 功能锁平台

推荐做法：

1. SYSTEM_STATE 按系统 / 主要能力分区。
2. 每个 Codex 只修改自己 Changed Surface 对应的区域。
3. 尽量避免两个 Codex 同时修改同一个具体功能。
4. 不同功能即使位于同一个文件，Git 通常也能正常处理。
5. 真正发生语义冲突时再串行解决。

对小团队来说，这通常足够。

---

# 八、不要记录什么

不要放入：

- PID
- 实时 CPU / Memory
- 大量日志
- 每次测试明细
- Secret / Token
- 长篇开发历史
- 每个函数状态
- 所有 Commit 记录

这些信息应由它们各自的正式系统负责。

---

# 九、核心原则

> **SYSTEM_STATE 是当前能力索引，不是新的事实数据库。**

如果维护它的成本开始大于它提供的导航价值，应简化甚至关闭，而不是继续增加治理系统。
