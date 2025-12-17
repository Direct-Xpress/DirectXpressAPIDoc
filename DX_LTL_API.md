# DX_LTL 获取访问令牌 API 文档

V1.0  （如有建议，请在 issues 提出）

---

# 目录 <a name="index"/>
* [一. API 接口规范](#api_standard_index)
  * [接口响应规范](#api_resp_index)
  * [接口签名规范](#api_sign_index)
  * [接口请求示例](#api_demo_index)
* [二. API 说明](#api_common_index)
  * [获取访问令牌](#api_get_token)
  * [应用场景](#yycj_index)

# 一. API 接口规范 <a name="api_standard_index"/>

## 接口响应规范 <a name="api_resp_index"/>
HTTP 接口遵循统一返回格式：
```
{
  "code": int,        // 必选，返回码
  "message": "",      // 可选，返回消息，如 ok、error
  "data": {}          // 可选，返回内容
}
```
返回码参考（通常与 HTTP 状态码一致）：

code | value
--- | ---
200 | 成功
4xx | 客户端错误
5xx | 服务器端错误
自定义 | 业务自定义

## 接口签名规范 <a name="api_sign_index"/>
- 如需加密传输，可使用 AES 对请求体加密后再 Base64（base64UrlEncode）编码，将签名值放入参数（如 `sign=xxx`）。
- 示例参数：加密模式 AES-128-CBC（推荐）或 AES-128-ECB；密钥示例 `123456789`；填充 PKCS7。
- 仅在服务器要求签名时启用；本接口默认使用 HTTPS 明文 JSON。

## 接口请求示例 <a name="api_demo_index"/>
见下方接口示例。

# 二. API 说明 <a name="api_common_index"/>

## 获取访问令牌 <a name="api_get_token"/>
### 应用场景 <a name="yycj_index"/>
用于通过 `app_key` 与 `app_secret` 交换访问令牌。

描述 | 内容
--- | ---
接口功能 | 获取访问令牌
请求协议 | HTTPS
请求方法 | POST
请求格式 | `application/json`
请求 URL | `https://api.bz/ltl/v1/auth/get_access_token`
请求头 | `Content-Type: application/json`；`X-App-Language: zh/en/es`
备注 | `app_key`/`app_secret` 为敏感信息，务必妥善保管
响应格式 | JSON

请求参数：

参数 | 描述 | 必填 | 类型 | 规则
--- | --- | --- | --- | ---
app_key | DX 提供的 key | 是 | string | 正则建议 `^[A-Za-z0-9]{1,100}$`
app_secret | DX 提供的密钥 | 是 | string | 正则建议 `^[A-Za-z0-9]{1,200}$`

请求头参数：

参数名 | 示例值 | 必填 | 描述
--- | --- | --- | ---
Content-Type | application/json | 是 | 请求体 JSON
X-App-Language | en | 是 | 返回语言（`zh`、`en`、`es`）

请求示例：
```
POST https://api.bz/ltl/v1/auth/get_access_token
Content-Type: application/json
X-App-Language: en

{
  "app_key": "your_app_key",
  "app_secret": "your_app_secret"
}
```

响应示例（200）：
```
{
  "code": 0,
  "message": "Success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 86400
  }
}
```

前端校验建议：
- `app_key`：长度 1–100，字母数字。
- `app_secret`：长度 1–200，字母数字。
- 请求头：`Content-Type` 必须为 `application/json`，`X-App-Language` 必须为 `zh|en|es`。

# Changelog
- v1.0：获取访问令牌接口文档。

