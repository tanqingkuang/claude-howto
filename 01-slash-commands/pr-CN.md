---
description: 清理代码、暂存变更并准备 PR，确保提交前完成格式化、测试、差异检查和摘要整理
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git diff:*), Bash(npm test:*), Bash(npm run lint:*)
---

# PR 准备清单

在创建 PR 之前，请执行以下步骤，尽量把“提交前需要做的事”一次性收尾，避免把明显问题带进 PR：

1. 运行格式化：`prettier --write .`
2. 运行测试：`npm test`
3. 查看 git diff：`git diff HEAD`
4. 暂存变更：`git add .`
5. 按 conventional commits 创建提交信息：
   - `fix:` bug 修复
   - `feat:` 新功能
   - `docs:` 文档更新
   - `refactor:` 代码重构
   - `test:` 测试新增
   - `chore:` 维护和杂项工作

6. 生成 PR 摘要，包含：
   - 变更了什么
   - 为什么变更
   - 做了哪些测试
   - 可能的影响和后续注意事项
   - 是否还有未覆盖的风险或待办事项
