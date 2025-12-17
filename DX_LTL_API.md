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
  * [刷新 Access Token](#api_refresh_token)
  * [物流下单](#api_create_order)
  * [上传提货清单](#api_upload_delivery_note)

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

## 刷新 Access Token <a name="api_refresh_token"/>
用于刷新 Access Token，保持会话有效。

描述 | 内容
--- | ---
接口功能 | 刷新 Access Token
请求协议 | HTTPS
请求方法 | GET
请求格式 | 无请求体
请求 URL | `https://api.bz/ltl/v1/auth/refresh_access_token`
请求头 | `Authorization: Bearer <access_token>`；`X-App-Language: zh/en/es`
备注 | 无需请求体参数；需提供有效 Bearer Token
响应格式 | JSON

请求头参数：

参数名 | 示例值 | 必填 | 描述
--- | --- | --- | ---
X-App-Language | en | 是 | 返回语言（`zh`、`en`、`es`）
Authorization | Bearer your_access_token | 是 | 现有访问令牌

请求示例：
```
GET https://api.bz/ltl/v1/auth/refresh_access_token
X-App-Language: en
Authorization: Bearer your_access_token
```

响应示例（200）：
```
{
  "code": 0,
  "message": "Success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjozLCJpZGVudGl0eSI6InJuamhrdnQ4Y2RvdnRvdzAiLCJyb2xlIjoiY3VzdG9tZXIiLCJpc3MiOiJkeHByZXNzIiwiZXhwIjoxNzY0ODI3ODI2LCJuYmYiOjE3NjQ3NDE0MjYsImlhdCI6MTc2NDc0MTQyNn0.9g8CP3UFuOZinBB-CceG4wF0a5Z41lBO-Wd84RqotvU",
    "expires_in": 86172
  }
}
```

错误处理：
- 401：认证失败（token 无效或已过期）；前端应提示重新登录或重新获取 token。
- 404：接口/资源不存在。

前端校验建议：
- 请求头：`X-App-Language` 填写有效值，`Authorization` 携带有效 Bearer token。

## 物流下单 <a name="api_create_order"/>
用于创建物流订单（含收件/送件方与货物信息）。

描述 | 内容
--- | ---
接口功能 | 创建物流订单
请求协议 | HTTPS
请求方法 | POST
请求格式 | `application/json`
请求 URL | `https://api.bz/ltl/v1//logistics/create_order`
请求头 | `Content-Type: application/json`；`X-App-Language: zh/en/es`；`Authorization: Bearer <access_token>`
备注 | 确保必填字段齐全；确认实际认证方式（如 Bearer token）
响应格式 | JSON

请求体参数：

参数 | 描述 | 必填 | 类型 | 规则/说明
--- | --- | --- | --- | ---
logisticsList | 物流联系人数组 | 否 | array | 至少包含提货/送货地址
logisticsList[].contactName | 联系人姓名 | 是 | string | 1-100 字
logisticsList[].companyName | 公司名称 | 是 | string | 1-200 字
logisticsList[].email | 电子邮箱 | 是 | string | 建议正则 `^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$`
logisticsList[].mobile | 手机号码 | 是 | string | 示例为美国号，可用 `^1?\\d{10}$`
logisticsList[].addressType | 地址类型 | 是 | number | 1=提货，2=送货
logisticsList[].fullAddress | 完整地址 | 是 | string | 含街道、城市、州、省/邮编
logisticsList[].actionTime | 取/送时间 | 是 | number | UTC 时间戳
cargoList | 货物列表 | 是 | array | 至少 1 条
cargoList[].cargo_type | 货物类型 | 是 | number | LTL=1
cargoList[].length | 长 | 是 | number | 非负
cargoList[].width | 宽 | 是 | number | 非负
cargoList[].height | 高 | 是 | number | 非负
cargoList[].weight | 单件重量 | 是 | number | 非负
cargoList[].quantity | 数量 | 是 | number | 正整数，建议 `^[1-9]\\d*$`
cargoList[].description | 货物描述 | 是 | string | 可选文案
ref | 参考号 | 否 | string | 客户自定义单号

请求头参数：

参数名 | 示例值 | 必填 | 描述
--- | --- | --- | ---
Content-Type | application/json | 是 | 请求体 JSON
X-App-Language | en | 是 | 返回语言（`zh`、`en`、`es`）
Authorization | Bearer your_access_token | 是 | 访问令牌

请求示例：
```
POST https://api.bz/ltl/v1//logistics/create_order
Content-Type: application/json
X-App-Language: en
Authorization: Bearer your_access_token

{
  "logisticsList": [
    {
      "contactName": "DX test",
      "companyName": "The Home Depot",
      "email": "test@o2o.us",
      "mobile": "9493356666",
      "addressType": 1,
      "fullAddress": "6140 Hamner Ave, Mira Loma, CA 91752",
      "actionTime": 1764737842
    },
    {
      "contactName": "Little",
      "companyName": "The Home Depot",
      "email": "test2@o2o.us",
      "mobile": "9493356666",
      "addressType": 2,
      "fullAddress": "14549 Ramona Ave, Chino, CA 91710",
      "actionTime": 1764719842
    }
  ],
  "cargoList": [
    {
      "cargo_type": 1,
      "length": 40,
      "width": 40,
      "height": 80,
      "weight": 200,
      "quantity": 1,
      "description": "furniture"
    }
  ],
  "ref": "your-ref-123"
}
```

响应示例（200）：
```
{
  "code": 0,
  "message": "Success",
  "data": {
    "orderId": 1100000000000006
  }
}
```

错误处理：
- 400：参数缺失或格式错误。
- 401：认证失败，需检查 Authorization。
- 404：接口/资源不存在。
- 500：服务器异常，建议稍后重试并记录日志。

前端校验建议：
- 邮箱：`^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$`
- 手机（示例为美区）：`^1?\\d{10}$`
- 地址类型：限定 1/2。
- 尺寸/重量：非负数字；数量：正整数。
- Header：确保 `Content-Type: application/json`、`X-App-Language`、`Authorization: Bearer <token>`。

## 上传提货清单 <a name="api_upload_delivery_note"/>
用于上传提货清单（PDF Base64），关联已有订单。

描述 | 内容
--- | ---
接口功能 | 上传提货清单
请求协议 | HTTPS
请求方法 | POST
请求格式 | `application/json`
请求 URL | `https://api.bz/ltl/v1/logistics/upload_delivery_note`
请求头 | `Content-Type: application/json`；`X-App-Language: zh/en/es`；`Authorization: Bearer <access_token>`
备注 | `stream` 需为 PDF 文件的 Base64；orderId 需准确
响应格式 | JSON

请求体参数：

参数 | 描述 | 必填 | 类型 | 规则/说明
--- | --- | --- | --- | ---
orderId | 订单号 | 是 | number | 正整数；为提货单号依据
stream | PDF Base64 | 是 | string | 仅支持 PDF，Base64 编码

请求头参数：

参数名 | 示例值 | 必填 | 描述
--- | --- | --- | ---
Content-Type | application/json | 是 | 请求体 JSON
X-App-Language | en | 是 | 返回语言（`zh`、`en`、`es`）
Authorization | Bearer your_access_token | 是 | 访问令牌

请求示例：
```
POST https://api.bz/ltl/v1/logistics/upload_delivery_note
Content-Type: application/json
X-App-Language: en
Authorization: Bearer your_access_token

{
  "orderId": 1100000000000006,
  "stream": "BASE64_OF_PDF"
}
```

响应示例（200）：
```
{
  "code": 0,
  "message": "Success",
  "data": {}
}
```

错误处理：
- 400：参数缺失或格式错误（如 Base64 非法）。
- 401：认证失败，检查 Authorization。
- 404：接口/资源不存在。
- 500：服务器异常。

前端校验建议：
- `orderId`：正整数。
- `stream`：Base64 字符串；文件类型应为 PDF（可在上传前校验 MIME）。
- Header：`Content-Type: application/json`、`X-App-Language` 合法、`Authorization` 携带有效 token。

# Changelog
- v1.3：新增上传提货清单接口。
- v1.2：新增物流下单接口。
- v1.1：刷新 Access Token 接口明确为无请求体，仅需 Header 携带 Bearer Token。
- v1.0：获取访问令牌接口文档。

