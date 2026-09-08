# Codex Task 模板

> 用途：GPT 完成目标分析后，把这一段直接交给 Codex。
> 原则：简洁、自包含、围绕完整结果 Claim，不按目录机械拆任务。

```text
CLAIM:
用一句话描述本轮必须实现的最终结果。

CONTEXT:
- 当前已经确认的事实
- 当前问题 / 根因
- 与本轮直接相关的约束

RISK:
L1 / L2 / L3

CHANGED_SURFACE:
- 允许改变的功能 / Contract / 模块
- 只写本轮真正需要改变的范围

ACCEPTANCE:
1. 必须证明的结果 1
2. 必须证明的结果 2
3. 必须证明的结果 3

CONSTRAINTS:
- 不得修改的区域
- 不得扩大到的无关问题
- 必要生产安全限制
```

## Codex 执行原则

收到任务后，Codex 应：

1. 先读取真实 Git / 代码 / 配置 / 必要 Runtime 状态。
2. 优先复用现有能力，选择完成 Claim 的最小可靠方案。
3. 测试范围由 Changed Surface 决定。
4. L2/L3 按项目规范形成固定 Review Snapshot。
5. 发现其他问题只记录 `FOLLOW_UP`，不要顺手扩大修改。
6. Acceptance Evidence 足够后停止。

## 示例

```text
CLAIM:
修复共享鉴权中间件在 Token 过期时错误返回 500 的问题，并保持现有认证 Contract 兼容。

CONTEXT:
- 已确认异常发生在过期 Token 分支。
- 正常 Token 和缺失 Token 行为当前正确。
- 该中间件被多个 API 入口共享。

RISK:
L2

CHANGED_SURFACE:
- 共享鉴权中间件的过期 Token 处理
- 相关 Contract 测试
- 必要消费者回归测试

ACCEPTANCE:
1. 过期 Token 返回既定鉴权错误，不再产生 500。
2. 正常 Token、缺失 Token 行为不变。
3. 主要消费者相关测试通过。

CONSTRAINTS:
- 不重构整个认证系统。
- 不改变现有公开响应 Schema。
- 不处理与本 Claim 无关的登录功能问题。
```
