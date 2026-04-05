# [HTTP 方法] /api/v1/[endpoint]

## 说明
简要说明该端点的职责、输入输出以及设计原因。

## 认证
所需的认证方式，例如 Bearer token。

## 参数

### 路径参数
| 名称 | 类型 | 必填 | 描述 |
|------|------|------|------|
| id | string | 是 | 资源 ID |

### 查询参数
| 名称 | 类型 | 必填 | 描述 |
|------|------|------|------|
| page | integer | 否 | 页码（默认：1） |
| limit | integer | 否 | 每页条数（默认：20） |

### 请求体
```json
{
  "field": "value"
}
```

## 响应

### 200 OK
```json
{
  "success": true,
  "data": {
    "id": "123",
    "name": "示例"
  }
}
```

### 400 Bad Request
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "无效输入"
  }
}
```

### 404 Not Found
```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "资源未找到"
  }
}
```

## 示例

### cURL
```bash
curl -X GET "https://api.example.com/api/v1/endpoint" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json"
```

### JavaScript
```javascript
const response = await fetch('/api/v1/endpoint', {
  headers: {
    'Authorization': 'Bearer token',
    'Content-Type': 'application/json'
  }
});
const data = await response.json();
```

### Python
```python
import requests

response = requests.get(
    'https://api.example.com/api/v1/endpoint',
    headers={'Authorization': 'Bearer token'}
)
data = response.json()
```

## 速率限制
- 已认证用户每小时 1000 次请求
- 公开端点每小时 100 次请求

## 注意事项
- 说明幂等性、分页规则或排序规则
- 提醒调用方注意字段兼容性和版本差异
- 如有异步行为，请说明任务完成后的回调或轮询方式

## 相关端点
- [GET /api/v1/related](#)
- [POST /api/v1/related](#)
