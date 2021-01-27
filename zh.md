# 交易加速-合作厂商自动支付API文档

## 签名鉴权
>授权ID：access_id

>授权密钥：secret_key

**签名方法**  
1，将POST的dict按照 `key1=value1&key2=value2&key3=value3`按照key升序排序组成待签名字符串  
2，hmac/sha1 使用secret_key对字符串进行签名，生成signature，并对signature进行base64编码 

**环境**  
测试环境：http://8.210.181.251:8122 密钥请申请，仅接口调用，未请求加速

生产环境 https://pushtx.com 密钥请申请。

**签名生成示例**
#### NODE
```
// 参数按照key正序排列
const payloadKeys: string[] = Object.keys(payload).sort();
// tslint:disable-next-line:variable-name
let originString: string = '';
for (const key of payloadKeys) {
    if (key === 'signature') {
        continue;
    }
    // @ts-ignore
    originString += `${key}=${payload[key]}&`;
}
originString = originString.slice(0, -1);
let enCryptoSHA1 = crypto
    .createHmac('sha1', secretKey)
    .update(originString)
    .digest('hex');
enCryptoSHA1 = Buffer.from(enCryptoSHA1, 'hex').toString('base64');
```
#### PYTHON
```
#!/usr/bin/env python3
import base64
import hashlib
import hmac

import requests


def sign_dict(secret_key, parameters):
    sortedparameters = sorted(parameters.items(), key=lambda para: para[0])
    canonicalizedquerystring = []
    for (k, v) in sortedparameters:
        canonicalizedquerystring.append('{0}={1}'.format(k, v))
    canonicalizedquerystring = '&'.join(canonicalizedquerystring)
    h = hmac.new(secret_key.encode('utf8'), canonicalizedquerystring.encode('utf8'), hashlib.sha1)
    signature = base64.b64encode(h.digest()).strip()
    return signature


class PushtxMerchant(object):
    def __init__(self, access_id, secret_key, endpoint, timeout=30):
        self.access_id = access_id
        self.secret_key = secret_key
        self.endpoint = endpoint
        self.timeout = timeout

    def create_order(self, transaction_id, description=''):
        data = {
            'access_id': self.access_id,
            'transaction_id': transaction_id,
            'description': description,
        }
        data['signature'] = sign_dict(self.secret_key, data).decode('utf8')
        import json
        print(json.dumps(data,indent=2))
        headers = {'Content-Type': 'application/json'}
        url = self.endpoint + '/xxxxx'
        response = requests.post(url, headers=headers, json=data, timeout=self.timeout)
        print(json.dumps(response.json(),indent=2))
        return response.json()


if __name__ == '__main__':
    pushtx = PushtxMerchant('xxxxx', 'xxxxx', 'http://xxxxx:xxxx')
    res = pushtx.create_order('xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx')
    print(res)
```

## 接口列表
#### 下单接口
>/api/merchant-payment/order
```
Request:
{
    access_id: 应用id,   
    payment_type: 支付类型[cash、opennode],
    payment_coin: 支付币种类型, [如非cash可以为空] 选填
    transaction_id: 交易hash, 
    signature: 签名,  
}
Response:
{
    "err_no": 0, // 错误编码
    "data": {
        "order_id": "4tml1ftn2qo2njtd_1611726972363", // 订单编号
        "transaction_id": "9cf8293d8633c030e7c8e9fc92efb301a3f73a5877250a15a88ded4b2e6bea3f", // 交易hash
        "status": "unpaid", // 交易状态
        "total_size": 1222, // 交易体积
        "payment_channel": "trxusdt", // 支付方式
        "payment_amount": 115.47,   // 支付数量
        "payment_address": "TByTMoubyhqfU8sjGbyzjcr1Hqimtkwbvd" // 支付Address地址
    },
    "message": "success"
}
```
#### 订单详情
>/api/merchant-payment/order-detail
```
Request:
{
    access_id: 应用id,   
    order_id: 订单ID,【支持多ID联合查询，以","分割，最多一次查询10个ID】 
    signature: 签名,  
}
Response:
{
    "err_no": 0,
    "data": [
        {
            "order_id": "4tml1ftn2qo2njtd_1611726972363",
            "transaction_id": "9cf8293d8633c030e7c8e9fc92efb301a3f73a5877250a15a88ded4b2e6bea3f",
            "status": "unpaid",
            "payment_channel": "trxusdt",
            "payment_amount": 11547,
            "payment_address": "TByTMoubyhqfU8sjGbyzjcr1Hqimtkwbvd",
            "created_at": "2021-01-26T21:56:23.000Z",
            "updated_at": "2021-01-26T21:56:23.000Z"
        },
        {
            "order_id": "iv2pscdqctl6q31m_1611720158591",
            "transaction_id": "1ecc705eb4e152d90d370d24445976e35eeac54bd59f653d7a4891ed3ec636e3",
            "status": "unpaid",
            "payment_channel": "trxusdt",
            "payment_amount": 2157,
            "payment_address": "TB64Q4uR7N8mEDnirZMxQzHy7e9icDEvvE",
            "created_at": "2021-01-26T20:02:42.000Z",
            "updated_at": "2021-01-26T20:02:42.000Z"
        }
    ],
    "message": "success"
}
```

**err_no错误代码：**
* 1001 access_id参数不符合规范
* 1002 交易哈希参数不符合规范
* 1003 签名参数不符合规范
* 1004 订单详情-查询订单数超过限制
* 4001 access_id参数校验失败
* 4002 签名参数校验失败
* 2001 hash无效
* 2002 hash已被确认
* 5001 订单创建失败
* 0 请求正常  
