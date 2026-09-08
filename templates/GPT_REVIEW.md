# GPT Delta Review 模板

> 用途：收到 Codex Handoff 后，GPT 对固定 Git Review Snapshot 做独立审查。
> 核心要求：**不得只依据 Codex 摘要自称接受。**

---

## 审查输入

至少需要：

```text
REPOSITORY
REVIEW_BASE
REVIEW_HEAD
REVIEW_REF（推荐）
GPT_REVIEW_READY=YES
```

如果关键固定 SHA 无法读取，应明确：

```text
BLOCKED
```

---

## 审查顺序

### 1. 固定审查对象

确认：

- Repository 正确
- `REVIEW_BASE` 存在
- `REVIEW_HEAD` 存在
- `REVIEW_HEAD` 与 Codex Handoff 一致
- Review Branch / Ref 如提供则可回读

### 2. 读取实际 Delta

实际读取：

```text
REVIEW_BASE..REVIEW_HEAD
```

检查：

- Changed Files
- 关键 Diff
- 必要代码上下文

### 3. 对照 Claim

回答：

- Claim 是否真的实现？
- 有没有只修表象、没修根因？
- 是否存在遗漏的生产者 / 消费者？

### 4. 检查 Changed Surface

回答：

- 修改是否集中在完成 Claim 所需范围？
- 是否发生无关重构？
- 是否顺手改变其他 Contract / 行为？

### 5. 检查 Contract 与兼容性

特别关注：

- 公共 API / Schema
- Gateway / Adapter / Provider
- 共享模块
- 多消费者逻辑
- 数据格式
- 权限边界

### 6. 检查测试

验证：

- T0 是否足够
- T1 是否直接覆盖 Changed Surface
- 公共能力变化是否有必要 T2
- 是否存在“跑了很多测试，但没测核心新增逻辑”

### 7. 检查 Clean Reproduction

若适用，判断：

- 是否来自远程固定 Review Snapshot
- 是否存在 Source Escape
- 是否偷偷依赖 canonical workspace、其他 Worktree 或私有 Runtime 状态

### 8. 检查 Runtime / Outcome 边界

明确区分：

```text
Git Review ACCEPTED
≠ Runtime Accepted
≠ Outcome Accepted
```

如果本轮只完成源码 Review，不要提前宣称 Runtime / Outcome 已通过。

---

# Review 输出模板

```text
GPT_REVIEW: ACCEPTED / REVISION_REQUIRED / BLOCKED

REPOSITORY:
...

REVIEW_BASE:
...

REVIEW_HEAD:
...

CLAIM_ASSESSED:
...

FINDINGS:
1. ...
2. ...

TEST_ASSESSMENT:
...

CLEAN_REPRO_ASSESSMENT:
...

RUNTIME_REQUIREMENT:
REQUIRED / NOT_APPLICABLE / ALREADY_PROVEN

DECISION_REASON:
为什么接受 / 要求修订 / 阻断。

NEXT:
INTEGRATION / CODEX_REVISION / PROVIDE_EVIDENCE / STOP
```

---

# 三种正式结论

## ACCEPTED

只有当固定 `REVIEW_HEAD` 的代码变化已经满足当前源码 Review Acceptance 时使用。

注意：

```text
ACCEPTED
```

不自动代表：

- Integration PASS
- Runtime PASS
- Outcome PASS
- main 已 Push

## REVISION_REQUIRED

应给出最小、明确、可执行的修订项：

```text
ISSUE:
问题是什么

WHY_IT_BLOCKS:
为什么影响 Acceptance

REQUIRED_CHANGE:
最小需要怎么改
```

然后生成新的 Codex 修订 Task。

新 Commit 产生新的 `REVIEW_HEAD` 后，必须重新绑定新 Head 做 Review。

## BLOCKED

适合：

- 固定 SHA 无法读取
- Diff 无法读取
- 关键 Contract 无法确认
- 核心测试证据缺失
- 权限不足
- Runtime / Outcome 无法验证但又是当前 Claim 必需条件

不得根据 Codex 的“已通过”摘要替代真实证据。

---

# 停止原则

一旦当前 Acceptance Surface 已经有充分证据：

```text
STOP
```

不要继续扫描无关目录，也不要为了“更全面”扩大 Review Surface。
