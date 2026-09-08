# Integration 参考

> Integration 是把已经通过 GPT Review 的固定 Review Snapshot 安全带入正式 canonical branch / main 的过程。

本规范**不绑定某个操作系统、脚本或 Git 平台**。

你可以使用：

- GitHub
- GitLab
- Gitea
- Bitbucket
- PowerShell
- Shell
- CI/CD
- 团队已有发布流程

但应该满足相同的核心 Contract。

---

# 一、核心原则

```text
1. 基于最新 canonical branch
2. Canonical working tree clean
3. Review Branch ≠ canonical branch
4. 固定 Review Head 已通过 GPT Review
5. 禁止 force push
6. 冲突时返回开发阶段修复
7. Integration 阶段不偷偷改变业务逻辑
```

推荐优先使用可明确证明源码来源的集成方式。

---

# 二、推荐流程

```text
GPT Review ACCEPTED
        ↓
确认固定 REVIEW_HEAD
        ↓
获取最新 canonical branch
        ↓
确认 canonical clean
        ↓
验证是否可以安全集成
        ↓
Integration
        ↓
必要源码 / 集成测试
        ↓
必要 Runtime Acceptance
        ↓
必要 Outcome Acceptance
        ↓
Push canonical / main
```

---

# 三、为什么强调固定 REVIEW_HEAD

GPT 接受的是：

```text
REVIEW_BASE..REVIEW_HEAD
```

而不是：

```text
某个会继续变化的 branch 名称
```

如果 GPT Review 后 Review Branch 又新增 Commit：

```text
旧 ACCEPTED
≠ 新 Commit 自动 ACCEPTED
```

应形成新的 `REVIEW_HEAD` 并重新 Review。

---

# 四、Fast-forward

对于结构简单的小团队，推荐优先采用：

```text
fast-forward only
```

因为它：

- 简单
- 可追踪
- 不会额外制造复杂 Merge 结果
- 容易确认正式源码就是被 Review 的源码

但本规范不强制所有团队必须使用 fast-forward。

如果团队已有经过验证的 Merge / PR 流程，只要可以明确证明：

> **进入正式分支的源码与已审 Review Snapshot 的关系清晰且可复核。**

就可以继续使用。

---

# 五、发生冲突怎么办

不要在正式 Integration 阶段临时编辑大量源码来“把冲突解决掉”。

推荐：

```text
Integration FAIL
→ 返回原开发分支 / Worktree
→ 基于最新 canonical 更新
→ 解决冲突
→ 重新测试
→ 新 Review Snapshot
→ GPT Review
→ 再 Integration
```

原因：

如果 Integration 阶段修改了业务逻辑，那么：

```text
最终代码
≠ GPT 实际 Review 的代码
```

原来的 Acceptance 就失去意义。

---

# 六、禁止 force push

默认禁止：

```text
git push --force
```

尤其禁止对 canonical / main 强制覆盖历史。

如果项目存在特殊 Git 历史治理方式，应由团队明确制定独立安全规则，不应让 Codex 自行决定。

---

# 七、Integration 与 Runtime Acceptance 的关系

```text
GPT Review ACCEPTED
→ 说明固定源码 Delta 可以进入下一阶段

Integration PASS
→ 说明源码已经安全进入正式集成状态

Runtime Acceptance PASS
→ 说明真实运行行为正确

Outcome Acceptance PASS
→ 说明最终业务 / 用户结果正确
```

它们互相不能替代。

---

# 八、没有 Runtime 的项目

例如：

- 纯库
- 文档
- 本地工具
- 不存在长期正式运行环境的代码

可以：

```text
RUNTIME_ACCEPTANCE=NOT_APPLICABLE
```

不要为了形式创建一个虚假的 Runtime 阶段。

---

# 九、参考命令｜仅示意

以下只是思路，不是强制脚本：

```bash
git fetch origin
git checkout main
git pull --ff-only origin main
git status

git merge --ff-only <review-branch>

git push origin main
```

在真实项目中，应根据你的 branch policy、权限和部署规则调整。

---

# 十、核心原则

> **Integration 的目标不是“把代码合进去”，而是保证最终进入正式源码的变化仍然是 GPT 已经审过、且可以被后续 Runtime 验证的变化。**
