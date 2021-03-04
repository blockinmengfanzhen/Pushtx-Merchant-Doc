# 交易加速-合作厂商自动支付API文档

## 签名鉴权
>授权ID：access_id

>授权密钥：secret_key

**签名方法**  
1，将POST的dict按照 `key1=value1&key2=value2&key3=value3`按照key升序排序组成待签名字符串  
2，hmac/sha1 使用secret_key对字符串进行签名，生成signature，并对signature进行base64编码 

**环境**  
测试环境：http://8.210.181.251:8122 密钥请申请，仅接口调用，未请求加速

生产环境：https://pushtx.com 密钥请申请。

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

    def create_order(self, transaction_id, payment_type, payment_coin):
        data = {
            'access_id': self.access_id,
            'transaction_id': transaction_id,
            'payment_type': payment_type,
            'payment_coin': payment_coin
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
    pushtx = PushtxMerchant('access_id', 'secret_key', 'http://pushtx.com')
    res = pushtx.create_order('txhash', 'payment_type', 'payment_coin')
    print(res)
```

## 接口列表
#### 下单接口
>/api/merchant-payment/order
```
Request:
{
    access_id: 应用id,   
    payment_type: 支付类型[merchant, opennode],
    payment_coin: 支付币种类型, merchant: [trxusdt, 'ethusdt', 'eth', 'bch', 'ltc', 'doge', 'zec', 'dash']; opennode: [usd, cny, btc]
    transaction_id: 交易hash, 
    signature: 签名,  
}
Response:
{
    "err_no": 0,
    "data": {
        "order_id": "r8vk83w2xj5gogf6_1612181409151",                                           // 订单编号
        "transaction_id": "b0ccb17ff628f618e4d0bcbb1081d9b785dd99f22a2ac49bfa51d95e2480eeaf",   // 交易hash
        "status": "unpaid", // 订单状态
        "total_size": 373,  // 交易提及
        "payment_channel": "opennode",  // 支付方式
        "payment_coin": "btc",  // 支付币种
        "payment_amount": 272535,   // 支付数量 [btc以聪为单位，其他币种正常个数]
        "payment_onchain_address": "2MuDgneCxwK4dgM4Cb9JQ2PvN8ui8PCDL3p",   // 链上支付地址
        "payment_internal_address": "LNTB2725350N1PSP0MA...L9EWURHVQE5PGPSYC3KP" // 内部支付地址 [当支付方式为opennode时会有区分，merchant方式无差别]
    },
    "message": "success" // 请求状态描述
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
            "order_id": "r8vk83w2xj5gogf6_1612181409151",
            "transaction_id": "b0ccb17ff628f618e4d0bcbb1081d9b785dd99f22a2ac49bfa51d95e2480eeaf",
            "status": "unpaid",
            "payment_channel": "opennode",
            "payment_coin": "btc",
            "payment_amount": 272535,
            "payment_onchain_address": "2MuDgneCxwK4dgM4Cb9JQ2PvN8ui8PCDL3p",
            "payment_internal_address": "LNTB2725350N1PSP0MA2PP54W6YR7Y40JTPJMX5FHJZLAZQMXCYQ380XZX9N6VKJPREY00YL40SDPSWGU8V6ECXDMNY7R2X4NK7EMXXE0NZD33XGCNSVF5XQUNZDF3CQZPGSP5JG72LUS0YV9E83U5QJRML4K2FQ7HFXQ6XKNKVWCET59286W6MVWQ9QY9QSQ26LEYMH4U8FTJGVUPXRZUC7CURVJM37HXPWSGG3KWVYG6DUZAJQ3DVYTP2WCPTA935SNM27XECPZ3USRUSEG2XXXX4L9EWURHVQE5PGPSYC3KP",
            "created_at": "2021-02-01T04:10:18.000Z",
            "updated_at": "2021-02-01T04:10:18.000Z"
        },
        {
            ...
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
* 1005 商户下单，支付类型不符合规范
* 1006 商户下单，支付币种不符合规范
* 4001 access_id参数校验失败
* 4002 签名参数校验失败
* 2001 hash无效
* 2002 hash已被确认
* 5001 订单创建失败
* 5002 服务报价异常
* 0 请求正常  
