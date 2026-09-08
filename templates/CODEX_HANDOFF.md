# Codex Handoff 模板

> Handoff 是 **Codex → GPT** 的审查索引，不是 Acceptance 本身。
> 真正证据仍然来自固定 Git SHA、Diff、测试、正式 API 和 Runtime 回读。

---

## L1｜轻量交接

```text
STATUS:
COMPLETED / BLOCKED

CLAIM:
本轮要实现的结果。

OUTCOME:
实际完成了什么。

REPOSITORY:
owner/repo 或仓库 URL

CHANGED_FILES:
- path/to/file
- path/to/test

TEST:
- T0: PASS / FAIL / N/A
- T1: PASS / FAIL / N/A

GIT:
- BRANCH:
- COMMIT:
- PUSH:

SYSTEM_STATE:
NO_CHANGE / UPDATED / DISABLED

RISKS:
剩余风险；没有则写 NONE。

NEXT:
STOP / 后续必要动作
```

---

## L2 / L3｜Review Handoff

```text
STATUS:
COMPLETED / BLOCKED

RUN_ID:
可选；没有则 N/A

ACCEPTANCE_LEVEL:
SOURCE / CLEAN_REPRO / INTEGRATION / RUNTIME / OUTCOME

CLAIM:
本轮完整结果 Claim。

OUTCOME:
本轮已经真实完成的结果，不要把尚未执行的 Integration / Runtime 写成完成。

REPOSITORY:
owner/repo 或仓库 URL

REVIEW_BASE:
固定 base commit SHA

REVIEW_HEAD:
固定 head commit SHA

REVIEW_REF:
refs/heads/xxx 或明确 Review Branch

GPT_REVIEW_READY:
YES / NO

CLEAN_REPRO_APPLICABILITY:
REQUIRED / NOT_REQUIRED / BLOCKED

CLEAN_REPRO_STATUS:
PASS / FAIL / NOT_RUN / NOT_REQUIRED

GIT:
- REVIEW_PUSH: YES / NO
- REMOTE_VERIFY: PASS / FAIL / N/A

INTEGRATION_BASE:
尚未 Integration 写 N/A

HEAD_COMMIT:
尚未 Integration 写 N/A

MAIN_PUSH:
YES / NO

CANONICAL_CLEAN:
YES / NO / N/A

CHANGED_FILES:
- path/to/file
- path/to/test

TEST:
- T0:
- T1:
- T2:
- T3:

RUNTIME:
- RUNTIME_ACCEPTANCE: PASS / FAIL / NOT_APPLICABLE / NOT_RUN
- EVIDENCE: ...

SYSTEM_STATE:
NO_CHANGE / DISABLED / UPDATED
如 UPDATED：写明 SECTION 与 CHANGE。

EVIDENCE:
- 最关键的可复核证据
- 不要粘贴大量日志

RISKS:
- 尚存风险
- 没有则 NONE

FOLLOW_UP:
- 与当前 Claim 无关、但值得后续处理的问题

NEXT:
GPT_DELTA_REVIEW / INTEGRATION / RUNTIME_ACCEPTANCE / STOP
```

## 重要规则

```text
GPT_REVIEW_READY=YES
≠ GPT Review ACCEPTED

Review Push
≠ main Push

Test PASS
≠ Runtime Accepted

Handoff
≠ Acceptance Evidence
```

如果还没有进入 Integration：

```text
INTEGRATION_BASE=N/A
HEAD_COMMIT=N/A
MAIN_PUSH=NO
```

不要为了“字段看起来完整”虚构尚未发生的结果。
