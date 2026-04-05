---
description: 从源代码生成完整的 API 文档
---

# API 文档生成器

通过以下步骤生成 API 文档：

1. 扫描 `/src/api/` 下的所有文件
2. 提取函数签名和 JSDoc 注释
3. 按端点 / 模块进行组织
4. 创建带示例的 Markdown
5. 包含请求 / 响应结构
6. 补充错误文档

输出格式：
- 生成到 `/docs/api.md` 的 Markdown 文件
- 为所有端点包含 curl 示例
- 添加 TypeScript 类型
- 保持文档结构清晰，便于团队后续维护和发布
