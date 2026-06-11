---
name: "anqifu-payment"
description: "安企付(AQF)统一支付接口接入指南。Invoke when user asks about integrating with AnQiFu payment gateway, creating payment orders, handling callbacks, or implementing AQF unified payment API."
---

# 安企付统一支付接口接入指南

## 概述

安企付统一支付接口支持多种支付方式：
- 微信扫收款码支付 (W01)
- 微信公众号支付 (W02)
- 微信小程序支付 (W06)
- 微信小程序插件支付 (W07)
- 支付宝扫收款码支付 (A01)
- 支付宝服务窗支付 (A02)
- 云闪付支付 (U02)

## 接口信息

- **请求地址**：`https://interface-aqf.aieppay.com/api/access/payInterface/unifiedPay`
- **请求方式**：POST
- **Content-Type**：`application/x-www-form-urlencoded`

## 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| merchantNo | String | 是 | 安企付商户编号。若传集团商户号，则根据业务决定是否必填 |
| groupNo | String | 否 | 集团商户号 |
| storeNo | String | 否 | 安企付门店编号 |
| merchantOrderNo | String | 是 | 商户订单编号，必须唯一 |
| amount | String | 是 | 交易金额，单位：分 |
| notifyUrl | String | 是 | 交易结果回调通知地址 |
| backUrl | String | 否 | 交易成功回跳地址（仅支持 U02、W02） |
| payType | String | 是 | 交易方式，详见下方说明 |
| acct | String | 条件必填 | 用户标识。W02/W06 为 openid，A02 为 userid，U02 为银联 userId |
| orderName | String | 否 | 订单名称/订单信息 |
| note | String | 否 | 异步回调所带参数 |
| remark | String | 否 | 商户订单备注信息 |
| subAppid | String | 否 | 微信小程序/公众号 appid，多 appid 绑定时必填 |
| validTime | String | 否 | 有效时间(yyyyMMddHHmmss)，目前留空 |
| asinfo | String | 否 | 分账信息，格式：merchantNo:type:amount;... |
| goods | List | 否 | 单品优惠信息（W01 不支持） |
| chnlstoreid | String | 否 | 渠道门店编号，与 goods 联合使用 |
| goodsTag | String | 否 | 订单优惠标记 |
| mchCreateIp | String | 是 | 终端 IP 地址 |
| consumerIp | String | 条件必填 | 用户 IP，使用集团商户号且 merchantNo 不填时必填 |
| royalty | String | 否 | 余额分账标识：0-普通交易，1-余额分账，默认 0 |
| sign | String | 是 | 签名，详见签名规则 |

## 响应参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| status | String | 状态：0000-成功，9999-失败 |
| errorCode | String | 错误代码（失败时返回） |
| errorMessage | String | 错误信息（失败时返回） |
| merchantNo | String | 商户编号 |
| orderNo | String | 安企付订单编号 |
| amount | String | 交易金额，单位：分 |
| payinfo | String | 支付信息串 |
| appId | String | 小程序 APPID（payType=W07 时返回） |

## 交易方式 (payType)

| 代码 | 说明 |
|------|------|
| W01 | 微信扫收款码支付 |
| W02 | 微信公众号支付 |
| W06 | 微信小程序支付 |
| W07 | 微信小程序插件支付 |
| A01 | 支付宝扫收款码支付 |
| A02 | 支付宝服务窗支付 |
| U02 | 云闪付 JS 支付 |

## 签名规则

### 签名步骤

1. **参数排序**：将所有请求参数（除 sign 外）按照参数名 ASCII 码从小到大排序
2. **拼接字符串**：使用 `&` 连接键值对，格式为 `key1=value1&key2=value2`
3. **追加密钥**：在拼接好的字符串末尾追加 `&key=您的密钥`
4. **MD5 加密**：对完整字符串进行 MD5 加密，得到 sign 值

### 签名示例

```
原始参数（已排序）：
amount=100
merchantNo=121050000024
merchantOrderNo=1234567891220
notifyUrl=http://your-domain.com/notify

拼接字符串：
amount=100&merchantNo=121050000024&merchantOrderNo=1234567891220&notifyUrl=http://your-domain.com/notify&key=your_secret_key

MD5 加密后得到 sign 值
```

## 代码示例

### Java 示例

```java
import java.security.MessageDigest;
import java.util.*;

public class AqfPayment {
    
    private static final String API_URL = "https://interface-aqf.aieppay.com/api/access/payInterface/unifiedPay";
    
    /**
     * 创建统一支付订单
     */
    public static Map<String, String> createOrder(Map<String, String> params, String secretKey) {
        // 生成签名
        String sign = generateSign(params, secretKey);
        params.put("sign", sign);
        
        // 发送 POST 请求
        return sendPostRequest(API_URL, params);
    }
    
    /**
     * 生成签名
     */
    public static String generateSign(Map<String, String> params, String secretKey) {
        // 过滤空值和 sign 参数
        Map<String, String> filteredParams = new TreeMap<>();
        for (Map.Entry<String, String> entry : params.entrySet()) {
            if (entry.getValue() != null && !entry.getValue().isEmpty() 
                && !"sign".equals(entry.getKey())) {
                filteredParams.put(entry.getKey(), entry.getValue());
            }
        }
        
        // 拼接字符串
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<String, String> entry : filteredParams.entrySet()) {
            if (sb.length() > 0) sb.append("&");
            sb.append(entry.getKey()).append("=").append(entry.getValue());
        }
        sb.append("&key=").append(secretKey);
        
        // MD5 加密
        return md5(sb.toString()).toUpperCase();
    }
    
    private static String md5(String input) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] digest = md.digest(input.getBytes("UTF-8"));
            StringBuilder sb = new StringBuilder();
            for (byte b : digest) {
                sb.append(String.format("%02x", b));
            }
            return sb.toString();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
    
    private static Map<String, String> sendPostRequest(String url, Map<String, String> params) {
        // 使用 HttpClient 或 OkHttp 发送请求
        // 返回解析后的 JSON 响应
        return new HashMap<>();
    }
}
```

### Python 示例

```python
import hashlib
import urllib.parse
import requests

API_URL = "https://interface-aqf.aieppay.com/api/access/payInterface/unifiedPay"

def generate_sign(params: dict, secret_key: str) -> str:
    """生成签名"""
    # 过滤空值和 sign 参数
    filtered = {k: v for k, v in params.items() 
                if v is not None and v != "" and k != "sign"}
    
    # 按 ASCII 排序并拼接
    sorted_items = sorted(filtered.items(), key=lambda x: x[0])
    sign_str = "&".join([f"{k}={v}" for k, v in sorted_items])
    sign_str += f"&key={secret_key}"
    
    # MD5 加密并转大写
    return hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()

def create_order(params: dict, secret_key: str) -> dict:
    """创建支付订单"""
    params['sign'] = generate_sign(params, secret_key)
    
    response = requests.post(API_URL, data=params)
    return response.json()

# 使用示例
order_params = {
    "merchantNo": "121050000024",
    "merchantOrderNo": "ORDER202406110001",
    "amount": "100",
    "notifyUrl": "https://your-domain.com/notify",
    "payType": "W02",
    "acct": "用户openid",
    "orderName": "测试订单",
    "mchCreateIp": "127.0.0.1"
}

result = create_order(order_params, "your_secret_key")
print(result)
```

### PHP 示例

```php
<?php

class AqfPayment {
    const API_URL = 'https://interface-aqf.aieppay.com/api/access/payInterface/unifiedPay';
    
    /**
     * 生成签名
     */
    public static function generateSign($params, $secretKey) {
        // 过滤空值和 sign
        $filtered = array_filter($params, function($v, $k) {
            return $v !== null && $v !== '' && $k !== 'sign';
        }, ARRAY_FILTER_USE_BOTH);
        
        // 按键名排序
        ksort($filtered);
        
        // 拼接字符串
        $signStr = http_build_query($filtered);
        $signStr .= '&key=' . $secretKey;
        
        // MD5 加密转大写
        return strtoupper(md5($signStr));
    }
    
    /**
     * 创建订单
     */
    public static function createOrder($params, $secretKey) {
        $params['sign'] = self::generateSign($params, $secretKey);
        
        $ch = curl_init(self::API_URL);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($params));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        
        $response = curl_exec($ch);
        curl_close($ch);
        
        return json_decode($response, true);
    }
}
?>
```

### Node.js 示例

```javascript
const crypto = require('crypto');
const axios = require('axios');
const querystring = require('querystring');

const API_URL = 'https://interface-aqf.aieppay.com/api/access/payInterface/unifiedPay';

/**
 * 生成签名
 */
function generateSign(params, secretKey) {
    // 过滤空值和 sign
    const filtered = Object.entries(params)
        .filter(([k, v]) => v !== null && v !== '' && v !== undefined && k !== 'sign')
        .sort(([a], [b]) => a.localeCompare(b));
    
    // 拼接字符串
    const signStr = filtered.map(([k, v]) => `${k}=${v}`).join('&') + `&key=${secretKey}`;
    
    // MD5 加密转大写
    return crypto.createHash('md5').update(signStr, 'utf8').digest('hex').toUpperCase();
}

/**
 * 创建订单
 */
async function createOrder(params, secretKey) {
    params.sign = generateSign(params, secretKey);
    
    const response = await axios.post(API_URL, querystring.stringify(params), {
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' }
    });
    
    return response.data;
}

// 使用示例
const orderParams = {
    merchantNo: '121050000024',
    merchantOrderNo: 'ORDER202406110001',
    amount: '100',
    notifyUrl: 'https://your-domain.com/notify',
    payType: 'W02',
    acct: '用户openid',
    orderName: '测试订单',
    mchCreateIp: '127.0.0.1'
};

createOrder(orderParams, 'your_secret_key')
    .then(result => console.log(result))
    .catch(err => console.error(err));
```

## 支付信息使用说明

### 微信公众号支付 (W02)

响应中的 `payinfo` 为 JSON 字符串，包含以下字段：
- `appId`：公众号 appid
- `timeStamp`：时间戳
- `nonceStr`：随机字符串
- `package`：统一下单接口返回的 prepay_id
- `signType`：签名类型
- `paySign`：支付签名

前端调用微信 JSAPI：
```javascript
wx.chooseWXPay({
    ...JSON.parse(payinfo),
    success: function(res) {
        // 支付成功
    }
});
```

### 微信小程序支付 (W06)

响应中的 `payinfo` 为 JSON 字符串，调用小程序支付 API：
```javascript
wx.requestPayment({
    ...JSON.parse(payinfo),
    success(res) {
        // 支付成功
    },
    fail(res) {
        // 支付失败
    }
});
```

### 微信小程序插件支付 (W07)

响应中的 `appId` 为可被拉起支付的小程序 APPID，`payinfo` 为带参路径。

### 支付宝服务窗支付 (A02)

响应中的 `payinfo` 为支付 JSON 串，参考支付宝官方文档调用。

## 回调通知处理

### 通知参数

| 参数名 | 说明 |
|--------|------|
| merchantNo | 商户编号 |
| orderNo | 安企付订单编号 |
| merchantOrderNo | 商户订单编号 |
| amount | 交易金额，单位：分 |
| status | 交易状态：1-成功 |
| payTime | 支付时间 |
| payType | 支付方式 |
| note | 异步回调所带参数 |
| sign | 签名 |

### 回调处理示例

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/notify', methods=['POST'])
def handle_notify():
    data = request.form.to_dict()
    
    # 1. 验证签名
    received_sign = data.pop('sign', None)
    calculated_sign = generate_sign(data, "your_secret_key")
    
    if received_sign != calculated_sign:
        return "SIGN_ERROR", 400
    
    # 2. 处理业务逻辑
    if data.get('status') == '1':
        # 支付成功，更新订单状态
        order_no = data.get('merchantOrderNo')
        # TODO: 更新数据库
        pass
    
    # 3. 返回成功响应
    return "SUCCESS"

if __name__ == '__main__':
    app.run(port=8080)
```

## 常见问题

### Q: 签名失败怎么办？
A: 请检查：
1. 参数是否按 ASCII 码排序
2. 空值参数是否已过滤
3. 密钥是否正确
4. MD5 结果是否转大写

### Q: 回调通知未收到？
A: 请检查：
1. notifyUrl 是否可公网访问
2. 是否正确返回 "SUCCESS"
3. 服务器防火墙是否放行

### Q: 如何测试？
A: 测试环境和生产环境使用相同域名：`https://interface-aqf.aieppay.com`，请联系安企付获取测试商户号。

## 相关接口

- 交易查询：`/api/access/payInterface/queryOrder`
- 交易退款：`/api/access/payInterface/refund`
- 关闭订单：`/api/access/payInterface/closeOrder`
- 撤销订单：`/api/access/payInterface/reverseOrder`

## 注意事项

1. **订单号唯一性**：`merchantOrderNo` 必须全局唯一，重复提交会被拒绝
2. **金额单位**：所有金额参数单位为**分**，如 1 元传 "100"
3. **签名安全**：密钥不要暴露在前端代码中
4. **回调幂等**：同一笔订单可能收到多次回调，需做好幂等处理
5. **IP 白名单**：确保服务器 IP 已在安企付后台配置
