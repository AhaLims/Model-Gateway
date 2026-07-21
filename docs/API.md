# ModelPort API 接口文档

## 目录
- [客户端 API](#客户端-api)
- [运维相关 API](#运维相关-api)
- [管理后台 API](#管理后台-api)

---

## 客户端 API

### 1. 获取可用模型列表

| 项目 | 内容 |
|------|------|
| **HTTP 方法** | `GET` |
| **路由路径** | `/v1/models` |
| **功能描述** | 获取所有可用的模型列表，包含模型 ID、显示名称等信息 |
| **认证方式** | API Key (`Authorization: Bearer <token>` 或 `x-api-key: <token>`) |
| **响应示例** | 见下方代码块 |

### 2. 发送消息请求

| 项目 | 内容 |
|------|------|
| **HTTP 方法** | `POST` |
| **路由路径** | `/v1/messages` |
| **功能描述** | 向 LLM 发送消息请求，支持流式和非流式响应 |
| **认证方式** | API Key (`Authorization: Bearer <token>` 或 `x-api-key: <token>`) |
| **请求 Body** | ```json
{
  "model": "deepseek-v4-flash",
  "max_tokens": 1024,
  "messages": [
    {"role": "user", "content": "Hello"}
  ],
  "system": [{"type": "text", "text": "You are a helpful assistant"}],
  "stream": false
}
``` |
| **请求参数** | - `model` (string, required): 模型 ID<br>- `max_tokens` (number, optional): 最大输出 token 数<br>- `messages` (array, required): 消息数组<br>- `system` (array, optional): 系统提示<br>- `stream` (boolean, optional): 是否流式输出 |

---

## 运维相关 API

### 3. 存活检查

| 项目 | 内容 |
|------|------|
| **HTTP 方法** | `GET` |
| **路由路径** | `/livez` |
| **功能描述** | 简单的存活检查，无需认证 |
| **响应示例** | `{"status": "ok", "service": "model-port"}` |

### 4. 就绪检查

| 项目 | 内容 |
|------|------|
| **HTTP 方法** | `GET` |
| **路由路径** | `/readyz` |
| **功能描述** | 检查服务是否就绪，需要认证 |

### 5. 健康检查

| 项目 | 内容 |
|------|------|
| **HTTP 方法** | `GET` |
| **路由路径** | `/health` |
| **功能描述** | 获取服务健康状态，认证后返回详细信息 |
| **响应示例** | ```json
{
  "status": "ok",
  "service": "model-port",
  "providers": ["deepseek"],
  "providerHealth": {
    "deepseek": {"status": "ok", "latency": 120}
  }
}
``` |

### 6. Prometheus 指标

| 项目 | 内容 |
|------|------|
| **HTTP 方法** | `GET` |
| **路由路径** | `/metrics` |
| **功能描述** | 获取 Prometheus 格式的监控指标 |
| **认证方式** | API Key |

---

## 管理后台 API

### 认证相关

| 项目 | 内容 |
|------|------|
| **登录** | `POST /admin/auth/login` - Body: `{"username": "admin", "password": "xxx"}` |
| **登出** | `POST /admin/auth/logout` |
| **当前用户** | `GET /admin/auth/me` - 获取当前登录的管理员信息 |

### 仪表盘

| 项目 | 内容 |
|------|------|
| **HTTP 方法** | `GET` |
| **路由路径** | `/admin/dashboard` |
| **查询参数** | - `range`: 时间范围 (1d/3d/7d/custom)<br>- `from`: 开始时间 (毫秒时间戳)<br>- `to`: 结束时间 (毫秒时间戳) |

---

## 供应商管理 API

### 供应商 CRUD

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/providers` | 获取所有供应商列表 |
| `POST` | `/admin/providers` | 创建供应商 |
| `PUT` | `/admin/providers/:provider_id` | 更新供应商 |
| `DELETE` | `/admin/providers/:provider_id` | 删除供应商 |

**创建/更新供应商 Body 示例**:
```json
{
  "id": "deepseek",
  "displayName": "DeepSeek",
  "protocol": "anthropic",
  "baseUrl": "https://api.deepseek.com/anthropic",
  "apiKeyEnv": "DEEPSEEK_ANTHROPIC_AUTH_TOKEN",
  "defaultModel": "deepseek-v4-flash",
  "models": ["deepseek-v4-flash", "deepseek-v4-pro"],
  "status": "active"
}
```

### 模型管理

| 方法 | 路由 | 说明 |
|------|------|------|
| `POST` | `/admin/providers/:provider_id/models/discover` | 发现供应商模型 |
| `POST` | `/admin/providers/:provider_id/models` | 添加模型 |
| `PUT` | `/admin/providers/:provider_id/models/:model_id` | 更新模型 |
| `DELETE` | `/admin/providers/:provider_id/models/:model_id` | 删除模型 |

### 凭证管理

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/providers/:provider_id/credentials` | 获取凭证列表 |
| `POST` | `/admin/providers/:provider_id/credentials` | 添加凭证 |
| `PUT` | `/admin/providers/:provider_id/credentials/:credential_id` | 更新凭证 |
| `DELETE` | `/admin/providers/:provider_id/credentials/:credential_id` | 删除凭证 |
| `POST` | `/admin/providers/:provider_id/credentials/:credential_id/select` | 选择活跃凭证 |
| `PUT` | `/admin/providers/:provider_id/credential-pool` | 设置凭证池模式 |

---

## 别名管理 API

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/aliases` | 获取所有别名 |
| `POST` | `/admin/aliases` | 创建别名 - Body: `{"alias": "fast", "target": "deepseek:deepseek-v4-flash"}` |
| `DELETE` | `/admin/aliases/:alias` | 删除别名 |

---

## 设置管理 API

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/settings` | 获取系统设置 |
| `PUT` | `/admin/settings` | 更新系统设置 |
| `POST` | `/admin/settings/reload-config` | 重新加载配置 |
| `POST` | `/admin/settings/test-provider` | 测试供应商连接 |

---

## 用户管理 API

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/users` | 获取用户列表 |
| `POST` | `/admin/users` | 创建用户 |
| `PUT` | `/admin/users/:user_id` | 更新用户 |
| `DELETE` | `/admin/users/:user_id` | 删除用户 |

**创建用户 Body 示例**:
```json
{
  "username": "user1",
  "email": "user1@example.com",
  "password": "password",
  "role": "user",
  "status": "active"
}
```

---

## API Key 管理 API

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/api-keys` | 获取所有 API Key |
| `POST` | `/admin/api-keys` | 创建 API Key |
| `GET` | `/admin/users/:user_id/api-keys` | 获取用户的 API Keys |
| `PUT` | `/admin/api-keys/:key_id` | 更新 API Key |
| `DELETE` | `/admin/api-keys/:key_id` | 吊销/删除 API Key |

**创建 API Key Body 示例**:
```json
{
  "userId": "user-id",
  "name": "My API Key",
  "expiresAt": "2025-12-31T23:59:59Z"
}
```

---

## 配额管理 API

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/quotas` | 获取配额列表 |
| `POST` | `/admin/quotas` | 创建配额 |
| `PUT` | `/admin/quotas/:quota_id` | 更新配额 |
| `DELETE` | `/admin/quotas/:quota_id` | 删除配额 |

---

## 日志与监控 API

| 方法 | 路由 | 说明 |
|------|------|------|
| `GET` | `/admin/logs` | 获取请求日志 |
| `GET` | `/admin/latency` | 获取延迟统计 |
| `GET` | `/admin/audit` | 获取审计日志 |
| `GET` | `/admin/backup` | 获取备份信息 |

---

## 认证说明

### 认证方式
- **API Key 认证**: 使用 `Authorization: Bearer <token>` 或 `x-api-key: <token>` 头
- **Session Cookie**: 管理员后台使用 `modelport_admin_session` cookie
- **CSRF 保护**: 使用 `x-modelport-csrf` 头

### 角色权限
- `admin`: 管理员，可访问所有 API
- `user`: 普通用户，限制访问

### 环境变量中的 Token
在 `.env` 文件中配置:
- `MODELPORT_AUTH_TOKEN` - 路由鉴权 token
- `ANTHROPIC_AUTH_TOKEN` - Claude Code 使用的 token
