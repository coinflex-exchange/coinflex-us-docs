# REST API

**TEST** 

* `https://stgapi.coinflex.us`

**LIVE** site

* `https://api.coinflex.us`

For clients who do not wish to take advantage of CoinFLEX US's native WebSocket API, CoinFLEX US offers a RESTful API that implements much of the same functionality.

## Error Codes

Code | Description |
---- | ----------- |
429 | Rate limit reached |
10001 | General networking failure |
20001 | Invalid parameter |
30001 | Missing parameter |
40001 | Alert from the server |
50001 | Unknown server error |

## Rate Limits

Each IP is limited to:

* 100 requests per second
* 20 POST v1/orders requests per second
* 2500 requests over 5 minutes

Certain endpoints have extra IP restrictions:

* `s` denotes a second
* Requests limited to `1/s` & `2/10s`
  * Only 1 request is permitted per second and only 2 requests are permitted within 10 seconds
* Request limit `1/10s`
  * The endpoint will block for 10 seconds after an incorrect 2FA code is provided (if the endpoint requires a 2FA code)

Affected APIs:

* [POST /v1/withdrawal](?json#rest-api-v1-deposits-and-withdrawals-post-v1-withdrawal)
* [POST /v1/transfer](?json#rest-api-v1-deposits-and-withdrawals-post-v1-transfer)
* [POST /v1/flexasset/mint](?json#rest-api-v1-flex-assets-post-v1-flexasset-mint)
* [POST /v1/flexasset/redeem](?json#rest-api-v1-flex-assets-post-v1-flexasset-redeem)

## Authentication

> **Request**

```json
{
    "Content-Type": "application/json",
    "AccessKey": "<string>",
    "Timestamp": "<string>",
    "Signature": "<string>",
    "Nonce": "<string>"
}
```

```python
import requests
import hmac
import base64
import hashlib
import datetime
import json


# rest_url = 'https://api.coinflex.us'
# rest_path = 'api.coinflex.us'

rest_url = 'https://stgapi.coinflex.us'
rest_path = 'stgapi.coinflex.us'

api_key = "API-KEY"
api_secret = "API-SECRET"

ts = datetime.datetime.utcnow().isoformat()
nonce = 123
method = "API-METHOD"

# Optional and can be omitted depending on the REST method being called 
body = json.dumps({'key1': 'value1', 'key2': 'value2'})

if body:
    path = method + '?' + body
else:
    path = method

msg_string = '{}\n{}\n{}\n{}\n{}\n{}'.format(ts, nonce, 'GET', rest_path, method, body)
sig = base64.b64encode(hmac.new(api_secret.encode('utf-8'), msg_string.encode('utf-8'), hashlib.sha256).digest()).decode('utf-8')

header = {'Content-Type': 'application/json', 'AccessKey': api_key,
          'Timestamp': ts, 'Signature': sig, 'Nonce': str(nonce)}

resp = requests.get(rest_url + path, headers=header)
# When calling an endpoint that uses body
# resp = requests.post(rest_url + method, data=body, headers=header)
print(resp.json())
```

Public market data methods do not require authentication, however private methods require a *Signature* to be sent in the header of the request.  These private REST methods  use HMAC SHA256 signatures. 

The HMAC SHA256 signature is a keyed HMAC SHA256 operation using a client's API Secret as the key and a message string as the value for the HMAC operation.

The message string is constructed as follows:-

`msgString = f'{Timestamp}\n{Nonce}\n{Verb}\n{URL}\n{Path}\n{Body}'`

Component | Required | Example | Description| 
-------------------------- |--------- |------- |------- | 
Timestamp | Yes | 2020-04-30T15:20:30 | YYYY-MM-DDThh:mm:ss
Nonce | Yes | 123 | User generated
Verb | Yes| GET | Uppercase
Path | Yes | stg.coinflex.us |
Method | Yes | /v1/positions | Available REST methods
Body | No | marketCode=BTC-USD-SWAP-LIN | Optional and dependent on the REST method being called

The constructed message string should look like:-

  `2020-04-30T15:20:30\n
  123\n
  GET\n
  stg.coinflex.us\n
  /v1/positions\n
  marketCode=BTC-USD-SWAP-LIN`

Note the newline characters after each component in the message string. 
If *Body* is omitted it's treated as an empty string.

Finally, you must use the HMAC SHA256 operation to get the hash value using the API Secret as the key, and the constructed message string as the value for the HMAC operation. Then encode this hash value with BASE-64.  This output becomes the signature for the specified authenticated REST API method. 

The signature must then be included in the header of the REST API call like so:

`header = {'Content-Type': 'application/json', 'AccessKey': API-KEY, 'Timestamp': TIME-STAMP, 'Signature': SIGNATURE, 'Nonce': NONCE}`


## Account & Wallet - Private

### GET `/v1/accounts`

Get account information

> **Request**

```
GET /v1/accounts
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "accountId": "21213",
            "name": "main",
            "accountType": "LINEAR",
            "balances": [
                {
                    "asset": "BTC",
                    "total": "2.823",
                    "available": "2.823",
                    "reserved": "0",
                    "lastUpdatedAt": "1593627415234"
                },
                {
                    "asset": "FLEX",
                    "total": "1585.890",
                    "available": "325.890",
                    "reserved": "1260",
                    "lastUpdatedAt": "1593627415123"
                }
            ],
            "notionalBalance": "2314221",
            "feeTier": "6",
            "createdAt": "1611665624601"
        }
    ]
}
```

Response Field | Type | Description |
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
name | STRING | Account name |
accountType | STRING | Account type |
balances | LIST of dictionaries | |
asset | STRING | Asset name |
total | STRING | Total balance|
available | STRING | Available balance |
reserved | STRING | Reserved balance |
lastUpdatedAt | STRING | Timestamp of updated at |
notionalBalance | STRING | Total dollar value of the account |
feeTier | STRING | Fee tier |
createdAt | STRING | Timestamp indicating when the account was created |


### GET `/v1/wallets`

Get wallet history.

> **Request**

```
GET /v1/wallets?type={type}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
          "accountId": "21213",
          "walletHistory": [
              {
                  "asset": "USD",
                  "type": "deposit",
                  "amount": "10",
                  "createdAt": "162131535213"
              }
          ]
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
type | STRING | NO | Type of the history, e.g. `DEPOSIT` `WITHDRAWAL`, return all if not provided |
limit | ULONG | NO | Default 200, max 500 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. | endTime | ULONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. |

Response Field | Type | Description |
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
asset | STRING | Asset name |
type | STRING | Type of the history |
amount | STRING | Amount |
createdAt | STRING | Millisecond timestamp of created at |


### GET `/v1/balances`

Get balances of accounts.

> **Request**

```
GET /v1/balances?asset={asset}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "accountId": "21213",
            "name": "main",
            "balances": [
                {
                    "asset": "BTC",
                    "total": "4468.823",
                    "available": "4468.823",
                    "reserved": "0",
                    "lastUpdatedAt": "1593627415234"
                },
                {
                    "asset": "FLEX",
                    "total": "1585.890",
                    "available": "325.890",
                    "reserved": "1260",
                    "lastUpdatedAt": "1593627415123"
                }
            ]
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name |

Response Field | Type | Description |
-------------- | ---- | ----------- |
accountId | STRING | Account ID |
name | STRING | Account name |
asset | STRING | Asset name |
total | STRING | Total balance |
available | STRING | Available balance |
reserved | STRING | Reserved balance |
lastUpdatedAt | STRING | Millisecond timestamp of last updated at |


### GET `/v1/trades`

Get historical trades sorted by time in descending order (most recent trades first).

> **Request**

```
GET /v1/trades?marketCode={marketCode}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "orderId": "160067484555913076",
            "clientOrderId": "123",
            "matchId": "160067484555913077",
            "marketCode": "FLEX-USD",
            "side": "SELL",
            "matchedQuantity": "0.1",
            "matchPrice": "0.065",
            "total": "0.0065",
            "orderMatchType": "TAKER",
            "feeAsset": "FLEX",
            "fee": "0.0096",
            "source": "1",
            "matchedAt": "1595514663626"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |
limit | ULONG | NO | Default 200, max 500 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | ULONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description |
-------------- | ---- | ----------- |
orderId | STRING | Order ID which generated by the server |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
matchId | STRING | Match ID |
marketCode | STRING | Market code |
side | STRING | Side of the match |
matchQuantity | STRING | Match quantity |
matchPrice | STRING | Match price |
total | STRING | Total cost |
orderMatchType | STRING | Order match type,  available values: `TAKER`,`MAKER` |
feeAsset | STRING | Asset name of the fees |
fees | STRING | Fees |
matchedAt | STRING | Millisecond timestamp of the trade |


## Deposits & Withdrawals - Private

### GET `/v1/deposit-addresses`

Deposit addresses

> **Request**

```
GET /v1/deposit-addresses?asset={asset}&network={network}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "address":"0xD25bCD2DBb6114d3BB29CE946a6356B49911358e"
    }
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES |
network | STRING | YES |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
address | STRING | Deposit address |
memo | STRING | Memo (tag) if applicable |


### GET `/v1/deposit`

Get deposit histories sorted by time in descending order (most recent deposits first).

> **Request**

```
GET /v1/deposit?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD",
            "network": "SLP",
            "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
            "quantity": "1000.0",
            "status": "COMPLETED",
            "txId": "38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d",
            "creditedAt": "1617940800000"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Default all assets |
limit | ULONG | NO | Default 50, max 200 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | ULONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | | 
network | STRING | |
address | STRING | Deposit address |
memo | STRING | Memo (tag) if applicable |
quantity | STRING | |
status | STRING | |
txId | STRING | |
creditedAt | STRING | Millisecond timestamp |


### GET `/v1/withdrawal-addresses`

Withdrawal addresses

> **Request**

```
GET /v1/withdrawal-addresses?asset={asset}?network={network}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "FLEX",
            "network": "ERC20",
            "address": "0x047a13c759D9c3254B4548Fc7e784fBeB1B273g39",
            "label": "farming",
            "whitelisted": true
        }
    ]
}
```

Provides a list of all saved withdrawal addresses along with their respected labels, network, and whitelist status

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO |  Default all assets |
network | STRING | NO | Default all networks |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable |
label | STRING | Withdrawal address label |
whitelisted | BOOL | |


### GET `/v1/withdrawal`

Get withdrawal histories sorted by time in descending order (most recent withdrawals first).

> **Request**

```
GET /v1/withdrawal?id={id}&asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "id": "651573911056351237",
            "asset": "flexUSD",
            "network": "SLP",
            "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
            "quantity": "1000.0",
            "fee": "0.000000000",
            "status": "COMPLETED",
            "txId": "38c09755bff75d33304a3cb6ee839fcb78bbb38b6e3e16586f20852cdec4886d",
            "requestedAt": "1617940800000",
            "completedAt": "16003243243242"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
id | STRING | NO | |
asset | STRING | NO |  Default all assets |
limit | ULONG | NO | Default 50, max 200 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. This filter applies to "requestedAt"|
endTime | ULONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. This filter applies to "requestedAt" |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
id | STRING | |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable |
quantity | STRING | |
fee | STRING | |
id | STRING | |
status | STRING | `COMPLETED`, `PROCESSING`, `PENDING`, `ON HOLD`, `CANCELED`, or `FAILED` |
txId | STRING | |
requestedAt | STRING | Millisecond timestamp |
completedAt | STRING | Millisecond timestamp |


### POST `/v1/withdrawal`

Withdrawal request

> **Request**

```
POST /v1/withdrawal
```
```json
{
    "asset": "flexUSD",
    "network": "SLP",
    "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
    "quantity": "100",
    "externalFee": true,
    "tfaType": "GOOGLE",
    "code": "743249"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "id": "651573911056351237",
        "asset": "flexUSD",
        "network": "SLP",
        "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
        "quantity": "1000.0",
        "externalFee": true,
        "fee": "0",
        "status": "PENDING",
        "requestedAt": "1617940800000"
    }
}
```
Withdrawals may only be initiated by API keys that are linked to the main account and have withdrawals enabled. If the wrong 2fa code is provided the endpoint will block for 10 seconds.

Request Parameter | Type | Required | Description | 
----------------- | ---- |--------- | ----------- |
asset | STRING | YES |
network | STRING | YES |
address | STRING | YES |
memo | STRING | NO |Memo is required for chains that support memo tags |
quantity | STRING | YES |
externalFee | BOOL |YES | If false, then the fee is taken from the quantity |
tfaType | STRING | NO | GOOGLE, or AUTHY_SECRET, or YUBIKEY |
code | STRING | NO | 2fa code if required by the account |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
id | STRING | |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | | 
quantity | STRING | |
externalFee | BOOL | If false, then the fee is taken from the quantity |
fee | STRING | |
status | STRING | |
requestedAt | STRING | Millisecond timestamp |


### GET `/v1/withdrawal-fee`

Withdrawal fee estimate

> **Request**

```
GET /v1/withdrawal-fee?asset={asset}&network={network}&address={address}&memo={memo}&quantity={quantity}&externalFee={externalFee}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "network": "SLP",
        "address": "simpleledger:qzlg6uvceehgzgtz6phmvy8gtdqyt6vf35fxqwx3p7",
        "quantity": "1000.0",
        "externalFee": true,
        "estimatedFee": "0"
    }
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | |
network | STRING | YES | |
address | STRING | YES | |
memo | STRING | NO | Required only for 2 part addresses (tag or memo)|
quantity | STRING | YES | |
externalFee | BOOL | NO | Default false. If false, then the fee is taken from the quantity|

Response Field | Type | Description | 
---------------| ---- | ----------- |
asset | STRING | |
network | STRING | |
address | STRING | |
memo | STRING | Memo (tag) if applicable|
quantity | STRING | |
externalFee | BOOL | If false, then the fee is taken from the quantity|
estimatedFee | STRING | |


## Flex Assets - Private

### POST `/v1/flexasset/mint`

Mint

> **Request**

```
POST /v1/flexasset/mint
```
```json
{
    "asset": "flexUSD",
    "quantity": "100"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "quantity": "100"
    }
}
```
<aside class="notice">
Minting is restricted starting 1 minute before an interest payment until 1 minute after the interest payment.
Interest payments occur at 04:00, 12:00, and 20:00 UTC.
</aside>

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD` |
quantity | STRING | YES | Quantity to mint , minimum quantity required: `10 flexUSD`|

Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING |  |
quantity | STRING | |


### POST `/v1/flexasset/redeem`

Redeem

> **Request**

```
POST /v1/flexasset/redeem
```
```json
{
    "asset": "flexUSD",
    "quantity": "100",
    "type": "NORMAL"
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": {
        "asset": "flexUSD",
        "quantity": "100",
        "type": "NORMAL",
        "redemptionAt": "1617940800000"
    }
}
```

<aside class="notice">
Redemptions of any type are restricted starting 1 minute before an interest payment until 1 minute after the interest payment.
Interest payments occur at 04:00, 12:00, and 20:00 UTC.
</aside>

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | YES | Asset name, available assets: `flexUSD` |
quantity | STRING | YES | Quantity to redeem, minimum quantity required: `10 flexUSD` |
type | STRING | YES | `NORMAL` queues a redemption until the following interest payment and incurs no fee |


Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING |  |
quantity | STRING | |
type | STRING | Available types: `NORMAL` |
redemptionAt | STRING | Millisecond timestamp indicating when redemption will take place|


### GET `/v1/flexasset/mint`

Get historical flexasset mints sorted by time in descending order (most recent mints first).

> **Request**

```
GET /v1/flexasset/mint?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
          "asset": "flexUSD",
          "quantity": "1000.0",
          "mintedAt": "16003243243242"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name, available assets: `flexUSD` |
limit | ULONG | NO | Default 50, max 200 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | ULONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
mintedAt | STRING | |


### GET `/v1/flexasset/redeem`

Get historical flexasset redemptions sorted by time in descending order (most recent redemptions first).

> **Request**

```
GET /v1/flexasset/redeem?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
          "asset": "flexUSD",
          "quantity": "1000.0",
          "requestedAt": "16003243243242",
          "redeemedAt": "16003243243242"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name, available assets: `flexUSD` |
limit | ULONG | NO | Default 50, max 200 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. Here startTime and endTime refer to the "requestedAt" timestamp |
endTime | ULONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. Here startTime and endTime refer to the "requestedAt" timestamp |


Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | |
quantity | STRING | |
requestedAt | STRING | Millisecond timestamp indicating when redemption was requested |
redeemedAt | STRING | Millisecond timestamp indicating when the flexAssets were redeemed (if applicable) |


### GET `/v1/flexasset/earned`

Get historical flexasset interest payments sorted by time in descending order (most recent payments first).

> **Request**

```
GET /v1/flexasset/earned?asset={asset}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "flexUSD",
            "apr": "25",
            "rate": "0.00022831",
            "amount": "2.28310502",
            "paidAt": "1635822660847"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Asset name, available assets: `flexUSD` |
limit | ULONG | NO | Default 50, max 200 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | ULONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description | 
-------------- | ---- | ----------- |
asset | STRING | Asset name |
apr | STRING | Annualized APR (%) = rate * 3 * 365 * 100 |
rate | STRING | Period interest rate |
amount | STRING | |
paidAt | STRING | |


## Orders - Private

### POST `/v1/orders/place`

Place order.

> **Request**

```
POST /v1/orders/place
```
```json
{
    "recvWindow": 3000,
    "timestamp": 1637100050453,
    "responseType":"FULL",
    "orders": [
        {
            "clientOrderId": 1612249737434,
            "marketCode": "USDC-flexUSD",
            "side": "BUY",
            "quantity": "0.01",
            "timeInForce": "GTC",
            "orderType": "LIMIT",
            "price": "0.9998"
        }
    ]
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "notice": "OrderOpened",
            "accountId": "100356",
            "orderId": "1000000705002",
            "submitted": true,
            "clientOrderId": "1612249737434",
            "marketCode": "USDC-flexUSD",
            "status": "OPEN",
            "side": "BUY",
            "price": "0.9998",
            "isTriggered": false,
            "quantity": "0.01",
            "orderType": "LIMIT",
            "timeInForce": "GTC",
            "createdAt": "1650959751803"
        }
    ]
}
```

<aside class="notice">
You can place up to 8 orders at a time in REST API
</aside>

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
recvWindow | ULONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
timestamp | ULONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | A list of orders |
clientOrderId | ULONG | NO | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | YES | Market code |
side | STRING | YES | Side of the order, `BUY` or `SELL` |
quantity | STRING | YES | Quantity submitted |
timeInForce | STRING | NO | Default `GTC` |
orderType | STRING | YES | Type of the order, `LIMIT` or `MARKET` or `STOP` |
price | STRING | NO | Price submitted |
stopPrice | STRING | NO | Stop price for the stop order |

Response Parameters | Type | Description |
--------------------| ---- | ----------- |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
accountId | STRING | Account ID |
orderId | STRING | Order ID which generated by the server |
submitted | BOOL | Whether the request is submitted or not submitted |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Market code |
status | STRING | Order status |
side | STRING | `SELL` or `BUY` |
price | STRING | Price submitted |
stopPrice | STRING | Stop price for the stop order |
isTriggered | STRING | false (or true for STOP order types) |
quantity | STRING | Quantity submitted |
remainQuantity | STRING | Remainning quantity |
matchId | STRING | Exchange match ID |
matchPrice | STRING | Matched price |
matchQuantity | STRING | Matched quantity |
feeInstrumentId | STRING | Instrument ID of fees paid from this match ID |
fees | STRING | Amount of fees paid from this match ID |
orderType | STRING | `MARKET` or `LIMIT` or `STOP` |
timeInForce | STRING | Time in force |
createdAt | STRING | Millisecond timestamp of created at |
lastModifiedAt | STRING | Millisecond timestamp of last modified at |
lastMatchedAt | STRING | Millisecond timestamp of last matched at |


### GET `/v1/orders`

Get order history.

> **Request**

```
GET /v1/orders?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}&limit={limit}&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "orderId": "1000000700000",
            "clientOrderId": "1612249737434",
            "marketCode": "USDC-flexUSD",
            "status": "CLOSED",
            "side": "BUY",
            "price": "0.9998",
            "quantity": "0.01",
            "remainQuantity": "0.01",
            "matchedQuantity": "0",
            "orderType": "LIMIT",
            "timeInForce": "GTC",
            "createdAt": "1650959840448",
            "closedAt": "1650959970325"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |
clientOrderId | ULONG | NO | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
orderId | STRING | Order ID which generated by the server |
limit | ULONG | NO | Default 50, max 200 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other |
endTime | ULONG | NO | Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other |

Response Field | Type | Description |
-------------- | ---- | ----------- |
orderId | STRING | Order ID which generated by the server |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Market code |
status | STRING | Status of the order, available values: `CLOSED` `OPEN` `PARTIALLY_FILLED` `FILLED` |
side | STRING | Side of the order, `BUY` or `SELL` |
price | STRING | Price submitted |
stopPrice | STRING | Stop price for the stop order |
isTriggered | BOOL | Available values: `true` `false`, `true` is for stop orders |
quantity | STRING | Quantity submitted |
remainQuantity | STRING | Remainning quantity |
matchedQuantity | STRING | Matched quantity |
avgFillPrice | STRING | Average of filled price |
fees | LIST of dictionaries | Overall fees with instrument ID, if FLEX is no enough to pay the fee then USD will be paid |
orderType | STRING | Type of the order, `LIMIT` or `STOP` |
timeInForce | STRING | Time in force |
createdAt | STRING | Millisecond timestamp of created at |
lastModifiedAt | STRING | Millisecond timestamp of last modified at |
lastMatchedAt | STRING | Millisecond timestamp of last matched at |
closedAt | STRING | Millisecond timestamp of closed at |


### GET `/v1/orders/working`

Get working order history.

> **Request**

```
GET /v1/orders/working?marketCode={marketCode}&orderId={orderId}&clientOrderId={clientOrderId}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "orderId": "1000000705002",
            "clientOrderId": "1612249737434",
            "marketCode": "USDC-flexUSD",
            "status": "OPEN",
            "side": "BUY",
            "price": "0.9998",
            "quantity": "0.01",
            "remainQuantity": "0.01",
            "matchedQuantity": "0.00",
            "orderType": "LIMIT",
            "timeInForce": "GTC",
            "createdAt": "1650959751718",
            "lastModifiedAt": "1650959751799"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |
orderId | STRING | NO | Order ID which generated by the server |
clientOrderId | ULONG | NO | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |

Response Field | Type | Description |
-------------- | ---- | ----------- |
orderId | STRING | Order ID which generated by the server |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Market code |
status | STRING | Status of the working order, available values: `OPEN` `PARTIALLY_FILLED` |
side | STRING | Side of the order, `BUY` or `SELL` |
price | STRING | Price submitted |
stopPrice | STRING | Stop price for the stop order |
isTriggered | BOOL | Available values: `true` `false`, `true` is for stop orders |
quantity | STRING | Quantity submitted |
remainQuantity | STRING | Remainning quantity |
matchedQuantity | STRING | Matched quantity |
orderType | STRING | Type of the order, `LIMIT` or `STOP` |
timeInForce | STRING | Time in force |
createdAt | STRING | Millisecond timestamp of created at |
lastModifiedAt | STRING | Millisecond timestamp of last modified at |
lastMatchedAt | STRING | Millisecond timestamp of last matched at |


### DELETE `/v1/orders/cancel`

Cancel order by ID.

> **Request**

```
DELETE /v1/orders/cancel
```
```json
{
    "recvWindow": 3000,
    "timestamp": 1737100050453,
    "responseType": "FULL",
    "orders": [
        {
            "marketCode": "BTC-flexUSD",
            "clientOrderId": "1612249737434",
            "orderId": "1000000545000"
        }
    ]
}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "notice": "OrderClosed",
            "accountId": "100020",
            "orderId": "1000000545000",
            "submitted": true,
            "clientOrderId": "1612249737434",
            "marketCode": "BTC-flexUSD",
            "status": "CANCELED_BY_USER",
            "side": "BUY",
            "price": "25000.0",
            "isTriggered": false,
            "quantity": "0.9",
            "remainQuantity": "0.9",
            "orderType": "LIMIT",
            "timeInForce": "GTC",
            "closedAt": "1648200825424"
        }
    ]
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
recvWindow | ULONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
timestamp | ULONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
responseType | STRING | YES | `FULL` or `ACK` |
orders | LIST | YES | A list of orders |
marketCode | STRING | YES | Market code |
clientOrderId | ULONG | Either one of orderId or clientOrderId is required | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
orderId | STRING | Either one of orderId or clientOrderId is required | Order ID |

Response Parameters | Type | Description |
--------------------| ---- | ----------- |
notice | STRING | `OrderClosed` or `OrderMatched` or `OrderOpend` |
accountId | STRING | Account ID |
orderId | STRING | Order ID which generated by the server |
submitted | BOOL | Whether the request is submitted or not submitted |
clientOrderId | STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Market code |
status | STRING | Order status |
side | STRING | `SELL` or `BUY` |
price | STRING | Price submitted |
stopPrice | STRING | Stop price for the stop order |
isTriggered | STRING | false (or true for STOP order types) |
quantity | STRING | Quantity submitted |
remainQuantity | STRING | Remainning quantity |
orderType | STRING | `MARKET` or `LIMIT` or `STOP` |
timeInForce | STRING | Time in force |
closedAt | STRING | Millisecond timestamp of closed at |


### DELETE `/v1/orders/cancel-all`

Cancel all orders.

> **Request**

```
DELETE /v1/orders/cancel-all
```
```json
{
    "marketCode": "BTC-flexUSD"
}
```

> **Successful response format**

```json
{
    "success": true, 
    "data": {
        "notice": "Orders queued for cancelation"
    }
}
```

Request Parameters | Type | Required | Description |
------------------ | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code, if it's null or not sent then cancel all orders for the account |

Response Parameters | Type | Description |
--------------------| ---- | ----------- |
notice | STRING | `Orders queued for cancelation` or `No working orders found` |


## Market Data - Public

### GET `/v1/markets`

Get a list of markets on CoinFlex US.

> **Request**

```
GET /v1/markets?marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USD",
            "name": "BTC/USD",
            "referencePair": "BTC/USD",
            "base": "BTC",
            "counter": "USD",
            "type": "SPOT",
            "tickSize": "0.1",
            "minSize": "0.001",
            "listedAt": "1593345600000",
            "upperPriceBound": "65950.5",
            "lowerPriceBound": "60877.3",
            "markPrice": "63413.9",
            "lastUpdatedAt": "1635848576163"
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market Code |
name | STRING | Name of the contract |
referencePair | STRING | Reference pair |
base | STRING | Base asset |
counter | STRING | Counter asset |
type | STRING | Type of the contract |
tickSize | STRING | Tick size of the contract |
minSize | STRING | Minimum quantity |
listedAt | STRING | Listing date of the contract |
upperPriceBound | STRING | Sanity bound |
lowerPriceBound | STRING | Sanity bound |
markPrice | STRING | Mark price |
lastUpdatedAt | STRING | Millisecond timestamp of last updated time |


### GET `/v1/assets`

Get a list of assets supported on CoinFLEX US.

> **Request**

```
GET /v1/assets?asset={asset}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "asset": "USDC",
            "networkList": [
                {
                    "network": "ERC20",
                    "transactionPrecision": "6",
                    "isWithdrawalFeeChargedToUser": true,
                    "canDeposit": true,
                    "canWithdraw": true,
                    "minWithdrawal": "0.0001"
                }
            ]
        },
        {
            "asset": "BTC",
            "networkList": [
                {
                    "network": "BTC",
                    "transactionPrecision": "8",
                    "isWithdrawalFeeChargedToUser": true,
                    "canDeposit": true,
                    "canWithdraw": true,
                    "minWithdrawal": "0.0001"
                }
            ]
        }
    ]
}
```

Request Parameter | Type | Required | Description | 
----------------- | ---- | -------- | ----------- |
asset | STRING | NO | Name of the asset |

Response Field | Type | Description |
-------------- | ---- | ----------- |
asset | STRING | Asset name |
networkList | LIST | List of dictionaries |
network | STRING | Network for deposit and withdrawal |
transactionPrecision | STRING | Precision for the transaction |
isWithdrawalFeeChargedToUser | BOOL | Indicates the withdrawal fee is charged to user or not |
canDeposit | BOOL | Indicates can deposit or not |
canWithdraw | BOOL | Indicates can withdraw or not |
minDeposit | STRING | Minimum deposit amount |
minWithdrawal | STRING | Minimum withdrawal amount |


### GET `/v1/tickers`

Get tickers.

> **Request**

```
GET /v1/tickers?marketCode={marketCode}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
            "marketCode": "BTC-USD-SWAP-LIN",
            "markPrice": "41512.4",
            "open24h": "41915.3",
            "high24h": "42662.2",
            "low24h": "41167.0",
            "volume24h": "114341.4550",
            "currencyVolume24h": "2.733",
            "lastTradedPrice": "41802.5",
            "lastTradedQuantity": "0.001",
            "lastUpdatedAt": "1642585256002"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market code |
markPrice | STRING | Mark price |
open24h | STRING | 24 hour rolling opening price |
high24h | STRING | 24 hour highest price |
low24h | STRING | 24 hour lowest price |
volume24h | STRING | Volume in 24 hours |
currencyVolume24h | STRING | 24 hour rolling trading volume in counter currency |
lastTradedPrice | STRING | Last traded price |
lastTradedQuantity | STRIN | Last traded quantity |
lastUpdatedAt | STRING | Millisecond timestamp of last updated time |


### GET `/v1/candles`

Get candles.

> **Request**

```
GET /v1/candles?marketCode={marketCode}&timeframe={timeframe}&limit={limit}
&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true, 
    "timeframe": "3600s", 
    "data": [
        {
            "open": "35888.80000000", 
            "high": "35925.30000000", 
            "low": "35717.00000000", 
            "close": "35923.40000000", 
            "volume": "0",
            "currencyVolume": "0",
            "openedAt": "1642932000000"
        },
        {
            "open": "35805.50000000", 
            "high": "36141.50000000", 
            "low": "35784.90000000", 
            "close": "35886.60000000", 
            "volume": "0",
            "currencyVolume": "0",
            "openedAt": "1642928400000"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Market code |
timeframe | STRING | NO | Available values: `60s`,`300s`,`900s`,`1800s`,`3600s`,`7200s`,`14400s`,`86400s`, default is `3600s` |
limit | ULONG | NO | Default 200, max 500 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. |
endTime | ULONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. |

Response Field | Type | Description |
-------------- | ---- | ----------- |
timeframe | STRING | Available values: `60s`,`300s`,`900s`,`1800s`,`3600s`,`7200s`,`14400s`,`86400s` |
open | STRING | Opening price |
high | STRING | Highest price |
low | STRING | Lowest price |
close | STRING | Closing price |
volume | STRING | Trading volume in counter currency |
currencyVolume | STRING | Trading volume in base currency |
openedAt | STRING | Millisecond timestamp of the candle opened at |


### GET `/v1/depth`

Get depth.

> **Request**

```
GET /v1/depth?marketCode={marketCode}&level={level}
```

> **Successful response format**

```json
{
    "success": true, 
    "level": "5", 
    "data": {
        "marketCode": "BTC-USD-SWAP-LIN", 
        "lastUpdatedAt": "1643016065958", 
        "asks": [
            [
                39400, 
                0.261
            ], 
            [
                41050.5, 
                0.002
            ], 
            [
                41051, 
                0.094
            ], 
            [
                41052.5, 
                0.002
            ], 
            [
                41054.5, 
                0.002
            ]
        ], 
        "bids": [
            [
                39382.5, 
                0.593
            ], 
            [
                39380.5, 
                0.009
            ], 
            [
                39378, 
                0.009
            ], 
            [
                39375.5, 
                0.009
            ], 
            [
                39373, 
                0.009
            ]
        ]
    }
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | YES | Market code |
level | ULONG | NO | Default 5, max 100 |

Response Field | Type | Description |
-------------- | ---- | ----------- |
level | ULONG | Level |
marketCode | STRING | Market code |
lastUpdatedAt | STRING | Millisecond timestamp of the depth last updated at |
asks | LIST of floats | Sell side depth: [price, quantity] |
bids | LIST of floats | Buy side depth: [price, quantity] |


### GET `/v1/exchange-trades`

Get historical exchange trades sorted by time in descending order (most recent trades first).

> **Request**

```
GET /v1/exchange-trades?marketCode={marketCode}&limit={limit}
&startTime={startTime}&endTime={endTime}
```

> **Successful response format**

```json
{
    "success": true,
    "data": [
        {
          "marketCode": "BTC-USD-SWAP-LIN",
          "matchPrice": "9600.000000000",
          "matchQuantity": "0.100000000",
          "side": "BUY",
          "matchedAt": "1595585860254"
        }
    ]
}
```

Request Parameter | Type | Required | Description |
----------------- | ---- | -------- | ----------- |
marketCode | STRING | NO | Market code |
limit | ULONG | NO | Default 300, max 300 |
startTime | ULONG | NO | Millisecond timestamp. Default 24 hours ago. startTime and endTime must be within 7 days of each other. |
endTime | ULONG | NO |  Millisecond timestamp. Default time now. startTime and endTime must be within 7 days of each other. |

Response Field | Type | Description |
-------------- | ---- | ----------- |
marketCode | STRING | Market code|
matchPrice | STRING | |
matchQuantity | STRING | |
side | STRING | Aggressor (taker) side, available values: `BUY` `SELL`|
matchedAt | STRING | Millisecond timestamp of matched at |
