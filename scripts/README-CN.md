<picture>
  <source media="(prefers-color-scheme: dark)" srcset="../resources/logos/claude-howto-logo-dark.svg">
  <img alt="Claude How To" src="../resources/logos/claude-howto-logo.svg">
</picture>

# EPUB 构建脚本

使用这个脚本可将 Claude How To 的全部 Markdown 内容构建成一本 EPUB 电子书，适合离线阅读、移动端阅读，或导入电子书阅读器。

## 功能

- 按文件夹结构组织章节（例如 `01-slash-commands`、`02-memory` 等）
- 通过 Kroki.io API 将 Mermaid 图表渲染成 PNG 图片
- 使用异步并发抓取，所有图表会并行渲染
- 从项目 Logo 生成封面图
- 将内部 Markdown 链接转换为 EPUB 章节引用
- 使用严格错误模式，只要有任何图表无法渲染就直接失败

## 需求

- Python 3.10+
- [uv](https://github.com/astral-sh/uv)
- 用于 Mermaid 图表渲染的互联网连接

## 快速开始

最简单的方式是直接让 `uv` 处理所有依赖并运行脚本：

```bash
# 最简单的方式 - uv 会处理一切
uv run scripts/build_epub.py
```

## 开发环境

如果你要修改这个脚本本身，可以先创建虚拟环境，再安装开发依赖：

```bash
# 创建虚拟环境
uv venv

# 激活并安装依赖
source .venv/bin/activate
uv pip install -r requirements-dev.txt

# 运行测试
pytest scripts/tests/ -v

# 运行脚本
python scripts/build_epub.py
```

## 命令行选项

```text
usage: build_epub.py [-h] [--root ROOT] [--output OUTPUT] [--verbose]
                     [--timeout TIMEOUT] [--max-concurrent MAX_CONCURRENT]

options:
  -h, --help            show this help message and exit
  --root, -r ROOT       Root directory (default: repo root)
  --output, -o OUTPUT   Output path (default: claude-howto-guide.epub)
  --verbose, -v         Enable verbose logging
  --timeout TIMEOUT     API timeout in seconds (default: 30)
  --max-concurrent N    Max concurrent requests (default: 10)
```

## 示例

```bash
# 使用详细输出构建
uv run scripts/build_epub.py --verbose

# 指定自定义输出位置
uv run scripts/build_epub.py --output ~/Desktop/claude-guide.epub

# 限制并发请求数量（遇到速率限制时很有用）
uv run scripts/build_epub.py --max-concurrent 5
```

## 输出

脚本会在仓库根目录生成 `claude-howto-guide.epub`。

这个 EPUB 包含：
- 带项目 Logo 的封面
- 带嵌套章节的目录
- 转换为 EPUB 兼容 HTML 的全部 Markdown 内容
- 渲染为 PNG 图片的 Mermaid 图表

## 运行测试

```bash
# 使用虚拟环境
source .venv/bin/activate
pytest scripts/tests/ -v

# 或直接用 uv 运行
uv run --with pytest --with pytest-asyncio \
    --with ebooklib --with markdown --with beautifulsoup4 \
    --with httpx --with pillow --with tenacity \
    pytest scripts/tests/ -v
```

## 依赖

依赖通过 PEP 723 的内联脚本元数据进行管理：

| 包 | 用途 |
|---------|---------|
| `ebooklib` | 生成 EPUB |
| `markdown` | Markdown 转 HTML |
| `beautifulsoup4` | HTML 解析 |
| `httpx` | 异步 HTTP 客户端 |
| `pillow` | 生成封面图 |
| `tenacity` | 重试逻辑 |

## 故障排查

**构建因网络错误失败**：检查网络连接和 Kroki.io 状态。可以尝试使用 `--timeout 60`。

**触发限流**：使用 `--max-concurrent 3` 降低并发请求数。

**缺少 Logo**：如果找不到 `claude-howto-logo.png`，脚本会生成仅文字版封面。
