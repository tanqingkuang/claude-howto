<picture>
<source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# 高级功能

全面介绍 Claude Code 的高级功能，包括规划模式、扩展思维、自动模式、后台任务、权限模式、打印模式（非交互）、会话管理、交互功能、通道、语音听写、远程控制、Web 会话、桌面应用程序、任务列表、提示建议、git 工作树、沙箱、托管设置和配置。

## 目录

1. [概述](#概述)
2. [规划模式](#规划模式)
3. [扩展思维](#扩展思维)
4. [自动模式](#自动模式)
5. [后台任务](#后台任务)
6. [定时任务](#定时任务)
7. [权限模式](#权限模式)
8. [无头模式](#无头模式)
9. [会话管理](#会话管理)
10. [交互功能](#交互功能)
11. [语音听写](#语音听写)
12. [消息通道](#消息通道channels)
13. [Chrome 集成](#chrome-集成)
14. [远程控制](#远程控制)
15. [Web 会话](#Web 会话)
16. [桌面应用程序](#桌面应用程序)
17. [任务列表](#任务列表)
18. [提示建议](#提示建议)
19. [Git 工作树](#git-工作树)
20. [沙盒](#沙盒)
21. [托管设置（企业）](#托管设置企业)
22. [配置和设置](#配置和设置)
23. [最佳实践](#最佳实践)
24. [其他资源](#其他资源)

---

## 概述

Claude Code 中的高级功能通过规划、推理、自动化和控制机制扩展了核心功能。这些功能为复杂的开发任务、代码审查、自动化和多会话管理提供了复杂的工作流程。

主要高级功能包括：
- 规划模式：在编码之前创建详细的实施计划
- 扩展思维：复杂问题的深度推理
- 自动模式：后台安全分类器在执行前检查每个操作（研究预览）
- 后台任务：运行长时间操作而不阻塞对话
- 权限模式：控制Claude可以做什么（`default`、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions`）
- 打印模式：以非交互方式运行 Claude Code 以实现自动化和 CI/CD (`claude -p`)
- 会话管理：管理多个工作会话
- 交互功能：键盘快捷键、多行输入和命令历史记录
- 语音听写：即按即说语音输入，支持 20 种语言 STT
- 通道：MCP 服务器将消息推送到正在运行的会话中（研究预览）
- 远程控制：从 Claude.ai 或 Claude 应用程序控制 Claude Code
- Web 会话：在浏览器中运行 Claude Code，网址为 claude.ai/code
- 桌面应用程序：用于视觉差异审查和多个会话的独立应用程序
- 任务列表：跨上下文压缩的持久任务跟踪
- 提示建议：基于上下文的智能命令建议
- Git 工作树：用于并行工作的独立工作树分支
- 沙箱：操作系统级文件系统和网络隔离
- 托管设置：通过 plist、注册表或托管文件进行企业部署
- 配置：使用 JSON 配置文件自定义行为

---

## 规划模式

规划模式允许Claude在实施复杂的任务之前仔细考虑它们，创建一个可供您审查和批准的详细计划。

### 什么是规划模式？

规划模式是一个两阶段的方法：
1. 规划阶段：Claude分析任务并制定详细的实施计划
2. 实施阶段：批准后，Claude执行计划

### 何时使用规划模式

✅ 使用规划模式：
- 复杂的多文件重构
- 新功能的实现
- 架构变化
- 数据库迁移
- 主要 API 重新设计

❌ 不要将规划模式用于：
- 简单的错误修复
- 格式更改
- 单文件编辑
- 快速查询

### 激活规划模式

斜线命令：
```bash
/plan Implement user authentication system
```

CLI 标志：
```bash
claude --permission-mode plan
```

设置为默认值：
```json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

键盘快捷键：
- `Shift + Tab` - 切换权限模式（包括计划）
- `Alt + M` - 切换权限模式 (Windows/Linux)

### 规划模式的好处

- 结构化实施方法：实施步骤的详细分解
- 审核和批准：执行前批准或调整计划
- 风险识别：在实施前识别潜在问题
- 明确的阶段：有组织的实施阶段和里程碑

### 示例：功能实现

```
User: /plan Add a real-time notification system

Claude: I'll create a plan for implementing real-time notifications.

## 实现计划

### 阶段 1：基础设施搭建
1. Install WebSocket libraries (socket.io)
2. Set up WebSocket server
3. Configure connection management

### 阶段 2：后端实现
4. Create notification service
5. Implement notification queue
6. Add database schema for notifications
7. Create API endpoints

### 阶段 3：前端集成
8. Set up WebSocket client
9. Create notification UI components
10. Implement notification state management
11. Add notification permissions

### 阶段 4：测试与部署
12. Write unit tests
13. Write integration tests
14. Update documentation
15. Deploy to staging

Estimated time: 3-4 hours
Files to modify: 12
New files to create: 8

Ready to proceed? (yes/no/modify plan)

User: yes

Claude: [Begins implementation following the plan]
```

### 计划修改

```
User: Modify the plan - skip the queue for now, we'll add it later

Claude: Updated plan:
[Shows modified plan with queue removed]

User: Looks good, proceed

Claude: [Implements modified plan]
```

### 规划模式配置

规划模式通过 CLI 标志或斜杠命令激活：

```bash
# Activate plan mode via CLI
claude --permission-mode plan

# Or use the /plan slash command inside the REPL
/plan Implement user authentication system
```

用于规划的模型别名：使用 `opusplan` 作为模型别名，以使用 Opus 进行规划并使用 Sonnet 进行执行：

```bash
claude --model opusplan "design and implement the new API"
```

外部编辑计划：按 `Ctrl+G` 在外部编辑器中打开当前计划以进行详细修改。

---

## 扩展思维

扩展思维使Claude能够在提供解决方案之前花更多时间推理复杂问题。

### 什么是扩展思维？

扩展思维是一个深思熟虑的、逐步的推理过程，Claude：
- 分解复杂的问题
- 考虑多种方法
- 评估权衡
- 通过边缘案例分析原因

### 激活扩展思维

键盘快捷键：
- `Option + T` (macOS) / `Alt + T` (Windows/Linux) - 切换扩展思维

自动激活：
- 默认情况下对所有模型启用（Opus 4.6、Sonnet 4.6、Haiku 4.5）
- Opus 4.6：具有努力级别的自适应推理：`low` (○)、`medium` (◐)、`high` (●)、`max`（仅限 Opus 4.6）
- 其他模型：固定预算最多 31,999 个代币

配置方法：
- 切换：`Alt+T` / `Option+T`，或通过 `/config`
- 查看推理：`Ctrl+O`（详细模式）
- 设置努力：`/effort` 命令或 `--effort` 标志

定制预算：
```bash
export MAX_THINKING_TOKENS=1024
```

努力程度（仅限 Opus 4.6）：
```bash
export CLAUDE_CODE_EFFORT_LEVEL=high   # low (○), medium (◐), high (●), or max (Opus 4.6 only)
```

CLI 标志：
```bash
claude --effort high "complex architectural review"
```

斜线命令：
```
/effort high
```

> 注意：提示中的关键字"ultrathink"将激活深度推理模式。努力程度 `low`、`medium`、`high` 和 `max`（仅限 Opus 4.6）控制Claude执行推理的程度。

### 扩展思维的好处

- 深度推理：花更多时间思考复杂问题
- 更好的架构决策：通过综合评估做出明智的选择
- 考虑边缘情况：思考潜在的问题和场景
- 彻底的分析：解决复杂问题的结构化方法

### 示例：架构决策

```
User: /think Should we use microservices or a monolith for our e-commerce platform?

Claude: Let me think through this architectural decision carefully...

<extended_thinking>
## Context Analysis
- Team size: 5 developers
- Expected traffic: 10k daily users initially
- Need to scale to 100k+ users
- Limited DevOps resources
- 6-month timeline to MVP

## Approach 1: Microservices
Pros:
+ Independent scaling
+ Technology flexibility
+ Team independence
+ Fault isolation

Cons:
- Operational complexity
- Network overhead
- Distributed tracing needed
- Requires DevOps expertise
- Slower initial development

## Approach 2: Monolith
Pros:
+ Simpler deployment
+ Faster initial development
+ Easier debugging
+ Lower operational overhead
+ Single codebase

Cons:
- Scaling challenges later
- All-or-nothing deployments
- Technology lock-in
- Potential for tight coupling

## Decision Factors
1. Team size (5 devs) → Favors monolith
2. DevOps resources (limited) → Favors monolith
3. Timeline (6 months) → Favors monolith
4. Future scale (100k users) → Achievable with monolith + caching
5. Complexity budget → Limited

## Recommendation
Start with a **modular monolith**:
- Single deployment unit
- Clear module boundaries
- Clean interfaces between modules
- Prepare for future extraction to microservices if needed

This gives you:
✓ Fast initial development
✓ Low operational complexity
✓ Option to extract services later
✓ Clear architecture

Migration path: When you reach 50k+ users or specific modules need independent scaling, extract those modules into microservices.
</extended_thinking>

**Recommendation**: Implement a modular monolith with clean module boundaries.

This approach balances your current constraints (team size, timeline, DevOps resources) with future scalability needs. You can migrate specific modules to microservices later when there's a clear business need.
```

### 扩展思维配置

扩展思维是通过环境变量、键盘快捷键和 CLI 标志来控制的：

```bash
# Set thinking token budget
export MAX_THINKING_TOKENS=16000

# Set effort level (Opus 4.6 only): low (○), medium (◐), high (●), or max (Opus 4.6 only)
export CLAUDE_CODE_EFFORT_LEVEL=high
```

在会话期间使用 `Alt+T` / `Option+T` 进行切换，使用 `/effort` 设置工作量，或通过 `/config` 进行配置。

---

## 自动模式

自动模式是一种研究预览权限模式（2026 年 3 月），它使用后台安全分类器在执行前审查每个操作。它让Claude能够自主工作，同时阻止危险操作。

### 要求

- 计划：团队计划（企业和API推出）
- 模型：Claude Sonnet 4.6 或 Opus 4.6
- 分类器：在 Claude Sonnet 4.6 上运行（增加额外的代币成本）

### 启用自动模式

```bash
# Unlock auto mode with CLI flag
claude --enable-auto-mode

# Then cycle to it with Shift+Tab in the REPL
```

或者将其设置为默认权限模式：

```bash
claude --permission-mode auto
```

通过配置设置：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 分类器如何工作

背景分类器使用以下决策顺序评估每个动作：

1. 允许/拒绝规则——首先检查显式权限规则
2. 只读/编辑自动批准——文件读取和编辑自动通过
3.分类器——后台分类器审查动作
4. 回退——在连续 3 个或总共 20 个区块后回退到提示

### 默认阻止的操作

自动模式默认阻止以下内容：

|被阻止的行动 |示例|
|----------------|---------|
|管道到外壳安装 | `curl \| bash` |
|外部发送敏感数据 | API 密钥、网络凭据 |
|生产部署|部署针对生产的命令 |
|批量删除 |大型目录上的 `rm -rf` |
| IAM 变化 |权限和角色修改 |
|强制推送到主 | `git push --force origin main` |

### 默认允许的操作

|允许的操作 |示例|
|----------------|---------|
|本地文件操作|读取、写入、编辑项目文件 |
|声明的依赖项安装 |清单中的 `npm install`、`pip install` |
|只读 HTTP | `curl` 用于获取文档 |
|推送到当前分支 | `git push origin feature-branch` |

### 配置自动模式

将默认规则打印为 JSON：
```bash
claude auto-mode defaults
```

通过企业部署的 `autoMode.environment` 托管设置配置可信基础设施。这允许管理员定义可信的 CI/CD 环境、部署目标和基础设施模式。

### 后备行为

当分类器不确定时，自动模式会回退到提示用户：
- 连续 3 个分类器块之后
- 会话中总共 20 个分类器块之后

这可以确保当分类器无法自信地批准某个操作时，用户始终保留控制权。

### 播种自动模式等效权限（无需团队计划）

如果您没有团队计划或想要一种没有背景分类器的更简单方法，则可以使用保守的安全权限规则基线为 `~/.claude/settings.json` 播种。该脚本以只读和本地检查规则开始，然后让您仅在需要时选择编辑、测试、本地 git 写入、包安装和 GitHub 写入操作。

文件：`09-advanced-features/setup-auto-mode-permissions.py`

```bash
# Preview what would be added (no changes written)
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# Apply the conservative baseline
python3 09-advanced-features/setup-auto-mode-permissions.py

# Add more capability only when you need it
python3 09-advanced-features/setup-auto-mode-permissions.py --include-edits --include-tests
python3 09-advanced-features/setup-auto-mode-permissions.py --include-git-write --include-packages
```

该脚本添加了跨这些类别的规则：

|类别 |示例 |
|----------|---------|
|核心只读工具 | `Read(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)`、`WebFetch(*)` |
|当地考察| `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)`、`Bash(cat:*)` |
|可选编辑 | `Edit(*)`、`Write(*)`、`NotebookEdit(*)` |
|可选测试/构建 | `Bash(pytest:*)`、`Bash(python3 -m pytest:*)`、`Bash(cargo test:*)` |
|可选的 git 写入 | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git stash:*)` |
| Git（本地写入）| `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
|包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
|构建和测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
|普通外壳 | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

有意排除危险操作（`rm -rf`、`sudo`、强制推送、`DROP TABLE`、`terraform destroy` 等）。该脚本是幂等的——运行两次不会重复规则。

---

## 后台任务

后台任务允许执行长时间运行的操作，而不会阻塞您的对话。

### 什么是后台任务？

当您继续工作时，后台任务会异步运行：
- 长测试套件
- 构建流程
- 数据库迁移
- 部署脚本
- 分析工具

基本用法：
```bash
User: 在后台运行测试

Claude: 已启动任务 bg-1234

/task list           # 显示所有任务
/task status bg-1234 # 查看进度
/task show bg-1234   # 查看输出
/task cancel bg-1234 # 取消任务
```

### 启动后台任务

```
User: 在后台运行完整测试套件

Claude: 正在后台启动测试（task-id: bg-1234）
测试运行期间你可以继续工作。

[你可以继续当前对话]

User: 与此同时，我们来重构 auth 模块

Claude: [在后台测试运行时处理 auth 模块]

[稍后，当测试完成]

Claude: 📢 后台任务 bg-1234 已完成：
✅ 245 个测试通过
❌ 3 个测试失败
查看结果：/task show bg-1234
```

### 管理后台任务

列出活动任务：
```
User: /task list

活动中的后台任务：
1. [bg-1234] Running tests (50% complete, 2min remaining)
2. [bg-1235] Building Docker image (25% complete, 8min remaining)
3. [bg-1236] Deploying to staging (90% complete, 30sec remaining)
```

检查任务状态：
```
User: /task status bg-1234

任务 bg-1234：运行测试
状态：进行中
进度：120/245 个测试（49%）
开始时间：2025-11-08 10:30:15
预计完成：2025-11-08 10:34:22
```

查看任务输出：
```
User: /task show bg-1234

[显示测试运行的实时输出]
```

取消任务：
```
User: /task cancel bg-1234

已取消后台任务 bg-1234
```

### 示例：并行开发

```
User: 在后台运行构建

Claude: 正在启动构建……（task-id: bg-5001）

User: 也把 linter 放到后台运行

Claude: 正在启动 linter……（task-id: bg-5002）

User: 在它们运行期间，我们来实现新的 API 端点

Claude: [在构建和 linter 运行时实现 API 端点]

[10 分钟后]

Claude: 📢 构建已成功完成（bg-5001）
📢 Linter 发现了 12 个问题（bg-5002）

User: 把 linter 的问题给我看

Claude: [展示 bg-5002 的 linter 输出]
```

### 配置

```json
{
  "backgroundTasks": {
    "enabled": true,
    "maxConcurrentTasks": 5,
    "notifyOnCompletion": true,
    "autoCleanup": true,
    "logOutput": true
  }
}
```

---

## 定时任务

定时任务可让您按照定期计划自动运行提示或作为一次性提醒。任务是会话范围的——它们在 Claude Code 处于活动状态时运行，并在会话结束时被清除。自 v2.1.72+ 起可用。

### `/loop` 命令

```bash
# Explicit interval
/loop 5m check if the deployment finished

# Natural language
/loop check build status every 30 minutes
```

还支持标准 5 字段 cron 表达式以进行精确调度。

### 一次性提醒

设置在特定时间触发一次的提醒：

```
remind me at 3pm to push the release branch
in 45 minutes, run the integration tests
```

### 管理定时任务

|工具|描述 |
|------|-------------|
| `CronCreate` |创建新的定时任务 |
| `CronList` |列出所有活动的定时任务 |
| `CronDelete` |删除定时任务 |

限制和行为：
- 每个会话最多 50 个定时任务
- 会话范围 - 会话结束时清除
- 重复任务在 3 天后自动过期
- 任务仅在 Claude Code 运行时触发 - 无法追赶错过的触发

### 行为细节

|方面|详情 |
|--------|--------|
|反复出现的抖动|最多 10% 的间隔（最多 15 分钟）|
|一次性抖动 |在 :00/:30 边界上最长 90 秒 |
|错过火灾|没有追赶——如果 Claude Code 没有运行则跳过 |
|坚持|重新启动后不保留|

### 云定时任务

使用 `/schedule` 创建在 Anthropic 基础设施上运行的云定时任务：

```
/schedule daily at 9am run the test suite and report failures
```

云定时任务在重新启动后仍然存在，并且不需要 Claude Code 在本地运行。

### 禁用定时任务

```bash
export CLAUDE_CODE_DISABLE_CRON=1
```

### 示例：监控部署

```
/loop 5m check the deployment status of the staging environment.
        If the deploy succeeded, notify me and stop looping.
        If it failed, show the error logs.
```

> 提示：定时任务是会话范围的。对于在重新启动后仍然存在的持久自动化，请改用 CI/CD 管道、GitHub Actions 或桌面应用程序定时任务。

---

## 权限模式

权限模式控制Claude可以在没有明确批准的情况下执行哪些操作。

### 可用的权限模式

|模式|行为 |
|---|---|
| `default` |只读取文件；提示所有其他操作 |
| `acceptEdits` |读取和编辑文件；命令提示 |
| `plan` |仅读取文件（研究模式，无编辑）|
| `auto` |所有操作均带有后台安全分类器检查（研究预览）|
| `bypassPermissions` |所有操作，无权限检查（危险）|
| `dontAsk` |仅执行预先批准的工具；所有其他人均否认|

在 CLI 中使用 `Shift+Tab` 循环切换模式。使用 `--permission-mode` 标志或 `permissions.defaultMode` 设置设置默认值。

### 激活方法

键盘快捷键：
```bash
Shift + Tab  # Cycle through all 6 modes
```

斜线命令：
```bash
/plan                  # Enter plan mode
```

CLI 标志：
```bash
claude --permission-mode plan
claude --permission-mode auto
```

环境：
```json
{
  "permissions": {
    "defaultMode": "auto"
  }
}
```

### 权限模式示例

#### 默认模式
Claude要求确认重大行动：

```
User: 修复 auth.ts 中的 bug

Claude: 我需要修改 src/auth.ts 来修复这个 bug。
这次改动会更新密码校验逻辑。

批准这项变更吗？（yes/no/show）
```

#### 规划模式
执行前审查实施计划：

```
User: /plan Implement user authentication system

Claude: 我会为身份认证实现创建一份计划。

## 实现计划
[包含阶段和步骤的详细计划]

准备继续吗？（yes/no/modify）
```

#### 接受编辑模式
自动接受文件修改：

```
User: acceptEdits
User: 修复 auth.ts 中的 bug

Claude: [Makes changes without asking]
```

### 用例

代码审查：
```
User: claude --permission-mode plan
User: 审查这个 PR 并给出改进建议

Claude: [读取代码并给出反馈，但不能修改]
```

结对编程：
```
User: claude --permission-mode default
User: 我们一起来实现这个功能

Claude: [在每次改动前请求批准]
```

自动化任务：
```
User: claude --permission-mode acceptEdits
User: 修复整个代码库中的所有 lint 问题

Claude: [自动接受文件编辑，无需询问]
```

---

## 无头模式

打印模式 (`claude -p`) 允许 Claude Code 无需交互式输入即可运行，非常适合自动化和 CI/CD。这是非交互模式，取代了旧的 `--headless` 标志。

### 什么是打印模式？

打印模式可以：
- 自动执行脚本
- CI/CD 集成
- 批量处理
- 定时任务

### 在打印模式下运行（非交互式）

```bash
# 运行指定任务
claude -p "运行全部测试"

# 处理管道输入内容
cat error.log | claude -p "分析这些错误"

# CI/CD 集成（GitHub Actions）
- name: AI Code Review
  run: claude -p "审查 PR"
```

### 其他打印模式使用示例

```bash
# 运行指定任务并捕获输出
claude -p "运行全部测试并生成覆盖率报告"

# 使用结构化输出
claude -p --output-format json "分析代码质量"

# 通过 stdin 输入
echo "Analyze code quality" | claude -p "解释这段内容"
```

### 示例：CI/CD 集成

**GitHub Actions：**
```yaml
# .github/workflows/code-review.yml
name: AI Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Claude Code
        run: npm install -g @anthropic-ai/claude-code

      - name: Run Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          claude -p --output-format json \
            --max-turns 3 \
            "请从以下方面审查这个 PR：
            - 代码质量问题
            - 安全漏洞
            - 性能隐患
            - 测试覆盖率
            以 JSON 输出结果" > review.json

      - name: Post Review Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = JSON.parse(fs.readFileSync('review.json', 'utf8'));
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: JSON.stringify(review, null, 2)
            });
```

### 打印模式配置

打印模式 (`claude -p`) 支持多个自动化标志：

```bash
# Limit autonomous turns
claude -p --max-turns 5 "重构这个模块"

# Structured JSON output
claude -p --output-format json "分析这个代码库"

# With schema validation
claude -p --json-schema '{"type":"object","properties":{"issues":{"type":"array"}}}' \
  "找出这段代码中的 bug"

# Disable session persistence
claude -p --no-session-persistence "一次性分析"
```

---

## 会话管理

有效管理多个Claude Code 会话。

### 会话管理命令

|命令|描述 |
|---------|-------------|
| `/resume` |按 ID 或姓名恢复对话 |
| `/rename` |命名当前会话 |
| `/fork` |将当前会话分叉到一个新分支 |
| `claude -c` |继续最近的对话 |
| `claude -r "session"` |按名称或 ID 恢复会话 |

### 恢复会话

继续上次对话：
```bash
claude -c
```

恢复指定会话：
```bash
claude -r "auth-refactor" "finish this PR"
```

重命名当前会话（在 REPL 内）：
```
/rename auth-refactor
```

### 分叉会话

分叉一个会话来尝试替代方法而不丢失原始方法：

```
/fork
```

或者从 CLI：
```bash
claude --resume auth-refactor --fork-session "try OAuth instead"
```

### 会话持续性

会话会自动保存并可以恢复：

```bash
# Continue last conversation
claude -c

# Resume specific session by name or ID
claude -r "auth-refactor"

# Resume and fork for experimentation
claude --resume auth-refactor --fork-session "alternative approach"
```

---

## 互动功能

### 键盘快捷键

Claude Code 支持键盘快捷键以提高效率。这是官方文档的完整参考：

|快捷方式 |描述 |
|----------|-------------|
| `Ctrl+C` |取消当前输入/生成|
| `Ctrl+D` |退出Claude Code |
| `Ctrl+G` |在外部编辑器中编辑计划 |
| `Ctrl+L` |清除终端屏幕|
| `Ctrl+O` |切换详细输出（查看推理）|
| `Ctrl+R` |反向搜索历史记录 |
| `Ctrl+T` |切换任务列表视图 |
| `Ctrl+B` |后台运行任务|
| `Esc+Esc` |倒回代码/对话 |
| `Shift+Tab` / `Alt+M` |切换权限模式 |
| `Option+P` / `Alt+P` |开关模型|
| `Option+T` / `Alt+T` |切换扩展思维 |

行编辑（标准阅读行快捷键）：

|快捷方式 |行动|
|----------|--------|
| `Ctrl + A` |移至行首 |
| `Ctrl + E` |移至行尾 |
| `Ctrl + K` |切到行尾 |
| `Ctrl + U` |切至行首 |
| `Ctrl + W` |向后删除单词 |
| `Ctrl + Y` |粘贴（猛拉）|
| `Tab` |自动完成 |
| `↑ / ↓` |命令历史 |

### 自定义键绑定

通过运行 `/keybindings` 创建自定义键盘快捷键，这会打开 `~/.claude/keybindings.json` 进行编辑 (v2.1.18+)。

配置格式：

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null,
        "ctrl+k ctrl+s": "chat:stash"
      }
    },
    {
      "context": "Confirmation",
      "bindings": {
        "ctrl+a": "confirmation:yes"
      }
    }
  ]
}
```

将绑定设置为 `null` 以取消绑定默认快捷方式。

### 可用上下文

键绑定的范围仅限于特定的 UI 上下文：

|背景 |关键行动|
|---------|-------------|
|聊天 | `submit`、`cancel`、`cycleMode`、`modelPicker`、`thinkingToggle`、`undo`、`externalEditor`、`stash`、`imagePaste` |
|确认| `yes`、`no`、`previous`、`next`、`nextField`、`cycleMode`、`toggleExplanation` |
|全球| `interrupt`、`exit`、`toggleTodos`、`toggleTranscript` |
|自动完成 | `accept`、`dismiss`、`next`、`previous` |
|历史搜索 | `search`、`previous`、`next` |
|设置|特定于上下文的设置导航 |
|标签 |选项卡切换与管理 |
|帮助 |帮助面板导航 |

总共有 18 个上下文，包括 `Transcript`、`Task`、`ThemePicker`、`Attachments`、`Footer`、`MessageSelector`、`DiffDialog`、`ModelPicker` 和 `Select`。

### 和弦支持

键绑定支持和弦序列（多键组合）：

```
"ctrl+k ctrl+s"   → Two-key sequence: press ctrl+k, then ctrl+s
"ctrl+shift+p"    → Simultaneous modifier keys
```

击键语法：
- 修饰符：`ctrl`、`alt`（或 `opt`）、`shift`、`meta`（或 `cmd`）
- 大写意味着 Shift：`K` 相当于 `shift+k`
- 特殊键：`escape`、`enter`、`return`、`tab`、`space`、`backspace`、`delete`、方向键

### 保留键和冲突键

|关键|状态 |笔记|
|-----|--------|--------|
| `Ctrl+C` |保留 |不可反弹（中断）|
| `Ctrl+D` |保留 |无法反弹（退出）|
| `Ctrl+B` |终端冲突| tmux 前缀键 |
| `Ctrl+A` |终端冲突| GNU Screen 前缀键 |
| `Ctrl+Z` |终端冲突 |进程挂起|

> 提示：如果快捷方式不起作用，请检查与终端仿真器或多路复用器是否存在冲突。

### 制表符补全

Claude Code 提供智能制表符补全：

```
User: /rew<TAB>  # 输入缩写
→ /rewind

User: /plu<TAB>  # 输入缩写
→ /plugin

User: /plugin <TAB>
→ /plugin install
→ /plugin enable
→ /plugin disable
```

### 命令历史

访问之前的命令：

```
User: <↑>  # 上一条命令
User: <↓>  # 下一条命令
User: Ctrl+R  # 搜索历史

(reverse-i-search)`test': run all tests
```

### 多行输入

对于复杂的查询，使用多行模式：

```bash
User: \
> 一段复杂的长提示
> 跨越多行
> \end
```

例子：

```
User: \
> 实现一个用户身份认证系统
> 具备以下要求：
> - JWT 令牌
> - 邮箱验证
> - 密码重置
> - 支持 2FA
> \end

Claude: [处理这条多行请求]
```

### 内联编辑

发送前编辑命令：

```
User: Deploy to prodcution<Backspace><Backspace>uction

[发送前原地编辑]
```

### Vim 模式

启用 Vi/Vim 键绑定进行文本编辑：

激活：
- 使用 `/vim` 命令或 `/config` 启用
- 模式切换为 `Esc` 为 NORMAL，`i/a/o` 为 INSERT

导航键：
- `h` / `l` - 左/右移动
- `j` / `k` - 向下/向上移动
- `w` / `b` / `e` - 按字移动
- `0` / `$` - 移至行开头/结尾
- `gg` / `G` - 跳转到文本的开头/结尾

文本对象：
- `iw` / `aw` - 内部/周围字
- `i"` / `a"` - 内部/周围引用的字符串
- `i(` / `a(` - 内/括号内

### 重击模式

直接使用 `!` 前缀执行 shell 命令：

```bash
! npm test  # 运行测试
! git status  # 查看状态
! cat src/index.js  # 查看文件
```

使用它可以快速执行命令，而无需切换上下文。

---

## 语音听写

语音听写为 Claude Code 提供即按即说语音输入，使您可以说出提示而不是键入提示。

### 激活语音听写

```
/voice
```

### 特征

|特色 |描述 |
|---------|-------------|
|一键通 |按住按键录音，松开发送|
| 20 种语言 |语音转文本支持 20 种语言 |
|自定义按键绑定 |通过 `/keybindings` 配置一键通键 |
|账户要求 |需要 Claude.ai 帐户才能进行 STT 处理 |

### 配置

在键绑定文件 (`/keybindings`) 中自定义一键通键绑定。语音听写使用您的 Claude.ai 帐户进行语音到文本的处理。

---

## 消息通道（Channels）

消息通道（研究预览）允许 MCP 服务器将消息推送到正在运行的 Claude Code 会话中，从而实现与外部服务的实时集成。

### 订阅消息通道

```bash
# Subscribe to channel plugins at startup
claude --channels discord,telegram
```

### 支持的集成

|整合 |描述 |
|-------------|-------------|
|不和谐 |在您的会话中接收和回复 Discord 消息 |
|电报 |在您的会话中接收并回复 Telegram 消息 |

### 配置

企业部署的托管设置：

```json
{
  "allowedChannelPlugins": ["discord", "telegram"]
}
```

`allowedChannelPlugins` 托管设置控制整个组织允许哪些消息通道插件。

### 它是如何运作的

1. MCP服务器充当连接外部服务的消息通道插件
2. 传入消息被推送到活动的 Claude Code 会话中
3. Claude 可以在会话上下文中阅读和回复消息
4. 消息通道插件必须通过 `allowedChannelPlugins` 托管设置批准

---

## Chrome 集成

Chrome 集成将 Claude Code 连接到您的 Chrome 或 Microsoft Edge 浏览器，以实现实时 Web 自动化和调试。这是自 v2.0.73+ 起提供的测试版功能（v1.0.36+ 中添加了 Edge 支持）。

### 启用 Chrome 集成

启动时：

```bash
claude --chrome      # Enable Chrome connection
claude --no-chrome   # Disable Chrome connection
```

在一个会话中：

```
/chrome
```

选择"默认启用"可为所有未来会话激活 Chrome 集成。 Claude Code 共享您浏览器的登录状态，因此它可以与经过身份验证的 Web 应用程序进行交互。

### 能力

|能力|描述 |
|------------|-------------|
|现场调试|实时读取控制台日志、检查 DOM 元素、调试 JavaScript |
|设计验证|将渲染页面与设计模型进行比较 |
|表单验证 |测试表单提交、输入验证和错误处理 |
| Web 应用程序测试 |与经过身份验证的应用程序交互（Gmail、Google Docs、Notion 等）|
|数据提取|从网页中抓取和处理内容 |
|会议录音|将浏览器交互记录为 GIF 文件 |

### 站点级权限

Chrome 扩展程序管理每个站点的访问。通过扩展弹出窗口随时授予或撤销对特定站点的访问权限。Claude Code 仅与您明确允许的网站进行交互。

### 它是如何工作的

Claude Code 在可见窗口中控制浏览器 - 您可以实时观看操作发生。当浏览器遇到登录页面或验证码时，Claude 会暂停并等待您手动处理后再继续。

### 已知限制

- 浏览器支持：仅限 Chrome 和 Edge — 不支持 Brave、Arc 和其他 Chromium 浏览器
- WSL：在适用于 Linux 的 Windows 子系统中不可用
- 第三方提供商：Bedrock、Vertex 或 Foundry API 提供商不支持
- Service Worker 空闲：Chrome 扩展 Service Worker 可能会在长时间会话期间处于空闲状态

> 提示：Chrome 集成是测试版功能。浏览器支持可能会在未来版本中扩展。

---

## 遥控

远程控制可让您从手机、平板电脑或任何浏览器继续本地运行的 Claude Code 会话。您的本地会话继续在您的计算机上运行 - 没有任何内容移动到云端。适用于 Pro、Max、Team 和 Enterprise 计划 (v2.1.51+)。

### 启动远程控制

从 CLI：

```bash
# Start with default session name
claude remote-control

# Start with a custom name
claude remote-control --name "Auth Refactor"
```

从会话内：

```
/remote-control
/remote-control "Auth Refactor"
```

可用标志：

|标志|描述 |
|------|-------------|
| `--name "title"` |自定义会话标题，易于识别 |
| `--verbose` |显示详细的连接日志 |
| `--sandbox` |启用文件系统和网络隔离 |
| `--no-sandbox` |禁用沙箱（默认）|

### 连接到会话

从其他设备连接的三种方法：

1. 会话 URL — 会话开始时打印到终端；在任何浏览器中打开
2. 二维码 — 开始显示可扫描的二维码后按`spacebar`
3. 按名称查找 — 在 claude.ai/code 或 Claude 移动应用程序 (iOS/Android) 中浏览您的会话

### 安全

- 您的计算机上没有打开入站端口
- 出站 HTTPS 仅通过 TLS
- 范围凭证——多个短期、范围狭窄的令牌
- 会话隔离——每个远程会话都是独立的

### 远程控制与网络上的Claude Code

|方面|远程控制|网络上的Claude Code |
|--------|-------------|--------------------|
|执行 |在您的机器上运行 |在 Anthropic 云上运行 |
|本地工具|对本地 MCP 服务器、文件和 CLI 的完全访问权限没有本地依赖|
|使用案例 |从另一台设备继续本地工作 |从任何浏览器开始 |

### 限制

- 每个 Claude Code 实例一个远程会话
- 终端必须在主机上保持打开状态
- 如果网络无法访问，会话将在大约 10 分钟后超时

### 用例

- 离开办公桌时通过移动设备或平板电脑控制 Claude Code
- 使用更丰富的 claude.ai UI，同时保持本地工具执行
- 在完整的本地开发环境中进行快速代码审查

---

## Web 会话

Web 会话允许您直接在浏览器（claude.ai/code）中运行 Claude Code，或从 CLI 创建 Web 会话。

### 创建Web 会话

```bash
# Create a new web session from the CLI
claude --remote "implement the new API endpoints"
<a id="prompt-suggestions"></a>
## 提示词建议

提示词建议会基于你的 git 历史和当前会话上下文，显示灰色的示例命令。

### 工作方式

- 建议会以灰色文字显示在输入提示下方
- 按 `Tab` 可接受建议
- 按 `Enter` 可接受并立即提交
- 建议具备上下文感知能力，会结合 git 历史和当前对话状态

### 禁用建议

```bash
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
```

---

<a id="git-worktrees"></a>
## Git 工作树（Git Worktrees）

Git Worktrees 允许你在隔离的 worktree 中启动 Claude Code，从而在不同分支上并行工作，而无需 stash 或切换分支。

### 从 Worktree 启动

```bash
# 在隔离的 worktree 中启动 Claude Code
claude --worktree
# 或
claude -w
```

### Worktree 位置

worktree 会创建在：
```
<repo>/.claude/worktrees/<name>
```

### Monorepo 的 Sparse Checkout

使用 `worktree.sparsePaths` 设置可在 monorepo 中执行 sparse-checkout，以减少磁盘使用量和克隆时间：

```json
{
  "worktree": {
    "sparsePaths": ["packages/my-package", "shared/"]
  }
}
```

### Worktree 工具和 Hooks

| 项目 | 说明 |
|------|-------------|
| `ExitWorktree` | 用于退出并清理当前 worktree 的工具 |
| `WorktreeCreate` | 在创建 worktree 时触发的 hook 事件 |
| `WorktreeRemove` | 在移除 worktree 时触发的 hook 事件 |

### 自动清理

如果 worktree 中没有任何更改，会话结束时它会被自动清理。

### 使用场景

- 在保持主分支不变的同时处理功能分支
- 在隔离环境中运行测试，而不影响当前工作目录
- 在可丢弃环境中尝试实验性改动
- 在 monorepo 中仅对特定 package 进行 sparse-checkout，以加快启动

---

<a id="sandboxing"></a>
## 沙盒

沙盒为 Claude Code 执行的 Bash 命令提供操作系统级的文件系统和网络隔离。它与权限规则互补，可提供额外一层安全防护。

### 启用沙盒

**Slash 命令：**
```
/sandbox
```

**CLI 参数：**
```bash
claude --sandbox       # 启用沙盒
claude --no-sandbox    # 禁用沙盒
```

### 配置项

| 设置项 | 说明 |
|---------|-------------|
| `sandbox.enabled` | 启用或禁用沙盒 |
| `sandbox.failIfUnavailable` | 如果无法激活沙盒则直接失败 |
| `sandbox.filesystem.allowWrite` | 允许写入访问的路径 |
| `sandbox.filesystem.allowRead` | 允许读取访问的路径 |
| `sandbox.filesystem.denyRead` | 禁止读取访问的路径 |
| `sandbox.enableWeakerNetworkIsolation` | 在 macOS 上启用较弱的网络隔离 |

### 示例配置

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "filesystem": {
      "allowWrite": ["/Users/me/project"],
      "allowRead": ["/Users/me/project", "/usr/local/lib"],
      "denyRead": ["/Users/me/.ssh", "/Users/me/.aws"]
    },
    "enableWeakerNetworkIsolation": true
  }
}
```

### 工作方式

- Bash 命令会在受限文件系统访问的沙盒环境中运行
- 网络访问可被隔离，以避免意外的外部连接
- 可与权限规则一起使用，形成纵深防御
- 在 macOS 上，使用 `sandbox.enableWeakerNetworkIsolation` 进行网络限制（macOS 不支持完整网络隔离）

### 使用场景

- 安全运行不可信或生成的代码
- 防止意外修改项目外部文件
- 在自动化任务期间限制网络访问

---

<a id="managed-settings-enterprise"></a>
## 企业受管设置

受管设置允许企业管理员通过平台原生管理工具，在整个组织范围内部署 Claude Code 配置。

### 部署方式

| 平台 | 方法 | 自版本起 |
|----------|--------|-------|
| macOS | 受管 plist 文件（MDM） | v2.1.51+ |
| Windows | Windows Registry | v2.1.51+ |
| 跨平台 | 受管配置文件 | v2.1.51+ |
| 跨平台 | 受管 drop-in（`managed-settings.d/` 目录） | v2.1.83+ |

### 受管 Drop-ins

自 v2.1.83 起，管理员可以将多个受管设置文件部署到 `managed-settings.d/` 目录中。文件会按字母顺序合并，因此可在不同团队之间进行模块化配置：

```
~/.claude/managed-settings.d/
  00-org-defaults.json
  10-team-policies.json
  20-project-overrides.json
```

### 可用的受管设置

| 设置项 | 说明 |
|---------|-------------|
| `disableBypassPermissionsMode` | 阻止用户启用 bypass permissions |
| `availableModels` | 限制用户可选模型 |
| `allowedChannelPlugins` | 控制允许使用哪些通道插件 |
| `autoMode.environment` | 为 auto mode 配置可信基础设施 |
| 自定义策略 | 组织专用的权限和工具策略 |

### macOS plist 示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>disableBypassPermissionsMode</key>
  <true/>
  <key>availableModels</key>
  <array>
    <string>claude-sonnet-4-6</string>
    <string>claude-haiku-4-5</string>
  </array>
</dict>
</plist>
```

---

<a id="configuration-and-settings"></a>
## 配置与设置

### 配置文件位置

1. **全局配置**：`~/.claude/config.json`
2. **项目配置**：`./.claude/config.json`
3. **用户配置**：`~/.config/claude-code/settings.json`

### 完整配置示例

**核心高级功能配置：**

```json
{
  "permissions": {
    "mode": "default"
  },
  "hooks": {
    "PreToolUse:Edit": "eslint --fix ${file_path}",
    "PostToolUse:Write": "~/.claude/hooks/security-scan.sh"
  },
  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"]
      }
    }
  }
}
```

**扩展配置示例：**

```json
{
  "permissions": {
    "mode": "default",
    "allowedTools": ["Bash(git log:*)", "Read"],
    "disallowedTools": ["Bash(rm -rf:*)"]
  },

  "hooks": {
    "PreToolUse": [{ "matcher": "Edit", "hooks": ["eslint --fix ${file_path}"] }],
    "PostToolUse": [{ "matcher": "Write", "hooks": ["~/.claude/hooks/security-scan.sh"] }],
    "Stop": [{ "hooks": ["~/.claude/hooks/notify.sh"] }]
  },

  "mcp": {
    "enabled": true,
    "servers": {
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      }
    }
  }
}
```

### 环境变量

可通过环境变量覆盖配置：

```bash
# 模型选择
export ANTHROPIC_MODEL=claude-opus-4-6
export ANTHROPIC_DEFAULT_OPUS_MODEL=claude-opus-4-6
export ANTHROPIC_DEFAULT_SONNET_MODEL=claude-sonnet-4-6
export ANTHROPIC_DEFAULT_HAIKU_MODEL=claude-haiku-4-5

# API 配置
export ANTHROPIC_API_KEY=sk-ant-...

# Thinking 配置
export MAX_THINKING_TOKENS=16000
export CLAUDE_CODE_EFFORT_LEVEL=high

# 功能开关
export CLAUDE_CODE_DISABLE_AUTO_MEMORY=true
export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=true
export CLAUDE_CODE_DISABLE_CRON=1
export CLAUDE_CODE_DISABLE_GIT_INSTRUCTIONS=true
export CLAUDE_CODE_DISABLE_TERMINAL_TITLE=true
export CLAUDE_CODE_DISABLE_1M_CONTEXT=true
export CLAUDE_CODE_DISABLE_NONSTREAMING_FALLBACK=true
export CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION=false
export CLAUDE_CODE_ENABLE_TASKS=true
export CLAUDE_CODE_SIMPLE=true              # 由 --bare 参数设置

# MCP 配置
export MAX_MCP_OUTPUT_TOKENS=50000
export ENABLE_TOOL_SEARCH=true

# 任务管理
export CLAUDE_CODE_TASK_LIST_ID=my-project-tasks

# Agent teams（实验性）
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=true

# Subagent 与 plugin 配置
export CLAUDE_CODE_SUBAGENT_MODEL=sonnet
export CLAUDE_CODE_PLUGIN_SEED_DIR=./my-plugins
export CLAUDE_CODE_NEW_INIT=true

# 子进程与流式处理
export CLAUDE_CODE_SUBPROCESS_ENV_SCRUB="SECRET_KEY,DB_PASSWORD"
export CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=80
export CLAUDE_STREAM_IDLE_TIMEOUT_MS=30000
export ANTHROPIC_CUSTOM_MODEL_OPTION=my-custom-model
export SLASH_COMMAND_TOOL_CHAR_BUDGET=50000
```

### 配置管理命令

```
User: /config
[打开交互式配置菜单]
```

`/config` 命令会提供一个交互式菜单，用于切换以下设置：
- 打开/关闭扩展思考
- verbose 输出
- 权限模式
- 模型选择

### 每个项目的配置

在项目中创建 `.claude/config.json`：

```json
{
  "hooks": {
    "PreToolUse": [{ "matcher": "Bash", "hooks": ["npm test && npm run lint"] }]
  },
  "permissions": {
    "mode": "default"
  },
  "mcp": {
    "servers": {
      "project-db": {
        "command": "mcp-postgres",
        "env": {
          "DATABASE_URL": "${PROJECT_DB_URL}"
        }
      }
    }
  }
}
```

---

## 最佳实践

### Planning Mode
- ✅ 用于复杂的多步骤任务
- ✅ 在批准前审阅计划
- ✅ 需要时修改计划
- ❌ 不要用于简单任务

### Extended Thinking
- ✅ 用于架构决策
- ✅ 用于复杂问题求解
- ✅ 审阅推理过程
- ❌ 不要用于简单查询

### Background Tasks
- ✅ 用于长时间运行的操作
- ✅ 监控任务进度
- ✅ 优雅处理任务失败
- ❌ 不要启动过多并发任务

### Permissions
- ✅ 代码审查使用 `plan`（只读）
- ✅ 交互式开发使用 `default`
- ✅ 自动化工作流使用 `acceptEdits`
- ✅ 带安全护栏的自主工作使用 `auto`
- ❌ 除非绝对必要，否则不要使用 `bypassPermissions`

### Sessions
- ✅ 为不同任务使用不同会话
- ✅ 保存重要会话状态
- ✅ 清理旧会话
- ❌ 不要把不相关的工作混在同一会话中

---

## 更多资源

有关 Claude Code 及相关功能的更多信息：

- [官方交互模式文档](https://code.claude.com/docs/en/interactive-mode)
- [官方无头模式文档](https://code.claude.com/docs/en/headless)
- [CLI 参考](https://code.claude.com/docs/en/cli-reference)
- [Checkpoints 指南](../08-checkpoints/) - 会话管理与回退
- [Slash Commands](../01-slash-commands/) - 命令参考
- [Memory Guide](../02-memory/) - 持久上下文
- [Skills Guide](../03-skills/) - 自主能力
- [Subagents Guide](../04-subagents/) - 委派式任务执行
- [MCP Guide](../05-mcp/) - 外部数据访问
- [Hooks Guide](../06-hooks/) - 事件驱动自动化
- [Plugins Guide](../07-plugins/) - 打包扩展
- [官方定时任务文档](https://code.claude.com/docs/en/scheduled-tasks)
- [官方 Chrome 集成文档](https://code.claude.com/docs/en/chrome)
- [官方远程控制文档](https://code.claude.com/docs/en/remote-control)
- [官方快捷键文档](https://code.claude.com/docs/en/keybindings)
- [官方桌面应用文档](https://code.claude.com/docs/en/desktop)



---

## 示例完整中文译文补充

### 规划模式示例中文译文

```text
User: /plan 添加一个实时通知系统

Claude: 我会为实现实时通知创建一份计划。

## 实现计划

### 阶段 1：基础设施搭建
1. 安装 WebSocket 库（socket.io）
2. 搭建 WebSocket 服务器
3. 配置连接管理

### 阶段 2：后端实现
4. 创建通知服务
5. 实现通知队列
6. 为通知添加数据库 schema
7. 创建 API 端点

### 阶段 3：前端集成
8. 设置 WebSocket 客户端
9. 创建通知 UI 组件
10. 实现通知状态管理
11. 添加通知权限

### 阶段 4：测试与部署
12. 编写单元测试
13. 编写集成测试
14. 更新文档
15. 部署到 staging

预计时间：3-4 小时
需修改文件：12 个
需创建新文件：8 个

准备继续吗？（yes/no/modify plan）

User: yes

Claude: [按该计划开始实现]
```

### 架构决策示例中文译文

```text
User: /think 对于我们的电商平台，我们应该使用微服务还是单体架构？

Claude: 让我仔细思考一下这个架构决策……

<extended_thinking>
## 背景分析
- 团队规模：5 名开发者
- 预期流量：初期每日 1 万用户
- 需要扩展到 10 万以上用户
- DevOps 资源有限
- 距 MVP 上线还有 6 个月

## 方案 1：微服务
优点：
+ 独立扩缩容
+ 技术选型灵活
+ 团队独立性更强
+ 故障隔离更好

缺点：
- 运维复杂度高
- 网络开销更大
- 需要分布式追踪
- 需要 DevOps 专长
- 初期开发更慢

## 方案 2：单体
优点：
+ 部署更简单
+ 初期开发更快
+ 更容易调试
+ 运维开销更低
+ 单一代码库

缺点：
- 后期扩展会更困难
- 部署要么全部一起上，要么全部不动
- 技术栈锁定
- 可能产生紧耦合

## 决策因素
1. 团队规模（5 人）→ 更适合单体
2. DevOps 资源（有限）→ 更适合单体
3. 时间线（6 个月）→ 更适合单体
4. 未来规模（10 万用户）→ 单体 + 缓存仍可支撑
5. 复杂度预算 → 有限

## 建议
从模块化单体开始：
- 单一部署单元
- 清晰的模块边界
- 模块间干净的接口
- 为将来按需拆分成微服务做好准备

这样你可以获得：
- 更快的初期开发速度
- 更低的运维复杂度
- 后续可逐步拆服务
- 更清晰的架构

迁移路径：当用户规模达到 5 万以上，或某些模块需要独立扩展时，再把这些模块提取为微服务。
</extended_thinking>

建议：实现一个带有清晰模块边界的模块化单体。
```

### 后台任务示例中文译文

```text
User: 在后台运行完整测试套件

Claude: 正在后台启动测试（task-id: bg-1234）
测试运行期间你可以继续工作。

User: 与此同时，我们来重构 auth 模块

Claude: [在后台测试运行时处理 auth 模块]

Claude: 后台任务 bg-1234 已完成：
- 245 个测试通过
- 3 个测试失败
- 查看结果：/task show bg-1234
```
