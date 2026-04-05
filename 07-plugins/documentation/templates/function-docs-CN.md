# 函数：`functionName`

## 说明
简要说明该函数的作用、适用场景，以及它解决的核心问题。

## 签名
```typescript
function functionName(param1: Type1, param2: Type2): ReturnType
```

## 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| param1 | Type1 | 是 | param1 的说明 |
| param2 | Type2 | 否 | param2 的说明 |

## 返回值
**类型**：`ReturnType`

说明返回值的语义、边界条件，以及调用方在什么情况下应进一步处理它。

## 抛出异常
- `Error`：在输入无效时抛出
- `TypeError`：在传入错误类型时抛出

## 示例

### 基本用法
```typescript
const result = functionName('value1', 'value2');
console.log(result);
```

### 高级用法
```typescript
const result = functionName(
  complexParam1,
  { option: true }
);
```

## 注意事项
- 补充说明或警告
- 性能注意事项
- 最佳实践
- 兼容性或版本差异
- 常见误用与避免方式

## 另请参阅
- [相关函数](#)
- [API 文档](#)
