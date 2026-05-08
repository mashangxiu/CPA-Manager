# Kiro 额度查询 API 前端对接指南

## API 端点

```
POST /v0/management/api-call
```

---

## 请求格式

### 请求头

```http
POST /v0/management/api-call HTTP/1.1
Authorization: Bearer <MANAGEMENT_KEY>
Content-Type: application/json
```

### 请求体

```json
{
  "auth_index": "<KIRO_AUTH_INDEX>",
  "method": "POST",
  "url": "https://codewhisperer.us-east-1.amazonaws.com",
  "header": {
    "Content-Type": "application/x-amz-json-1.0",
    "x-amz-target": "AmazonCodeWhispererService.GetUsageLimits",
    "Authorization": "Bearer $TOKEN$"
  },
  "data": "{\"origin\": \"AI_EDITOR\", \"resourceType\": \"AGENTIC_REQUEST\"}"
}
```

### 参数说明

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `auth_index` | string | 是 | Kiro 凭证索引，从 `/v0/management/auth-files` 获取 |
| `method` | string | 是 | 固定为 `POST` |
| `url` | string | 是 | AWS CodeWhisperer 端点 |
| `header` | object | 是 | 请求头，`$TOKEN$` 会自动替换为实际 Token |
| `data` | string | 是 | JSON 字符串格式的请求体 |

---

## 响应格式

### 成功响应

```json
{
  "status_code": 200,
  "header": { ... },
  "body": "<JSON 字符串，需要再次 JSON.parse>"
}
```

### body 解析后的结构（成功时）

```json
{
  "daysUntilReset": 0,
  "nextDateReset": 1772323200.0,
  "subscriptionInfo": {
    "subscriptionTitle": "KIRO FREE",
    "type": "Q_DEVELOPER_STANDALONE_FREE",
    "overageCapability": "OVERAGE_INCAPABLE",
    "upgradeCapability": "UPGRADE_CAPABLE"
  },
  "usageBreakdownList": [
    {
      "resourceType": "CREDIT",
      "usageLimitWithPrecision": 50.0,
      "currentUsageWithPrecision": 10.5,
      "freeTrialInfo": {
        "freeTrialStatus": "ACTIVE",
        "usageLimitWithPrecision": 500.0,
        "currentUsageWithPrecision": 21.48,
        "freeTrialExpiry": 1772553426.991
      }
    }
  ],
  "userInfo": {
    "userId": "d-xxx.xxx-xxx"
  }
}
```

### body 解析后的结构（错误时，如 403）

```json
{
  "__type": "com.amazon.aws.codewhisperer#AccessDeniedException",
  "message": "Your User ID (d-xxx) temporarily is suspended...",
  "reason": "TEMPORARILY_SUSPENDED"
}
```

---

## 字段说明

### 顶层字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `nextDateReset` | number | 额度重置时间戳（秒） |
| `subscriptionInfo` | object | 订阅信息 |
| `usageBreakdownList` | array | 额度使用明细列表 |
| `userInfo` | object | 用户信息 |

> 注：`daysUntilReset` 字段会返回但无需解析，重置时间以 `nextDateReset` 为准。

### subscriptionInfo

| 字段 | 可能的值 | 说明 |
|------|----------|------|
| `subscriptionTitle` | `KIRO FREE`, `KIRO POWER` | 订阅名称 |

> 注：其他字段（`type`, `overageCapability`, `upgradeCapability`）会返回但无需解析。

### usageBreakdownList[0]

| 字段 | 说明 |
|------|------|
| `resourceType` | 资源类型，通常为 `CREDIT` |
| `usageLimitWithPrecision` | 基础额度总量 |
| `currentUsageWithPrecision` | 基础额度已使用 |
| `nextDateReset` | 该资源类型的额度重置时间戳（秒） |
| `freeTrialInfo` | 免费试用信息（可选，仅 FREE 用户） |

### freeTrialInfo

| 字段 | 说明 |
|------|------|
| `freeTrialStatus` | 试用状态：`ACTIVE`, `EXPIRED` |
| `usageLimitWithPrecision` | 试用总额度 |
| `currentUsageWithPrecision` | 试用已使用 |
| `freeTrialExpiry` | 试用到期时间戳（秒） |

### 错误响应字段

| 字段 | 说明 |
|------|------|
| `__type` | 错误类型 |
| `message` | 错误详情（可能包含用户ID和支持链接） |
| `reason` | 错误原因代码 |

### 常见 reason 值

| reason | 说明 |
|--------|------|
| `TEMPORARILY_SUSPENDED` | 账户被临时封禁 |
| `SUBSCRIPTION_EXPIRED` | 订阅已过期 |
| `QUOTA_EXCEEDED` | 额度已耗尽 |

---

## 完整响应示例

### KIRO FREE 账户

```json
{
  "daysUntilReset": 5,
  "nextDateReset": 1772323200.0,
  "subscriptionInfo": {
    "subscriptionTitle": "KIRO FREE",
    "type": "Q_DEVELOPER_STANDALONE_FREE",
    "overageCapability": "OVERAGE_INCAPABLE",
    "upgradeCapability": "UPGRADE_CAPABLE"
  },
  "usageBreakdownList": [{
    "resourceType": "CREDIT",
    "usageLimitWithPrecision": 50.0,
    "currentUsageWithPrecision": 0.0,
    "nextDateReset": 1772323200.0,
    "freeTrialInfo": {
      "freeTrialStatus": "ACTIVE",
      "usageLimitWithPrecision": 500.0,
      "currentUsageWithPrecision": 21.48,
      "freeTrialExpiry": 1772553426.991
    }
  }],
  "userInfo": {
    "userId": "d-xxx.xxx"
  }
}
```

### KIRO POWER 账户

```json
{
  "daysUntilReset": 0,
  "nextDateReset": 1772323200.0,
  "subscriptionInfo": {
    "subscriptionTitle": "KIRO POWER",
    "type": "Q_DEVELOPER_STANDALONE_POWER",
    "overageCapability": "OVERAGE_CAPABLE",
    "upgradeCapability": "UPGRADE_INCAPABLE"
  },
  "usageBreakdownList": [{
    "resourceType": "CREDIT",
    "usageLimitWithPrecision": 10000.0,
    "currentUsageWithPrecision": 57.45,
    "nextDateReset": 1772323200.0,
    "overageRate": 0.04,
    "currency": "USD"
  }],
  "userInfo": {
    "userId": "d-xxx.xxx"
  }
}
```

### 账户封禁错误

```json
{
  "__type": "com.amazon.aws.codewhisperer#AccessDeniedException",
  "message": "Your User ID (d-9067642ac7.54d87418-b0c1-7076-faec-20dbb7e1acef) temporarily is suspended. We detected unusual user activity and locked it as a security precaution. To restore access, please contact our support team to verify your identity: https://support.aws.amazon.com/#/contacts/kiro",
  "reason": "TEMPORARILY_SUSPENDED"
}
```
