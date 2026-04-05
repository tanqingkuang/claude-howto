<picture>
  <source media="(prefers-color-scheme: dark)" srcset="resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="resources/logos/claude-howto-logo.svg">
</picture>

# 为 Claude How To 贡献内容

感谢你愿意为这个项目做出贡献！本指南将帮助你了解如何高效、顺畅地参与贡献。

## 关于本项目

Claude How To 是一份视觉化、以示例驱动的 Claude Code 指南。我们提供：
- **Mermaid 图示**，帮助解释功能是如何工作的
- **可直接投入使用的模板**，拿来就能用
- **真实世界示例**，包含上下文和最佳实践
- **渐进式学习路径**，从入门到进阶逐步推进

## 贡献类型

### 1. 新示例或模板
为现有功能添加示例（slash commands、skills、hooks 等）：
- 可直接复制粘贴的代码
- 清晰解释它的工作方式
- 使用场景和收益
- 故障排查提示

### 2. 文档改进
- 让容易混淆的部分更清楚
- 修复错别字和语法问题
- 补充缺失信息
- 改进代码示例

### 3. 功能指南
为新的 Claude Code 功能创建指南：
- 分步教程
- 架构图
- 常见模式与反模式
- 真实工作流

### 4. Bug 报告
报告你遇到的问题：
- 说明你预期会发生什么
- 说明实际发生了什么
- 提供复现步骤
- 添加相关的 Claude Code 版本和操作系统信息

### 5. 反馈和建议
帮助改进这份指南：
- 建议更好的解释方式
- 指出覆盖上的空白
- 推荐新增章节或重组结构

## 入门

### 1. Fork 并克隆
```bash
git clone https://github.com/luongnv89/claude-howto.git
cd claude-howto
```

### 2. 创建分支
使用有描述性的分支名：
```bash
git checkout -b add/feature-name
git checkout -b fix/issue-description
git checkout -b docs/improvement-area
```

### 3. 配置环境

pre-commit 钩子会在每次提交前本地运行与 CI 相同的检查。PR 被接受前，这四项检查都必须通过。

**所需依赖：**

```bash
# Python 工具链（uv 是本项目的包管理器）
pip install uv
uv venv
source .venv/bin/activate
uv pip install -r scripts/requirements-dev.txt

# Markdown lint 工具（Node.js）
npm install -g markdownlint-cli

# Mermaid 图表校验工具（Node.js）
npm install -g @mermaid-js/mermaid-cli

# 安装 pre-commit 并激活钩子
uv pip install pre-commit
pre-commit install
```

**验证你的环境：**

```bash
pre-commit run --all-files
```

每次提交都会运行以下 hooks：

| Hook | 检查内容 |
|------|----------|
| `markdown-lint` | Markdown 格式和结构 |
| `cross-references` | 相对链接、锚点、代码围栏 |
| `mermaid-syntax` | 所有 ` ```mermaid ` 块是否能正确解析 |
| `link-check` | 外部 URL 是否可访问 |
| `build-epub` | 在 `.md` 变更时是否能无错误生成 EPUB |

## 目录结构

```
├── 01-slash-commands/      # 用户手动触发的快捷方式
├── 02-memory/              # 持久上下文示例
├── 03-skills/              # 可复用能力
├── 04-subagents/           # 专门化 AI 助手
├── 05-mcp/                 # Model Context Protocol 示例
├── 06-hooks/               # 事件驱动自动化
├── 07-plugins/             # 打包功能
├── 08-checkpoints/         # 会话快照
├── 09-advanced-features/   # 规划、思考、后台任务
├── 10-cli/                 # CLI 参考
├── scripts/                # 构建和工具脚本
└── README-CN.md            # 主指南
```

## 如何贡献示例

### 添加 Slash Command
1. 在 `01-slash-commands/` 中创建一个 `.md` 文件
2. 包含：
   - 清楚说明它做什么
   - 使用场景
   - 安装说明
   - 使用示例
   - 自定义提示
3. 更新 `01-slash-commands/README-CN.md`

### 添加 Skill
1. 在 `03-skills/` 中创建一个目录
2. 包含：
   - `SKILL.md` - 主文档
   - `scripts/` - 如有需要可放辅助脚本
   - `templates/` - 提示词模板
   - 在 README 中加入使用示例
3. 更新 `03-skills/README-CN.md`

### 添加 Subagent
1. 在 `04-subagents/` 中创建一个 `.md` 文件
2. 包含：
   - Agent 的目的和能力
   - 系统提示词结构
   - 示例使用场景
   - 集成示例
3. 更新 `04-subagents/README-CN.md`

### 添加 MCP 配置
1. 在 `05-mcp/` 中创建一个 `.json` 文件
2. 包含：
   - 配置说明
   - 必需的环境变量
   - 安装说明
   - 使用示例
3. 更新 `05-mcp/README-CN.md`

### 添加 Hook
1. 在 `06-hooks/` 中创建一个 `.sh` 文件
2. 包含：
   - Shebang 和说明
   - 解释逻辑的清晰注释
   - 错误处理
   - 安全注意事项
3. 更新 `06-hooks/README-CN.md`

## 编写指南

### Markdown 风格
- 使用清晰的标题层级（章节用 H2，子章节用 H3）
- 保持段落简短且聚焦
- 列表使用项目符号
- 代码块要注明语言
- 各章节之间留空行

### 代码示例
- 示例要能直接复制粘贴
- 为不明显的逻辑加注释
- 同时提供简单版和进阶版
- 展示真实场景用法
- 标出潜在问题

### 文档
- 解释“为什么”，不只是“是什么”
- 包含前置条件
- 增加故障排查章节
- 链接相关主题
- 保持对初学者友好

### JSON/YAML
- 使用正确缩进（统一 2 或 4 个空格）
- 添加解释配置的注释
- 包含校验示例

### 图示
- 尽量使用 Mermaid
- 保持图示简单且可读
- 在图示下方附说明
- 链接相关章节

## 提交指南

请遵循 conventional commit 格式：
```text
type(scope): description

[optional body]
```

类型：
- `feat`：新功能或示例
- `fix`：Bug 修复或修正
- `docs`：文档变更
- `refactor`：代码结构调整
- `style`：格式调整
- `test`：测试新增或修改
- `chore`：构建、依赖等

示例：
```text
feat(slash-commands): Add API documentation generator
docs(memory): Improve personal preferences example
fix(README): Correct table of contents link
docs(skills): Add comprehensive code review skill
```

## 提交前

### 清单
- [ ] 代码符合项目风格和约定
- [ ] 新示例包含清晰文档
- [ ] README 已更新（本地目录和仓库根目录）
- [ ] 没有敏感信息（API keys、credentials）
- [ ] 示例已测试并可正常工作
- [ ] 链接已验证且正确
- [ ] 文件权限正确（脚本可执行）
- [ ] 提交信息清晰且有描述性

### 本地测试
```bash
# 运行所有 pre-commit 检查（与 CI 相同）
pre-commit run --all-files

# 查看你的变更
git diff
```

## PR 流程

1. **创建描述清晰的 PR**：
   - 这增加/修复了什么？
   - 为什么需要它？
   - 相关 issue（如有）

2. **包含相关细节**：
   - 新功能？加入使用场景
   - 文档？说明改进点
   - 示例？展示前后对比

3. **关联 issue**：
   - 使用 `Closes #123` 自动关闭相关 issue

4. **耐心面对审查**：
   - 维护者可能会提出改进建议
   - 根据反馈迭代
   - 最终决定权在维护者手中

## 代码审查流程

审查者会检查：
- **准确性**：它是否如描述那样工作？
- **质量**：是否达到可生产状态？
- **一致性**：是否符合项目模式？
- **文档**：是否清晰完整？
- **安全性**：是否存在漏洞？

## 报告问题

### Bug 报告
请包含：
- Claude Code 版本
- 操作系统
- 复现步骤
- 预期行为
- 实际行为
- 如适用，附上截图

### 功能请求
请包含：
- 使用场景或要解决的问题
- 建议方案
- 你考虑过的替代方案
- 其他上下文

### 文档问题
请包含：
- 哪些地方令人困惑或缺失
- 建议改进
- 示例或参考资料

## 项目政策

### 敏感信息
- 永远不要提交 API keys、tokens 或凭据
- 示例中使用占位符值
- 为配置文件提供 `.env.example`
- 记录所需环境变量

### 代码质量
- 保持示例聚焦且易读
- 避免过度设计
- 为不明显的逻辑加注释
- 在提交前充分测试

### 知识产权
- 原创内容归作者所有
- 项目采用教育许可
- 尊重现有版权
- 在需要时提供署名

## 获取帮助

- **问题**：在 GitHub issue 中发起 discussion
- **一般帮助**：查看现有文档
- **开发帮助**：参考相似示例
- **代码审查**：在 PR 中标记维护者

## 认可

贡献者会在以下位置被记录：
- README-CN.md 的 Contributors 部分
- GitHub contributors 页面
- commit history

## 安全

在贡献示例和文档时，请遵循安全编码实践：

- **永远不要硬编码密钥或 API keys** - 使用环境变量
- **提示安全影响** - 标出潜在风险
- **使用安全默认值** - 默认启用安全特性
- **验证输入** - 展示正确的输入校验和清理
- **包含安全说明** - 记录安全注意事项

有关安全问题，请查看 [SECURITY-CN.md](SECURITY-CN.md) 中的漏洞报告流程。

## 行为准则

我们致力于打造一个热情且包容的社区。请阅读 [CODE_OF_CONDUCT-CN.md](CODE_OF_CONDUCT-CN.md) 了解完整的社区标准。

简要来说：
- 保持尊重和包容
- 体面地接受反馈
- 帮助他人学习和成长
- 避免骚扰或歧视
- 向维护者报告问题

所有贡献者都应遵守本准则，并以善意与尊重对待彼此。

## 许可证

通过为本项目贡献内容，你同意你的贡献将按照 MIT License 进行许可。详情请参阅 [LICENSE](LICENSE) 文件。

## 有问题？

- 查看 [README-CN.md](README-CN.md)
- 阅读 [LEARNING-ROADMAP-CN.md](LEARNING-ROADMAP-CN.md)
- 浏览现有示例
- 提交 issue 进行讨论

感谢你的贡献！🙏

如果你打算提交较大的改动，建议先在 issue 里简要说明你的想法，这样可以先对齐方向，减少来回修改的成本，也更容易让维护者提前给出反馈。
