<picture>
<source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 检查点和回退

检查点允许你保存对话状态，并回退到 Claude Code 会话中的先前节点。这对于探索不同的方法、从错误中恢复或比较替代解决方案非常有价值。

## 概述

检查点允许你保存对话状态，并回退到之前的节点，从而可以安全地试验和探索多种方法。它们是对话状态的快照，包括：
- 交换的所有消息
- 进行文件修改
- 工具使用历史记录
- 会话上下文

在探索不同的方法、从错误中恢复或比较替代解决方案时，检查点都非常有价值。

## 关键概念

|概念 |描述 |
|---------|-------------|
| **检查点** |对话状态快照，包括消息、文件和上下文 |
| **回退** | 返回上一个检查点，放弃后续更改 |
| **分支点** |探索多种方法的检查点 |

## 访问检查点

你可以通过两种主要方式访问和管理检查点：

### 使用键盘快捷键
按`Esc`两次（`Esc` + `Esc`）打开检查点界面并浏览已保存的检查点。

### 使用斜杠命令
使用 `/rewind` 命令（别名：`/checkpoint`）进行快速访问：

```bash
# Open rewind interface
/rewind

# Or use the alias
/checkpoint
```

## 回退选项

当你回退时，会看到一个包含五个选项的菜单：

1. **恢复代码和对话** -- 将文件和消息恢复到该检查点
2. **恢复对话** -- 仅回退消息，保持当前代码不变
3. **恢复代码** -- 仅恢复文件更改，保留完整的对话历史记录
4. **从这里开始总结** -- 将此时的对话压缩为 AI 生成的摘要，而不是丢弃它。原始消息保留在抄本中。你可以选择提供说明，让摘要更聚焦于特定主题。
5. **没关系** -- 取消并返回当前状态

## 自动检查点

Claude Code 会自动为你创建检查点：

- **每个用户提示** - 每个用户输入都会创建一个新的检查点
- **持久** - 检查点在会话中持续存在
- **自动清理** - 检查点在 30 天后自动清理

这意味着你可以随时回退到对话中的任何先前节点，从几分钟前到几天前。

## 用例

|场景 |工作流程|
|----------|----------|
| **探索方法** |保存→尝试A→保存→回退→尝试B→比较|
| **安全重构** |保存 → 重构 → 测试 → 如果失败：回退 |
| **A/B 测试** |保存→设计A→保存→回退→设计B→比较|
| **错误恢复** |通知问题 → 回到最后的良好状态 |

## 使用检查点

### 查看和回退

按 `Esc` 两次或使用 `/rewind` 打开检查点浏览器。你会看到带有时间戳的所有可用检查点列表。选择任何检查点即可回退到该状态。

### 检查点详细信息

每个检查点显示：
- 创建时的时间戳
- 被修改的文件
- 对话中的消息数量
- 使用的工具

## 实际例子

### 示例 1：探索不同的方法

```
User: Let's add a caching layer to the API

Claude: I'll add Redis caching to your API endpoints...
[Makes changes at checkpoint A]

User: Actually, let's try in-memory caching instead

Claude: I'll rewind to explore a different approach...
[User presses Esc+Esc and rewinds to checkpoint A]
[Implements in-memory caching at checkpoint B]

User: Now I can compare both approaches
```

### 示例 2：从错误中恢复

```
User: Refactor the authentication module to use JWT

Claude: I'll refactor the authentication module...
[Makes extensive changes]

User: Wait, that broke the OAuth integration. Let's go back.

Claude: I'll help you rewind to before the refactoring...
[User presses Esc+Esc and selects the checkpoint before the refactor]

User: Let's try a more conservative approach this time
```

### 示例 3：安全实验

```
User: Let's try rewriting this in a functional style
[Creates checkpoint before experiment]

Claude: [Makes experimental changes]

User: The tests are failing. Let's rewind.
[User presses Esc+Esc and rewinds to the checkpoint]

Claude: I've rewound the changes. Let's try a different approach.
```

### 示例 4：分支方法

```
User: I want to compare two database designs
[Takes note of checkpoint - call it "Start"]

Claude: I'll create the first design...
[Implements Schema A]

User: Now let me go back and try the second approach
[User presses Esc+Esc and rewinds to "Start"]

Claude: Now I'll implement Schema B...
[Implements Schema B]

User: Great! Now I have both schemas to choose from
```

## 检查点保留

Claude Code 自动管理你的检查点：

- 根据每个用户提示自动创建检查点
- 旧检查点最多保留 30 天
- 自动清理检查点以防止存储无限增长

## 工作流程模式

### 探索的分支策略

在探索多种方法时：

```
1. Start with initial implementation → Checkpoint A
2. Try Approach 1 → Checkpoint B
3. Rewind to Checkpoint A
4. Try Approach 2 → Checkpoint C
5. Compare results from B and C
6. Choose best approach and continue
```

### 安全重构模式

进行重大更改时：

```
1. Current state → Checkpoint (auto)
2. Start refactoring
3. Run tests
4. If tests pass → Continue working
5. If tests fail → Rewind and try different approach
```

## 最佳实践

由于检查点是自动创建的，因此你可以专注于工作，而不必担心手动保存状态。但是，请记住这些做法：

### 有效使用检查点

✅ **做：**
- 在回退之前查看可用的检查点
- 当你想探索不同的方向时使用回退
- 保留检查点以比较不同的方法
- 了解每个回退选项的作用（恢复代码和对话、恢复对话、恢复代码或总结）

❌ **不要：**
- 仅依靠检查点来保存代码
- 期望检查点跟踪外部文件系统更改
- 使用检查点代替 git 提交

## 配置

你可以在设置中切换自动检查点：

```json
{
  "autoCheckpoint": true
}
```

- `autoCheckpoint`：在每个用户提示时启用或禁用自动检查点创建（默认值：`true`）

## 限制

检查点有以下限制：

- **未跟踪 Bash 命令更改** - 检查点中未捕获文件系统上的 `rm`、`mv`、`cp` 等操作
- **不跟踪外部更改** - 不会捕获 Claude Code 外部（在编辑器、终端等中）所做的更改
- **不是版本控制的替代品** - 使用 git 对代码库进行永久、可审核的更改

## 故障排除

### 缺少检查点

**问题**：未找到预期的检查点

**解决方案**：
- 检查检查点是否已清除
- 验证你的设置中是否启用了 `autoCheckpoint`
- 检查磁盘空间

### 回退失败

**问题**：无法回退到检查点

**解决方案**：
- 确保未提交的更改不会发生冲突
- 检查检查点是否损坏
- 尝试回退到不同的检查点

## 与 Git 集成

检查点补充（但不是取代）git：

| 特色 | git | 检查点 |
|---------|-----|-------------|
| 范围 | 文件系统 | 对话 + 文件 |
| 持久性 | 永久 | 基于会话 |
| 粒度 | 提交 | 任意点 |
| 速度 | 较慢 | 即时 |
| 分享 | 是 | 受限 |

两者一起使用：
1. 使用检查点进行快速实验
2. 使用 git commits 进行最终更改
3. git操作前创建检查点
4.将成功的检查点状态提交到git

## 快速入门指南

### 基本工作流程

1. **正常工作** - Claude Code自动创建检查点
2. **想要返回？** - 按 `Esc` 两次或使用 `/rewind`
3. **选择检查点** - 从列表中选择以回退
4. **选择要恢复的内容** - 选择恢复代码和对话、恢复对话、恢复代码、从此处总结或取消
5. **继续工作** - 你回到了那个点

### 键盘快捷键

- **`Esc` + `Esc`** - 打开检查点浏览器
- **`/rewind`** - 访问检查点的替代方法
- **`/checkpoint`** - `/rewind` 的别名

## 知道何时回退：上下文监控

检查点可以让你回去——但你怎么知道*什么时候*该回退？随着对话的深入，Claude 的上下文窗口会被填满，模型质量也会悄然下降。你可能会在没有意识到的情况下，从一个“半盲”模型里继续产出代码。

**[cc-context-stats](https://github.com/luongnv89/cc-context-stats)** 通过向 Claude Code 状态栏添加实时 **上下文区域** 来解决这个问题。它会跟踪你在上下文窗口中的位置，从**计划**（绿色，可以安全地规划和编码）到**代码**（黄色，避免开启新计划）再到**转储**（橙色，完成并回退）。当你看到区域变化时，就说明该创建检查点并重新开始了，而不是继续用质量下降的输出向前推进。

## 相关概念

- **[高级功能](../09-advanced-features/)** - 规划模式和其他高级功能
- **[内存管理](../02-内存/)** - 管理对话历史记录和上下文
- **[斜线命令](../01-slash-commands/)** - 用户调用的快捷方式
- **[Hooks](../06-hooks/)** - 事件驱动的自动化
- **[插件](../07-plugins/)** - 捆绑的扩展包

## 其他资源

- [官方检查点文档](https://code.claude.com/docs/en/checkpointing)
- [高级功能指南](../09-advanced-features/) - 扩展思维和其他能力

## 概括

检查点是 Claude Code 中的一项自动功能，可以让你安全地探索不同的方法，而不必担心丢失工作。每个用户提示都会自动创建一个新的检查点，因此你可以回退到会话中的任何先前节点。

主要优点：
- 大胆尝试多种方法
- 快速从错误中恢复
- 并排比较不同的解决方案
- 与版本控制系统安全集成

请记住：检查点并不能替代 git。使用检查点进行快速实验，使用 git 进行永久代码更改。
