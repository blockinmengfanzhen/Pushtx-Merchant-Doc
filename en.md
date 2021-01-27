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
    "err_no": 0, // error code
    "data": {
        "order_id": "4tml1ftn2qo2njtd_1611726972363", // order id
        "transaction_id": "9cf8293d8633c030e7c8e9fc92efb301a3f73a5877250a15a88ded4b2e6bea3f", // transaction hash
        "status": "unpaid", // transaction status
        "total_size": 1222, // transaction size
        "payment_channel": "trxusdt", // payment method
        "payment_amount": 115.47,   // payment amount
        "payment_address": "TByTMoubyhqfU8sjGbyzjcr1Hqimtkwbvd" // payment Address address
    },
    "message": "success"
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

**err_no error code：**

* 1001 access_id the parameter does not meet the specification

* 1002 transaction id does not meet the specification

* 1003 the signature does not meet the specification

* 1004 order details- query order number exceeds limit

* 4001 access_id parameter verification failed

* 4002 Signature parameter verification failed

* 2001 hash invalid

* 2002 hash has been confirmed

* 5001 order creation failed

* 0 the request is normal
