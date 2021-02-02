# Transaction Acceleration- Automatic Payment API Document

## Signature authentication
>ACCESS ID：access_id

>SECRET KEY：secret_key

**Signature method**  
1. Sort the POST dict according to `key1=value1&key2=value2&key3=value3` in ascending order of key to form the string to be signed
2. hmac/sha1 uses secret_key to sign the string, generates the signature, and base64 encodes the signature

**Surroundings**  
>Test environment: http://8.210.181.251:8122 Please apply for the key, invocation interface only, no acceleration requested

>Production environment https://pushtx.com Please apply for the key.

**Signature generation example**
#### NODE
```
// Parameters are arranged in positive order by key
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

## Interface list
#### Order create
>/api/merchant-payment/order
```
Request:
{
    access_id: Access id,   
    payment_type: type of payment[cash、opennode],
    payment_coin: type of coin, [blank if not cash] optional
    transaction_id: transaction hash, 
    signature: signature,  
}
Response:
{
    "err_no": 0,
    "data": {
        "order_id": "r8vk83w2xj5gogf6_1612181409151",                                           // order id
        "transaction_id": "b0ccb17ff628f618e4d0bcbb1081d9b785dd99f22a2ac49bfa51d95e2480eeaf",   // transaction id
        "status": "unpaid", // payment status
        "total_size": 373,  // transaction size
        "payment_channel": "opennode",  // payment method
        "payment_coin": "btc",  // payment coin
        "payment_amount": 272535,   // payment amount [btc is in satoshis, and other coins are in units]
        "payment_onchain_address": "2MuDgneCxwK4dgM4Cb9JQ2PvN8ui8PCDL3p",   // payment onchain address
        "payment_internal_address":             "LNTB2725350N1PSP0MA2PP54W6YR7Y40JTPJMX5FHJZLAZQMXCYQ380XZX9N6VKJPREY00YL40SDPSWGU8V6ECXDMNY7R2X4NK7EMXXE0NZD33XGCNSVF5XQUNZDF3CQZPGSP5JG72LUS0YV9E83U5QJRML4K2FQ7HFXQ6XKNKVWCET59286W6MVWQ9QY9QSQ26LEYMH4U8FTJGVUPXRZUC7CURVJM37HXPWSGG3KWVYG6DUZAJQ3DVYTP2WCPTA935SNM27XECPZ3USRUSEG2XXXX4L9EWURHVQE5PGPSYC3KP" // Internal payment address [There will be a distinction when the payment method is opennode, and there is no difference in the merchant method]
    },
    "message": "success" // request status description
}
```
#### Order detail
>/api/merchant-payment/order-detail
```
Request:
{
    access_id: access id,   
    order_id: order ID, [support multi-ID query, divided by ",", query up to 10 IDs at a time]
 
    signature: signature,  
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

**err_no error code：**

* 1001 the access_id parameter does not meet the specification

* 1002 the transaction hash parameter does not meet the specification

* 1003 the signature parameter does not meet the specification

* 1004 order details- query order number exceeds limit

* 4001 access_id parameter verification failed

* 4002 signature parameter verification failed

* 2001 hash is invalid

* 2002 hash has been confirmed

* 5001 order creation failed

* 0 the quest is normal
