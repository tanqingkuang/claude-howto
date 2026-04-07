<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# Hooks

Hooks 是在 Claude Code 会话中对特定事件做出响应时自动执行的脚本。它们可用于自动化、校验、权限管理和自定义工作流。

## 概览

Hooks 是自动化动作（shell 命令、HTTP webhook、LLM 提示词或子代理评估），会在 Claude Code 中的特定事件发生时自动执行。它们通过 JSON 接收输入，并通过退出码和 JSON 输出返回结果。

**主要特性：**
- 事件驱动自动化
- 基于 JSON 的输入/输出
- 支持 command、prompt、HTTP 和 agent 四种 hook 类型
- 可针对工具使用模式进行匹配

## 配置

Hooks 在设置文件中按特定结构配置：

- `~/.claude/settings.json` - 用户设置（适用于所有项目）
- `.claude/settings.json` - 项目设置（可共享、可提交）
- `.claude/settings.local.json` - 本地项目设置（不会提交）
- 受管策略 - 组织级设置
- 插件 `hooks/hooks.json` - 插件作用域 hooks
- Skill/Agent frontmatter - 组件生命周期 hooks

### 基础配置结构

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

**关键字段：**

| 字段 | 说明 | 示例 |
|-------|-------------|---------|
| `matcher` | 用于匹配工具名称的模式（区分大小写） | `"Write"`、`"Edit\|Write"`、`"*"` |
| `hooks` | hook 定义数组 | `[{ "type": "command", ... }]` |
| `type` | hook 类型：`"command"`（bash）、`"prompt"`（LLM）、`"http"`（webhook）或 `"agent"`（子代理） | `"command"` |
| `command` | 要执行的 shell 命令 | `"$CLAUDE_PROJECT_DIR/.claude/hooks/format.sh"` |
| `timeout` | 可选超时时间，单位秒（默认 60） | `30` |
| `once` | 如果为 `true`，则每个会话只运行一次 | `true` |

### Matcher 模式

| 模式 | 说明 | 示例 |
|---------|-------------|---------|
| 精确字符串 | 匹配特定工具 | `"Write"` |
| 正则模式 | 匹配多个工具 | `"Edit\|Write"` |
| 通配符 | 匹配所有工具 | `"*"` 或 `""` |
| MCP tools | server 和 tool 模式 | `"mcp__memory__.*"` |

## Hook 类型

Claude Code 支持四种 hook 类型：

### Command Hooks

默认 hook 类型。执行 shell 命令，并通过 JSON stdin/stdout 和退出码通信。

```json
{
  "type": "command",
  "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate.py\"",
  "timeout": 60
}
```

### HTTP Hooks

> 自 v2.1.63 起新增。

远程 webhook 端点，接收与 command hook 相同的 JSON 输入。HTTP hooks 会向 URL 发出 POST JSON 请求并接收 JSON 响应。在启用 sandbox 时，HTTP hooks 会通过 sandbox 进行路由。URL 中的环境变量插值需要显式的 `allowedEnvVars` 列表以保证安全。

```json
{
  "hooks": {
    "PostToolUse": [{
      "type": "http",
      "url": "https://my-webhook.example.com/hook",
      "matcher": "Write"
    }]
  }
}
```

**关键属性：**
- `"type": "http"` -- 表示这是一个 HTTP hook
- `"url"` -- webhook 端点 URL
- 在启用 sandbox 时会通过 sandbox 路由
- URL 中使用任何环境变量插值都需要显式的 `allowedEnvVars` 列表

### 提示词 Hook

由 LLM 评估的提示词，其中 hook 内容本身就是 Claude 要评估的提示词。主要用于 `Stop` 和 `SubagentStop` 事件，以便智能判断任务是否完成。

```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude completed all requested tasks.",
  "timeout": 30
}
```

LLM 会评估提示词并返回结构化决策（详情见 [基于 Prompt 的 Hook](#prompt-based-hooks)）。

### 子代理 Hook

基于子代理的验证 hook，会启动专用 agent 来评估条件或执行复杂检查。与提示词 Hook（单轮 LLM 评估）不同，子代理 Hook 可以使用工具并进行多步推理。

```json
{
  "type": "agent",
  "prompt": "Verify the code changes follow our architecture guidelines. Check the relevant design docs and compare.",
  "timeout": 120
}
```

**关键属性：**
- `"type": "agent"` -- 表示这是一个子代理 hook
- `"prompt"` -- 子代理的任务描述
- agent 可以使用工具（Read、Grep、Bash 等）执行评估
- 返回与提示词 Hook 类似的结构化决策

## Hook 事件

Claude Code 支持 **25 个 hook 事件**：

| 事件 | 触发时机 | Matcher 输入 | 能否阻断 | 常见用途 |
|-------|---------------|---------------|-----------|------------|
| **SessionStart** | 会话开始/恢复/清空/压缩 | startup/resume/clear/compact | 否 | 环境初始化 |
| **InstructionsLoaded** | CLAUDE.md 或规则文件加载后 | （无） | 否 | 修改/过滤指令 |
| **UserPromptSubmit** | 用户提交提示词 | （无） | 是 | 校验提示词 |
| **PreToolUse** | 工具执行前 | 工具名 | 是（allow/deny/ask） | 校验、修改输入 |
| **PermissionRequest** | 显示权限对话框时 | 工具名 | 是 | 自动批准/拒绝 |
| **PostToolUse** | 工具成功后 | 工具名 | 否 | 添加上下文、反馈 |
| **PostToolUseFailure** | 工具执行失败时 | 工具名 | 否 | 错误处理、日志记录 |
| **Notification** | 发送通知时 | 通知类型 | 否 | 自定义通知 |
| **SubagentStart** | 启动子代理时 | agent 类型名 | 否 | 子代理初始化 |
| **SubagentStop** | 子代理完成时 | agent 类型名 | 是 | 子代理验证 |
| **Stop** | Claude 完成回复时 | （无） | 是 | 任务完成检查 |
| **StopFailure** | API 错误结束回合时 | （无） | 否 | 错误恢复、日志记录 |
| **TeammateIdle** | agent 团队成员空闲时 | （无） | 是 | 团队协作协调 |
| **TaskCompleted** | 任务被标记完成时 | （无） | 是 | 任务后动作 |
| **TaskCreated** | 通过 TaskCreate 创建任务时 | （无） | 否 | 任务跟踪、日志记录 |
| **ConfigChange** | 配置文件变化时 | （无） | 是（策略除外） | 响应配置更新 |
| **CwdChanged** | 工作目录变化时 | （无） | 否 | 目录级初始化 |
| **FileChanged** | 监听的文件变化时 | （无） | 否 | 文件监控、重建 |
| **PreCompact** | 上下文压缩之前 | manual/auto | 否 | 压缩前动作 |
| **PostCompact** | 压缩完成后 | （无） | 否 | 压缩后动作 |
| **WorktreeCreate** | 创建 worktree 时 | （无） | 是（路径返回） | worktree 初始化 |
| **WorktreeRemove** | 移除 worktree 时 | （无） | 否 | worktree 清理 |
| **Elicitation** | MCP server 请求用户输入时 | （无） | 是 | 输入校验 |
| **ElicitationResult** | 用户回应 elicitation 时 | （无） | 是 | 响应处理 |
| **SessionEnd** | 会话终止时 | （无） | 否 | 清理、最终日志记录 |

### PreToolUse

在 Claude 创建工具参数后、真正处理前运行。可用于校验或修改工具输入。

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py"
          }
        ]
      }
    ]
  }
}
```

**常见 matcher：** `Task`、`Bash`、`Glob`、`Grep`、`Read`、`Edit`、`Write`、`WebFetch`、`WebSearch`

**输出控制：**
- `permissionDecision`: `"allow"`、`"deny"` 或 `"ask"`
- `permissionDecisionReason`: 决策说明
- `updatedInput`: 修改后的工具输入参数

### PostToolUse

在工具完成后立即运行。可用于验证、日志记录或向 Claude 返回上下文。

**配置：**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `"block"` 决策会向 Claude 反馈并阻断
- `additionalContext`: 追加给 Claude 的上下文

### UserPromptSubmit

在用户提交提示词、Claude 处理之前运行。

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py"
          }
        ]
      }
    ]
  }
}
```

**输出控制：**
- `decision`: `"block"` 表示阻止处理
- `reason`: 如果被阻止则说明原因
- `additionalContext`: 追加到提示词的上下文

### Stop 和 SubagentStop

在 Claude 完成回复（Stop）或子代理完成（SubagentStop）时运行。支持基于提示词的评估，用于智能检查任务是否完成。

**额外输入字段：** `Stop` 和 `SubagentStop` hooks 都会在 JSON 输入中收到 `last_assistant_message` 字段，其中包含 Claude 或子代理停止前的最后一条消息。它很适合用于评估任务完成情况。

**配置：**
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if Claude completed all requested tasks.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### SubagentStart

在子代理开始执行时运行。matcher 输入是 agent 类型名，因此 hooks 可以针对特定子代理类型。

**配置：**
```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "code-review",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/subagent-init.sh"
          }
        ]
      }
    ]
  }
}
```

### SessionStart

在会话开始或恢复时运行。可以持久化环境变量。

**Matcher：** `startup`、`resume`、`clear`、`compact`

**特殊功能：** 使用 `CLAUDE_ENV_FILE` 来持久化环境变量（在 `CwdChanged` 和 `FileChanged` hooks 中也可用）：

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=development' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```

### SessionEnd

在会话结束时运行，用于清理或最终日志记录。不能阻断终止。

**Reason 字段取值：**
- `clear` - 用户清空了会话
- `logout` - 用户退出登录
- `prompt_input_exit` - 用户通过 prompt 输入退出
- `other` - 其他原因

**配置：**
```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-cleanup.sh\""
          }
        ]
      }
    ]
  }
}
```

### Notification 事件

通知事件支持的 matcher：
- `permission_prompt` - 权限请求通知
- `idle_prompt` - 空闲状态通知
- `auth_success` - 认证成功
- `elicitation_dialog` - 向用户展示的对话框

## 组件作用域 Hook

可以在组件自己的 frontmatter 中把 hooks 附加到特定组件（skills、agents、commands）上：

**在 SKILL.md、agent.md 或 command.md 中：**

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/check.sh"
          once: true  # 每个会话只运行一次
---
```

**组件 hooks 支持的事件：** `PreToolUse`、`PostToolUse`、`Stop`

这样可以把 hooks 直接定义在使用它们的组件里，让相关代码更集中。

### 子代理 frontmatter 中的 Hooks

当在子代理的 frontmatter 中定义 `Stop` hook 时，它会自动转换成只作用于该子代理的 `SubagentStop` hook。这样可以确保 stop hook 只会在特定子代理完成时触发，而不会在主会话结束时触发。

```yaml
---
name: code-review-agent
description: Automated code review subagent
hooks:
  Stop:
    - hooks:
        - type: prompt
          prompt: "Verify the code review is thorough and complete."
  # 上面的 Stop hook 会自动转换为该子代理的 SubagentStop
---
```

## PermissionRequest 事件

使用自定义输出格式处理权限请求：

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow|deny",
      "updatedInput": {},
      "message": "自定义消息",
      "interrupt": false
    }
  }
}
```

## Hook 输入与输出

### JSON 输入（通过 stdin）

所有 hooks 都会通过 stdin 接收 JSON 输入：

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "content": "..."
  },
  "tool_use_id": "toolu_01ABC123...",
  "agent_id": "agent-abc123",
  "agent_type": "main",
  "worktree": "/path/to/worktree"
}
```

**常见字段：**

| 字段 | 说明 |
|----------|-------------|
| `session_id` | 唯一会话标识符 |
| `transcript_path` | 对话 transcript 文件路径 |
| `cwd` | 当前工作目录 |
| `hook_event_name` | 触发 hook 的事件名称 |
| `agent_id` | 运行该 hook 的 agent 标识 |
| `agent_type` | agent 类型（`"main"`、子代理类型名等） |
| `worktree` | 如果 agent 在 worktree 中运行，则为 git worktree 路径 |

### 退出码

| 退出码 | 含义 | 行为 |
|-----------|---------|----------|
| **0** | 成功 | 继续执行，解析 JSON stdout |
| **2** | 阻断错误 | 阻断操作，stderr 作为错误显示 |
| **其他** | 非阻断错误 | 继续执行，stderr 仅在 verbose 模式下显示 |

### JSON 输出（stdout，退出码 0）

```json
{
  "continue": true,
  "stopReason": "如果停止，可选消息",
  "suppressOutput": false,
  "systemMessage": "可选警告消息",
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow",
    "permissionDecisionReason": "File is in allowed directory",
    "updatedInput": {
      "file_path": "/modified/path.js"
    }
  }
}
```

## 环境变量

| 变量 | 可用范围 | 说明 |
|----------|-------------|-------------|
| `CLAUDE_PROJECT_DIR` | 所有 hooks | 项目根目录的绝对路径 |
| `CLAUDE_ENV_FILE` | SessionStart、CwdChanged、FileChanged | 用于持久化环境变量的文件路径 |
| `CLAUDE_CODE_REMOTE` | 所有 hooks | 如果运行在远程环境中则为 `"true"` |
| `${CLAUDE_PLUGIN_ROOT}` | 插件 hooks | 插件目录路径 |
| `${CLAUDE_PLUGIN_DATA}` | 插件 hooks | 插件数据目录路径 |
| `CLAUDE_CODE_SESSIONEND_HOOKS_TIMEOUT_MS` | SessionEnd hooks | 可配置的 SessionEnd hooks 超时时间（毫秒，覆盖默认值） |

## 基于提示词的 Hook

对于 `Stop` 和 `SubagentStop` 事件，你可以使用基于 LLM 的评估：

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if all tasks are complete. Return your decision.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**LLM 响应 Schema：**
```json
{
  "decision": "approve",
  "reason": "All tasks completed successfully",
  "continue": false,
  "stopReason": "Task complete"
}
```

## 示例

### 示例 1：Bash 命令校验器（PreToolUse）

**文件：** `.claude/hooks/validate-bash.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"\brm\s+-rf\s+/", "Blocking dangerous rm -rf / command"),
    (r"\bsudo\s+rm", "Blocking sudo rm command"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name != "Bash":
        sys.exit(0)

    command = input_data.get("tool_input", {}).get("command", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, command):
            print(message, file=sys.stderr)
            sys.exit(2)  # Exit 2 = blocking error

    sys.exit(0)

if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\""
          }
        ]
      }
    ]
  }
}
```

### 示例 2：安全扫描器（PostToolUse）

**文件：** `.claude/hooks/security-scan.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

SECRET_PATTERNS = [
    (r"password\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded password"),
    (r"api[_-]?key\s*=\s*['\"][^'\"]+['\"]", "Potential hardcoded API key"),
]

def main():
    input_data = json.load(sys.stdin)

    tool_name = input_data.get("tool_name", "")
    if tool_name not in ["Write", "Edit"]:
        sys.exit(0)

    tool_input = input_data.get("tool_input", {})
    content = tool_input.get("content", "") or tool_input.get("new_string", "")
    file_path = tool_input.get("file_path", "")

    warnings = []
    for pattern, message in SECRET_PATTERNS:
        if re.search(pattern, content, re.IGNORECASE):
            warnings.append(message)

    if warnings:
        output = {
            "hookSpecificOutput": {
                "hookEventName": "PostToolUse",
                "additionalContext": f"Security warnings for {file_path}: " + "; ".join(warnings)
            }
        }
        print(json.dumps(output))

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 3：自动格式化代码（PostToolUse）

**文件：** `.claude/hooks/format-code.sh`

```bash
#!/bin/bash

# 读取 stdin 中的 JSON
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_name', ''))")
FILE_PATH=$(echo "$INPUT" | python3 -c "import sys, json; print(json.load(sys.stdin).get('tool_input', {}).get('file_path', ''))")

if [ "$TOOL_NAME" != "Write" ] && [ "$TOOL_NAME" != "Edit" ]; then
    exit 0
fi

# 根据文件扩展名格式化
case "$FILE_PATH" in
    *.js|*.jsx|*.ts|*.tsx|*.json)
        command -v prettier &>/dev/null && prettier --write "$FILE_PATH" 2>/dev/null
        ;;
    *.py)
        command -v black &>/dev/null && black "$FILE_PATH" 2>/dev/null
        ;;
    *.go)
        command -v gofmt &>/dev/null && gofmt -w "$FILE_PATH" 2>/dev/null
        ;;
esac

exit 0
```

### 示例 4：Prompt 校验器（UserPromptSubmit）

**文件：** `.claude/hooks/validate-prompt.py`

```python
#!/usr/bin/env python3
import json
import sys
import re

BLOCKED_PATTERNS = [
    (r"delete\s+(all\s+)?database", "Dangerous: database deletion"),
    (r"rm\s+-rf\s+/", "Dangerous: root deletion"),
]

def main():
    input_data = json.load(sys.stdin)
    prompt = input_data.get("user_prompt", "") or input_data.get("prompt", "")

    for pattern, message in BLOCKED_PATTERNS:
        if re.search(pattern, prompt, re.IGNORECASE):
            output = {
                "decision": "block",
                "reason": f"Blocked: {message}"
            }
            print(json.dumps(output))
            sys.exit(0)

    sys.exit(0)

if __name__ == "__main__":
    main()
```

### 示例 5：智能 Stop Hook（基于 Prompt）

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review if Claude completed all requested tasks. Check: 1) Were all files created/modified? 2) Were there unresolved errors? If incomplete, explain what's missing.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### 示例 6：上下文使用跟踪器（Hook 配对）

结合使用 `UserPromptSubmit`（pre-message）和 `Stop`（post-response）hooks 来跟踪每个请求的 token 消耗。

**文件：** `.claude/hooks/context-tracker.py`

```python
#!/usr/bin/env python3
"""
Context Usage Tracker - Tracks token consumption per request.

Uses UserPromptSubmit as "pre-message" hook and Stop as "post-response" hook
to calculate the delta in token usage for each request.

Token Counting Methods:
1. Character estimation (default): ~4 chars per token, no dependencies
2. tiktoken (optional): More accurate (~90-95%), requires: pip install tiktoken
"""
import json
import os
import sys
import tempfile

# 配置
CONTEXT_LIMIT = 128000  # Claude 的上下文窗口（根据你的模型调整）
USE_TIKTOKEN = False    # 如果安装了 tiktoken，可设为 True 以提高准确性


def get_state_file(session_id: str) -> str:
    """获取临时文件路径，用于保存 pre-message token 数，按 session 隔离。"""
    return os.path.join(tempfile.gettempdir(), f"claude-context-{session_id}.json")


def count_tokens(text: str) -> int:
    """
    统计文本中的 token 数。

    如果可用，使用 tiktoken 和 p50k_base 编码（约 90-95% 准确率），
    否则回退到字符估算（约 80-90% 准确率）。
    """
    if USE_TIKTOKEN:
        try:
            import tiktoken
            enc = tiktoken.get_encoding("p50k_base")
            return len(enc.encode(text))
        except ImportError:
            pass  # 回退到估算方式

    # 基于字符的估算：英文大约 4 个字符对应 1 个 token
    return len(text) // 4


def read_transcript(transcript_path: str) -> str:
    """读取并拼接 transcript 文件中的全部内容。"""
    if not transcript_path or not os.path.exists(transcript_path):
        return ""

    content = []
    with open(transcript_path, "r") as f:
        for line in f:
            try:
                entry = json.loads(line.strip())
                # 从不同消息格式中提取文本内容
                if "message" in entry:
                    msg = entry["message"]
                    if isinstance(msg.get("content"), str):
                        content.append(msg["content"])
                    elif isinstance(msg.get("content"), list):
                        for block in msg["content"]:
                            if isinstance(block, dict) and block.get("type") == "text":
                                content.append(block.get("text", ""))
            except json.JSONDecodeError:
                continue

    return "\n".join(content)


def handle_user_prompt_submit(data: dict) -> None:
    """pre-message hook：在请求前保存当前 token 数。"""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # 保存到临时文件，供之后比较
    state_file = get_state_file(session_id)
    with open(state_file, "w") as f:
        json.dump({"pre_tokens": current_tokens}, f)


def handle_stop(data: dict) -> None:
    """post-response hook：计算并报告 token 增量。"""
    session_id = data.get("session_id", "unknown")
    transcript_path = data.get("transcript_path", "")

    transcript_content = read_transcript(transcript_path)
    current_tokens = count_tokens(transcript_content)

    # 读取 pre-message 的 token 数
    state_file = get_state_file(session_id)
    pre_tokens = 0
    if os.path.exists(state_file):
        try:
            with open(state_file, "r") as f:
                state = json.load(f)
                pre_tokens = state.get("pre_tokens", 0)
        except (json.JSONDecodeError, IOError):
            pass

    # 计算增量
    delta_tokens = current_tokens - pre_tokens
    remaining = CONTEXT_LIMIT - current_tokens
    percentage = (current_tokens / CONTEXT_LIMIT) * 100

    # 报告使用情况
    method = "tiktoken" if USE_TIKTOKEN else "estimated"
    print(f"Context ({method}): ~{current_tokens:,} tokens ({percentage:.1f}% used, ~{remaining:,} remaining)", file=sys.stderr)
    if delta_tokens > 0:
        print(f"This request: ~{delta_tokens:,} tokens", file=sys.stderr)


def main():
    data = json.load(sys.stdin)
    event = data.get("hook_event_name", "")

    if event == "UserPromptSubmit":
        handle_user_prompt_submit(data)
    elif event == "Stop":
        handle_stop(data)

    sys.exit(0)


if __name__ == "__main__":
    main()
```

**配置：**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/context-tracker.py\""
          }
        ]
      }
    ]
  }
}
```

**工作方式：**
1. `UserPromptSubmit` 在你的 prompt 被处理前触发 - 保存当前 token 数
2. `Stop` 在 Claude 响应后触发 - 计算差值并报告使用量
3. 每个 session 通过临时文件名中的 `session_id` 进行隔离

**Token 统计方法：**

| 方法 | 准确率 | 依赖 | 速度 |
|--------|----------|--------------|-------|
| 字符估算 | ~80-90% | 无 | <1ms |
| tiktoken（p50k_base） | ~90-95% | `pip install tiktoken` | <10ms |

> **注意：** Anthropic 还没有发布官方离线 tokenizer。两种方法都只是近似值。transcript 包含用户 prompt、Claude 的回复和工具输出，但不包含系统 prompt 或内部上下文。

### 示例 7：种子化自动模式权限（一次性设置脚本）

一个一次性设置脚本，用于向 `~/.claude/settings.json` 写入约 67 条安全权限规则，这些规则等价于 Claude Code 的自动模式基线。脚本不会使用 hook，也不会记住未来的选择。运行一次即可，重复运行也安全（会跳过已存在的规则）。

**文件：** `09-advanced-features/setup-auto-mode-permissions.py`

```bash
# 预览将要添加的内容
python3 09-advanced-features/setup-auto-mode-permissions.py --dry-run

# 应用
python3 09-advanced-features/setup-auto-mode-permissions.py
```

**会添加什么：**

| 类别 | 示例 |
|----------|---------|
| 内置工具 | `Read(*)`、`Edit(*)`、`Write(*)`、`Glob(*)`、`Grep(*)`、`Agent(*)`、`WebSearch(*)` |
| Git 读取 | `Bash(git status:*)`、`Bash(git log:*)`、`Bash(git diff:*)` |
| Git 写入（本地） | `Bash(git add:*)`、`Bash(git commit:*)`、`Bash(git checkout:*)` |
| 包管理器 | `Bash(npm install:*)`、`Bash(pip install:*)`、`Bash(cargo build:*)` |
| 构建与测试 | `Bash(make:*)`、`Bash(pytest:*)`、`Bash(go test:*)` |
| 常用 shell | `Bash(ls:*)`、`Bash(cat:*)`、`Bash(find:*)`、`Bash(cp:*)`、`Bash(mv:*)` |
| GitHub CLI | `Bash(gh pr view:*)`、`Bash(gh pr create:*)`、`Bash(gh issue list:*)` |

**脚本刻意排除的内容**（不会被添加）：
- `rm -rf`、`sudo`、force push、`git reset --hard`
- `DROP TABLE`、`kubectl delete`、`terraform destroy`
- `npm publish`、`curl | bash`、生产环境部署

## 插件 Hook

插件可以在它们的 `hooks/hooks.json` 文件中包含 hooks：

**文件：** `plugins/hooks/hooks.json`

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

**插件 hooks 中的环境变量：**
- `${CLAUDE_PLUGIN_ROOT}` - 插件目录路径
- `${CLAUDE_PLUGIN_DATA}` - 插件数据目录路径

这允许插件包含自定义的校验和自动化 hooks。

## MCP 工具 Hook

MCP 工具遵循 `mcp__<server>__<tool>` 模式：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "command",
            "command": "echo '{\"systemMessage\": \"Memory operation logged\"}'"
          }
        ]
      }
    ]
  }
}
```

## 安全注意事项

### 免责声明

**使用风险自负**：Hooks 会执行任意 shell 命令。你需要对以下内容承担全部责任：
- 你配置的命令
- 文件访问/修改权限
- 可能的数据丢失或系统损坏
- 在生产使用前先在安全环境中测试 hooks

### 安全说明

- **需要 workspace trust：** `statusLine` 和 `fileSuggestion` hook 输出命令现在在生效前需要 workspace trust 批准。
- **HTTP hooks 与环境变量：** HTTP hooks 需要显式的 `allowedEnvVars` 列表，才能在 URL 中使用环境变量插值。这可以防止敏感环境变量意外泄露到远程端点。
- **受管设置层级：** `disableAllHooks` 设置现在会遵守受管设置层级，这意味着组织级设置可以强制禁用 hooks，个人用户无法覆盖。

### 最佳实践

| 要做 | 不要做 |
|-----|-------|
| 校验并清理所有输入 | 盲目信任输入数据 |
| 给 shell 变量加引号：`"$VAR"` | 使用未加引号的：`$VAR` |
| 阻止路径穿越（`..`） | 允许任意路径 |
| 使用 `$CLAUDE_PROJECT_DIR` 的绝对路径 | 硬编码路径 |
| 跳过敏感文件（`.env`、`.git/`、keys） | 处理所有文件 |
| 先在隔离环境中测试 hooks | 部署未测试的 hooks |
| 为 HTTP hooks 使用显式的 `allowedEnvVars` | 把所有 env vars 暴露给 webhooks |

## 调试

### 启用 Debug 模式

使用 debug 标志启动 Claude，以获得详细的 hook 日志：

```bash
claude --debug
```

### Verbose 模式

在 Claude Code 中按 `Ctrl+O` 启用 verbose 模式，查看 hook 执行进度。

### 独立测试 Hooks

```bash
# 使用样例 JSON 输入测试
echo '{"tool_name": "Bash", "tool_input": {"command": "ls -la"}}' | python3 .claude/hooks/validate-bash.py

# 检查退出码
echo $?
```

## 完整配置示例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/format-code.sh\"",
            "timeout": 30
          },
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/security-scan.py\"",
            "timeout": 10
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"$CLAUDE_PROJECT_DIR/.claude/hooks/validate-prompt.py\""
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR/.claude/hooks/session-init.sh\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Verify all tasks are complete before stopping.",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

## Hook 执行详情

| 方面 | 行为 |
|--------|----------|
| **Timeout** | 默认 60 秒，每个 command 可单独配置 |
| **并行化** | 所有匹配的 hooks 并行运行 |
| **去重** | 相同的 hook command 会去重 |
| **环境** | 在当前目录下运行，使用 Claude Code 的环境 |

## 故障排查

### Hook 没有执行
- 验证 JSON 配置语法正确
- 检查 matcher 模式是否匹配工具名
- 确保脚本存在且可执行：`chmod +x script.sh`
- 运行 `claude --debug` 查看 hook 执行日志
- 确认 hook 是从 stdin 读取 JSON，而不是从命令参数读取

### Hook 发生意外阻断
- 用样例 JSON 测试 hook：`echo '{"tool_name": "Write", ...}' | ./hook.py`
- 检查退出码：允许应为 0，阻断应为 2
- 检查 stderr 输出（退出码 2 时会显示）

### JSON 解析错误
- 始终从 stdin 读取，不要从命令参数读取
- 使用正确的 JSON 解析，而不是字符串拼接
- 对缺失字段要优雅处理

## 安装

### 第 1 步：创建 Hooks 目录
```bash
mkdir -p ~/.claude/hooks
```

### 第 2 步：复制示例 Hooks
```bash
cp 06-hooks/*.sh ~/.claude/hooks/
chmod +x ~/.claude/hooks/*.sh
```

### 第 3 步：在设置中配置
编辑 `~/.claude/settings.json` 或 `.claude/settings.json`，加入上面展示的 hook 配置。

## 相关概念

- **[Checkpoints and Rewind](../08-checkpoints/)** - 保存并恢复对话状态
- **[Slash Commands](../01-slash-commands/)** - 创建自定义斜杠命令
- **[Skills](../03-skills/)** - 可复用的自主能力
- **[Subagents](../04-subagents/)** - 委派式任务执行
- **[Plugins](../07-plugins/)** - 捆绑式扩展包
- **[Advanced Features](../09-advanced-features/)** - 探索 Claude Code 的高级能力

## 额外资源

- **[官方 Hooks 文档](https://code.claude.com/docs/en/hooks)** - 完整 hooks 参考
- **[CLI 参考](https://code.claude.com/docs/en/cli-reference)** - 命令行接口文档
- **[Memory 指南](../02-memory/)** - 持久上下文配置
