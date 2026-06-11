# aieppay-payment-skills

安企付(AQF)统一支付接口接入指南，帮助开发者快速集成安企付支付网关。

## 简介

本项目是 [安企付](https://www.aieppay.com) 统一支付接口的 Skill 文档，支持多种支付方式：

- 微信扫收款码支付 (W01)
- 微信公众号支付 (W02)
- 微信小程序支付 (W06)
- 微信小程序插件支付 (W07)
- 支付宝扫收款码支付 (A01)
- 支付宝服务窗支付 (A02)
- 云闪付 JS 支付 (U02)

## 接口信息

| 项目 | 内容 |
|------|------|
| 请求地址 | `https://interface-aqf.aieppay.com/api/access/payInterface/unifiedPay` |
| 请求方式 | POST |
| Content-Type | `application/x-www-form-urlencoded` |

## 快速开始

### 1. 准备参数

```python
order_params = {
    "merchantNo": "你的商户号",
    "merchantOrderNo": "ORDER202406110001",
    "amount": "100",  # 单位：分
    "notifyUrl": "https://your-domain.com/notify",
    "payType": "W02",
    "acct": "用户openid",
    "orderName": "测试订单",
    "mchCreateIp": "127.0.0.1"
}
```

### 2. 生成签名

```python
import hashlib

def generate_sign(params: dict, secret_key: str) -> str:
    filtered = {k: v for k, v in params.items() 
                if v is not None and v != "" and k != "sign"}
    sorted_items = sorted(filtered.items(), key=lambda x: x[0])
    sign_str = "&".join([f"{k}={v}" for k, v in sorted_items])
    sign_str += f"&key={secret_key}"
    return hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()
```

### 3. 发起请求

```python
import requests

API_URL = "https://interface-aqf.aieppay.com/api/access/payInterface/unifiedPay"

params['sign'] = generate_sign(params, "your_secret_key")
response = requests.post(API_URL, data=params)
result = response.json()
```

## 项目结构

```
anqifu-payment-skills/
├── SKILL.md          # 完整的 Skill 文档（接口详情、代码示例）
└── README.md         # 本文件
```

## 详细文档

完整的接口文档、参数说明、签名规则、多语言示例代码（Java/Python/PHP/Node.js）以及回调处理方案，请查看 [SKILL.md](./SKILL.md)。

## 支持语言

- Java
- Python
- PHP
- Node.js

## 相关接口

- 交易查询：`/api/access/payInterface/queryOrder`
- 交易退款：`/api/access/payInterface/refund`
- 关闭订单：`/api/access/payInterface/closeOrder`
- 撤销订单：`/api/access/payInterface/reverseOrder`

## 注意事项

1. **订单号唯一性**：`merchantOrderNo` 必须全局唯一
2. **金额单位**：所有金额参数单位为**分**，如 1 元传 `"100"`
3. **签名安全**：密钥不要暴露在前端代码中
4. **回调幂等**：同一笔订单可能收到多次回调，需做好幂等处理
5. **IP 白名单**：确保服务器 IP 已在安企付后台配置

## 技术支持

- 安企付官网：[https://www.aieppay.com](https://www.aieppay.com)
- 接口文档：[语雀文档](https://www.yuque.com/aieppay/trade/kark2u4ggdgrtx9o)

## License

MIT
