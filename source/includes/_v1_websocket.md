
# Websocket API

> **Subscription request format**

```json
{
  "op": "<value>",
  "tag": "<value>",
  "args": ["<value1>", "<value2>",.....]
}
```

> **Subscription success response format**

```json
{
  "event": "<opValue>",
  "success": True,
  "tag": "<value>",
  "channel": "<argsValue>",
  "timestamp": "1592491945368"
}
```

> **Subscription failure response format**

```json
{
  "event": "<opValue>",
  "success": False,
  "tag": "<value>",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592498745368"
}
```

> **Command request format**

```json
{
  "op": "<value>",
  "tag": "<value>",
  "data": {"<key1>": "<value1>",.....}
}
```

> **Command success response format**

```json
{
  "event": "<opValue>",
  "success": True
  "tag": "<value>",
  "timestamp": "1592498745368",
  "data": {"<key1>": "<value1>",.....}
}
```

> **Command failure response format**

```json
{
  "event": "<opValue>",
  "success": False,
  "message": "<errorMessage>",
  "code": "<codeCode>",
  "timestamp": "1592498745368",
  "data": {"<key1>": "<value1>",.....}
}
```

**TEST** site

* `wss://stgapi.coinflex.us/v1/websocket`

**LIVE** site

* `wss://api.coinflex.us/v1/websocket`

CoinFLEX US's application programming interface (API) provides our clients programmatic access to control aspects of their accounts and to place orders on the CoinFLEX US trading platform. The API is accessible via WebSocket connection to the URIs listed above. Commands, replies, and notifications all traverse the WebSocket in text frames with JSON-formatted payloads.

Websocket commands can be sent in either of the following two formats:

**For subscription based requests**

`{"op": "<value>", "args": ["<value1>", "<value2>",.....]}`

`op`: can either be:

* subscribe
* unsubscribe

`args`: the value(s) will be the instrument ID(s) or asset ID(s), for example:

* order:BTC-flexUSD
* depth:ETH-flexUSD
* position:all

**All other commands**

`{"op": "<command>", "data": {"<key1>":"<value1>",.....}}`

`op`: can be:

* login
* placeorder
* cancelorder
* modifyorder

`data`: JSON string of the request object containing the required parameters

Further information regarding the error codes and corresponding error messages from a failed subscription or order command request can be found in a later section of this documentation [Error Codes](#websocket-api-error-codes).

**WebSocket adapter**

[coinflex-us-ws](https://pypi.org/project/coinflex-us-ws/) is a websocket wrapper to easily connect to CoinFLEX US's websockets.


## Authentication

> **Request format**

```json
{
  "op": "login",
  "tag": "<value>",
  "data": {
            "apiKey": "<string>",
            "timestamp": "<string>",
            "signature": "<string>"
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = 'API-KEY'
api_secret = 'API-SECRET'
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

msg_auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}

async def subscribe():
    async with websockets.connect('wss://stgapi.coinflex.us/v1/websocket') as ws:
        await ws.send(json.dumps(msg_auth))
        while ws.open:
            resp = await ws.recv()
            print(resp)

asyncio.get_event_loop().run_until_complete(subscribe())
```
```javascript
const CryptoJS = require("crypto-js");
const WebSocket = require('ws');

var apiKey = "API-KEY";
var secretKey = "API-SECRET";
const ts = '' + Date.now();

var sign = CryptoJS.enc.Base64.stringify(CryptoJS.HmacSHA256(ts +'GET/auth/self/verify', secretKey));
var msg = JSON.stringify({  
                            "op": "login",
                            "tag": 1,
                            "data": {
                              "apiKey": apiKey,
                              "timestamp": ts,
                              "signature": sign
                            }
                          });

var ws = new WebSocket('wss://stgapi.coinflex.us/v1/websocket');

ws.onmessage = function (e) {
  console.log('websocket message from server : ', e.data);
};

ws.onopen = function () {
    ws.send(msg);
};
```

> **Success response format**

```json
{
  "event": "login",
  "success": true,
  "tag": "<value>",
  "timestamp": "1592491803978"
}
```
```python
{
  "event": "login",
  "success": true,
  "tag": "1",
  "timestamp": "1592491808328"
}
```
```javascript
{
  "event": "login",
  "success": true,
  "tag": "1",
  "timestamp": "1592491808329"
}
```

> **Failure response format**

```json
{
  "event": "login",
  "success": false,
  "code": "<errorCode>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492069732"
}
```
```python
{
  "event": "login",
  "success": false,
  "code": "<errorCode>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492031972"
}
```
```javascript
{
  "event": "login",
  "success": false,
  "code": "<errorCode>",
  "message": "<errorMessage>",
  "tag": "1",
  "timestamp": "1592492031982"
}
```

The Websocket API consists of public and private methods. The public methods do not require authentication.  The private methods requires an authenticated websocket connection.

To autenticate a websocket connection a "login" message must be sent containing the clients signature.

The signature is constructed using a HMAC SHA256 operation to get a hash value, which in turn requires the clients API Secret as the key and a constructed message string as the value for the HMAC operation. This hash value is then encoded as a BASE-64 value which becomes the signature used for authentication.

API keys (public and corresponding secret key) can be generated via the GUI within the clients account.

The message string used in the HMAC SHA256 operation is constructed in the following way:

* `current millisecond timestamp + 'GET/auth/self/verify'`

The signature can therefore be summarised by the following:

* `Base64(HmacSHA256(current_ms_timestamp + 'GET/auth/self/verify', API-Secret))`

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | **'login'** |
tag | INTEGER or STRING | No | If given it will be echoed in the reply
data | DICTIONARY object | Yes |
apiKey | STRING | Yes | Clients public API key, visible in the GUI when created |
timestamp | STRING | Yes | Current millisecond timestamp |
signature | STRING | Yes | `Base64(HmacSHA256(current_ms_timestamp + 'GET/auth/self/verify', API-Secret))` |

## Session Keep Alive

To maintain an active WebSocket connection it is imperative to either be subscribed to a channel that pushes data at least once per minute (Depth or Balance) or send a ping to the server once per minute.

## Order Commands

### Place Limit Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "BTC-flexUSD",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": 1.5,
            "timeInForce": "GTC",
            "price": 9431.48
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_order = \
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "BTC-flexUSD",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": 1.5,
            "timeInForce": "GTC",
            "price": 9431.48
          }
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue

asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json

{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "BTC-flexUSD",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "1.5",
            "timeInForce": "GTC",
            "price": "9431.48",
            "orderId": "1000000700008",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "BTC-flexUSD",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "1.5",
            "timeInForce": "GTC",
            "price": "9431.48",
            "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
data | DICTIONARY object | Yes |
clientOrderId | ULONG | No | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Yes | Market code e.g. `BTC-flexUSD` |
orderType | STRING | Yes |  `LIMIT` |
price | FLOAT |  No | Price |
quantity |  FLOAT | Yes | Quantity (denominated by contractValCurrency) |
side | STRING | Yes | `BUY` or `SELL` |
timeInForce | ENUM | No | <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade)</li></ul>
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |


### Place Market Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-flexUSD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": 5
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = '' 
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_order = \
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-flexUSD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": 5
          }
}


url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1592491945248",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-flexUSD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "5",
            "orderId": "1000001700008",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-flexUSD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "5",
            "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

<sub>**Request Parameters**</sub> 

Parameter | Type | Required | Description |
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
data | DICTIONARY object | Yes |
clientOrderId | ULONG | No | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode | STRING | Yes | Market code e.g. `BTC-flexUSD` |
orderType | STRING | Yes |  `MARKET` |
quantity |  FLOAT | Yes | Quantity (denominated by contractValCurrency) |
side | STRING | Yes | `BUY` or `SELL` |
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |


### Place Stop Limit Order

> **Request format**

```json
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-flexUSD",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": 10,
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": 100,
            "limitPrice": 120
         }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_order = \
{
  "op": "placeorder",
  "tag": 123,
  "data": {
            "timestamp": 1638237934061,
            "recvWindow": 500,
            "clientOrderId": 1,
            "marketCode": "ETH-flexUSD",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": 10,
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": 100,
            "limitPrice": 120
         }
}


url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-flexUSD",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": "100",
            "limitPrice": "120",
            "orderId": "1000002700008",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-flexUSD",
            "side": "BUY",
            "orderType": "STOP_LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY_REPRICE",
            "stopPrice": "100",
            "limitPrice": "120",
            "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

<aside class="notice">
One account can only place up to 50 orders per second via websocket.
</aside>

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
data | DICTIONARY object | Yes |
clientOrderId | ULONG | No | Client assigned ID to help manage and identify orders with max value `9223372036854775807` |
marketCode| STRING| Yes| Market code e.g. `ETH-flexUSD`|
orderType|STRING| Yes|  `STOP_LIMIT` for stop-limit orders (stop-market orders not supported)|
quantity|FLOAT|Yes|Quantity (denominated by contractValCurrency)|
side|STRING| Yes| `BUY ` or `SELL`|
limitPrice| FLOAT |Yes | Limit price for the stop-limit order. <p><p>For **BUY** the limit price must be greater or equal to the stop price.<p><p>For **SELL** the limit price must be less or equal to the stop price.|
stopPrice| FLOAT |Yes|Stop price for the stop-limit order.<p><p>Triggered by the best bid price for the **SELL** stop-limit order.<p><p>Triggered by the best ask price for the **BUY** stop-limit order. |
timeInForce | ENUM | No | <ul><li>`GTC` (Good-till-Cancel) - Default</li><li> `IOC` (Immediate or Cancel, i.e. Taker-only)</li><li> `FOK` (Fill or Kill, for full size)</li><li>`MAKER_ONLY` (i.e. Post-only)</li><li> `MAKER_ONLY_REPRICE` (Reprices order to the best maker only price if the specified price were to lead to a taker trade)</li></ul>
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |


### Place Batch Orders

> **Request format**

```json
{
  "op": "placeorders",
  "tag": 123,
  "dataArray": [{
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 1,
                  "marketCode": "ETH-flexUSD",
                  "side": "BUY",
                  "orderType": "LIMIT",
                  "quantity": 10,
                  "timeInForce": "MAKER_ONLY",
                  "price": 100
                }, 
                {
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 2,
                  "marketCode": "BTC-flexUSD",
                  "side": "SELL",
                  "orderType": "MARKET",
                  "quantity": 0.2
                }]
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
place_batch_order =\
{
  "op": "placeorders",
  "tag": 123,
  "dataArray": [{
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 1,
                  "marketCode": "ETH-flexUSD",
                  "side": "BUY",
                  "orderType": "LIMIT",
                  "quantity": 10,
                  "timeInForce": "MAKER_ONLY",
                  "price": 100
                },
                {
                  "timestamp": 1638237934061,
                  "recvWindow": 500,
                  "clientOrderId": 2,
                  "marketCode": "BTC-USD",
                  "side": "SELL",
                  "orderType": "MARKET",
                  "quantity": 0.2
                }]
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(place_batch_order))
            elif 'event' in data and data['event'] == 'placeorder':
                continue

asyncio.get_event_loop().run_until_complete(subscribe())
```
> **Success response format**

```json
{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739098",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-flexUSD",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY",
            "price": "100",
            "orderId": "1000003700008",
            "source": 0
          }
}

AND

{
  "event": "placeorder",
  "submitted": True,
  "tag": "123",
  "timestamp": "1607639739136",
  "data": {
            "clientOrderId": "2",
            "marketCode": "BTC-USD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "0.2",
            "orderId": "1000004700009",
            "source": 0
          }
}
```

> **Failure response format**

```json
{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503359",
  "data": {
            "clientOrderId": "1",
            "marketCode": "ETH-flexUSD",
            "side": "BUY",
            "orderType": "LIMIT",
            "quantity": "10",
            "timeInForce": "MAKER_ONLY",
            "price": "100",
            "source": 0
          }
}

AND

{
  "event": "placeorder",
  "submitted": False,
  "tag": "123",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491503457",
  "data": {
            "clientOrderId": "2",
            "marketCode": "BTC-USD",
            "side": "SELL",
            "orderType": "MARKET",
            "quantity": "0.2",
            "source": 0
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderOpened, OrderMatched etc......).

All existing single order placement methods are supported:-

* LIMIT
* MARKET
* STOP LIMIT

The websocket reply from the exchange will repond to each order in the batch separately, one order at a time, and has the same message format as the reponse for the single order placement method.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required |Description|
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `placeorders`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
dataArray | LIST of dictionaries | Yes | A list of orders with each order in JSON format, the same format/parameters as the request for placing a single order. The max number of orders is still limited by the message length validation so by default up to 20 orders can be placed in a batch, assuming that each order JSON has 200 characters. |
timestamp | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |
recvWindow | LONG | NO | In milliseconds. If an order reaches the matching engine and the current timestamp exceeds timestamp + recvWindow, then the order will be rejected. If timestamp is provided without recvWindow, then a default recvWindow of 1000ms is used. If recvWindow is provided with no timestamp, then the request will not be rejected. If neither timestamp nor recvWindow are provided, then the request will not be rejected. |


### Cancel Order

> **Request format**

```json
{
  "op": "cancelorder",
  "tag": 456,
  "data": {
            "marketCode": "BTC-flexUSD",
            "orderId": 12
          }
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
cancel_order = \
{
  "op": "cancelorder",
  "tag": 456,
  "data": {
            "marketCode": "BTC-flexUSD",
            "orderId": 12
          }
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(cancel_order))
            elif 'event' in data and data['event'] == 'cancelorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "cancelorder",
  "submitted": True,
  "tag": "456",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-flexUSD",
            "clientOrderId": "1",
            "orderId": "12"
          }
}
```

> **Failure response format**

```json
{
  "event": "cancelorder",
  "submitted": False,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-flexUSD",
            "orderId": "12"
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderClosed etc......).

This command can also be actioned via the trading GUI using the **Cancel** button next to an open order in the **Open Orders** blotter for both Spot and Derivative markets.

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `cancelorder`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
data | DICTIONARY object | Yes |
marketCode|STRING|Yes|Market code e.g. `BTC-flexUSD`|
orderId|INTEGER|Yes|Unique order ID from the exchange|


### Cancel Batch Orders

> **Request format**

```json
{
  "op": "cancelorders",
  "tag": 456,
  "dataArray": [{
                  "marketCode": "BTC-flexUSD",
                  "orderId": 12
                },
                {
                  "marketCode": "BCH-USD",
                  "orderId": 34
                }]
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
cancel_batch_order = \
{
  "op": "cancelorders",
  "tag": 456,
  "dataArray": [{
                  "marketCode": "BTC-flexUSD",
                  "orderId": 12
                },
                {
                  "marketCode": "BCH-USD",
                  "orderId": 34
                }]
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)

            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(cancel_batch_order))
            elif 'event' in data and data['event'] == 'cancelorder':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "cancelorder",
  "submitted": True,
  "tag": "456",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-flexUSD",
            "clientOrderId": "1",
            "orderId": "12"
          }
}

AND

{
  "event": "cancelorder",
  "submitted": True,
  "tag": "456",
  "timestamp": "1592491173978",
  "data": {
            "marketCode": "BCH-USD",
            "orderId": "34"
          }
}

```

> **Failure response format**

```json
{
  "event": "cancelorder",
  "submitted": False,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491173964",
  "data": {
            "marketCode": "BTC-flexUSD",
            "orderId": "12"
          }
}

AND

{
  "event": "cancelorder",
  "submitted": False,
  "tag": "456",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "timestamp": "1592491173989",
  "data": {
            "marketCode": "BCH-USD",
            "orderId": "12"
          }
}
```

Requires an authenticated websocket connection.
Please also subscribe to the **User Order Channel** to receive push notifications for all message updates in relation to an account or sub-account (e.g. OrderClosed etc......).

<sub>**Request Parameters**</sub> 

Parameters | Type | Required | Description
-------------------------- | -----|--------- | -------------|
op | STRING | Yes | `cancelorders`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
dataArray | LIST of dictionaries | A list of orders with each order in JSON format, the same format/parameters as the request for cancelling a single order. The max number of orders is still limited by the message length validation so by default up to 20 orders can be placed in a batch, assuming that each order JSON has 200 characters.


## Subscriptions - Private

All subscriptions to private account channels requires an authenticated websocket connection.

Multiple subscriptions to different channels both public and private can be made within a single subscription command: 

`{"op": "subscribe", "args": ["<value1>", "<value2>",.....]}`


### Balance Channel

> **Request format**

```json
{
  "op": "subscribe",
  "args": ["balance:all"],
  "tag": 101
}

OR

{
  "op": "subscribe", 
  "args": ["balance:USD", "balance:FLEX", ........], 
  "tag": 101
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
balance = \
{
  "op": "subscribe",
  "args": ["balance:all"],
  "tag": 101
}
url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(balance))
            elif 'event' in data and data['event'] == 'balance':
                 continue
asyncio.get_event_loop().run_until_complete(subscribe())

```

> **Success response format**

```json
{
  "success": True, 
  "tag": "101", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

> **Balance channel format**

```json
{
  "table": "balance",
  "accountId": "<Your account ID>",
  "timestamp": "1599693365059",
  "tradeType": "LINEAR",
  "data": [ {
              "total": "10000",
              "reserved": "1000",
              "instrumentId": "USD",
              "available": "9000",
              "quantityLastUpdated": "1599694369431"
            },
            {
              "total": "100000",
              "reserved": "0",
              "instrumentId": "FLEX",
              "available": "100000",
              "quantityLastUpdated": "1599694343242"
            }, 
            ........
          ]
}
```

**Channel Update Frequency** : 250ms

The websocket will reply with the shown success response format for **each** balance asset channel which has been successfully subscribed to.

If a subscription has been made to **balance:all**, the data array in the message from this balance channel will contain a JSON **list**. Each JSON will contain balance details for each spot asset.  Otherwise the data array will contain a **single** JSON corresponding to one spot asset per asset channel subscription.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | LIST | Yes | `balance:all` or a list of individual assets `balance:<assetId>`
tag | INTEGER or STRING | No | If given it will be echoed in the reply

<sub>**Channel Update Fields**</sub> 

Fields |Type| Description |
--------|-----|---|
table | STRING| `balance`
accountId | STRING|  Account identifier
timestamp|STRING | Current millisecond timestamp
tradeType|STRING | `LINEAR`
data | LIST of dictionaries |
total | STRING | Total spot asset balance
reserved | STRING | Reserved asset balance for working spot and repo orders
instrumentId | STRING |  Base asset ID e.g. `BTC`
available | STRING| Remaining available asset balance (total - reserved)
quantityLastUpdated|STRING | Millisecond timestamp


### Order Channel

> **Request format**

```json
{
  "op": "subscribe", 
  "args": ["order:all"], 
  "tag": 102
}

OR

{
  "op": "subscribe", 
  "args": ["order:FLEX-USD", "order:ETH-flexUSD", .....], 
  "tag": 102
}
```
```python
import websockets
import asyncio
import time
import hmac
import base64
import hashlib
import json

api_key = ''
api_secret = ''
ts = str(int(time.time() * 1000))
sig_payload = (ts+'GET/auth/self/verify').encode('utf-8')
signature = base64.b64encode(hmac.new(api_secret.encode('utf-8'), sig_payload, hashlib.sha256).digest()).decode('utf-8')

auth = \
{
  "op": "login",
  "tag": 1,
  "data": {
           "apiKey": api_key,
           "timestamp": ts,
           "signature": signature
          }
}
order = \
{
  "op": "subscribe", 
  "args": ["order:all"], 
  "tag": 102
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                    await ws.send(json.dumps(auth))
            elif 'event' in data and data['event'] == 'login':
                if data['success'] == True:
                    await ws.send(json.dumps(order))
            elif 'event' in data and data['event'] == 'order':
                 continue
asyncio.get_event_loop().run_until_complete(subscribe())

```

> **Success response format**

```json
{
  "success": True, 
  "tag": "102", 
  "event": "subscribe", 
  "channel": "<args value>", 
  "timestamp": "1607985371401"
}
```

**Channel Update Frequency** : real-time, on order update

The websocket will reply with the shown success response format for EACH order channel which has been successfully subscribed to.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes |  `subscribe`
args | LIST | Yes | `order:all` or a list of individual markets `order:<marketCode>`
tag | INTEGER or STRING | No | If given it will be echoed in the reply


#### OrderOpened

> **OrderOpened message format - LIMIT order**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderOpened",
              "accountId": "<Your account ID>",
              "clientOrderId": "16",
              "orderId" : "123",
              "price": "9600",
              "quantity": "2" ,
              "side": "BUY",
              "status": "OPEN",
              "marketCode": "BTC-flexUSD",
              "timeInForce": "MAKER_ONLY",
              "timestamp": "1594943491077"
              "orderType": "LIMIT",
              "isTriggered": "False"
            } ]
}
```

> **OrderOpened message format - STOP LIMIT order**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderOpened",
              "accountId": "<Your account ID>",
              "clientOrderId": "17",
              "orderId": "2245",
              "quantity": "2",
              "side": "BUY",
              "status": "OPEN",
              "marketCode": "USDT-flexUSD",
              "timeInForce": "IOC",
              "timestamp": "1594943491077",
              "stopPrice": "9280",
              "limitPrice": "9300",
              "orderType": "STOP_LIMIT",
              "isTriggered": "True"
            } ]
}
```

<sub>**Channel Update Fields**</sub>

Fields |Type| Description |
--------|-----|---|
table | STRING | `order`
data | LIST of dictionary |
notice | STRING| `OrderOpened`
accountId | STRING| Account identifier
clientOrderId |  STRING | Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING | Unique order ID from the exchange
price |STRING | Limit price submitted (only applicable for LIMIT order types)
quantity | STRING| Quantity submitted
side|STRING|`BUY` or `SELL`
status|STRING|  Order status
marketCode | STRING |  Market code e.g. `FLEX-USD`
timeInForce|STRING| Client submitted time in force, `GTC` by default
timestamp|STRING |Current millisecond timestamp
orderType| STRING | `LIMIT` or `STOP_LIMIT`
stopPrice| STRING |Stop price submitted (only applicable for STOP LIMIT order types)
limitPrice|STRING|Limit price submitted (only applicable for STOP LIMIT order types)
isTriggered|STRING|`False` or `True` 


#### OrderClosed

> **OrderClosed message format - LIMIT order**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderClosed",
              "accountId": "<Your account ID>",
              "clientOrderId": "16",
              "orderId": "73",
              "price": "9600",
              "quantity": "2",
              "side": "BUY",
              "status": "<Canceled status>",
              "marketCode": "BTC-flexUSD",
              "timeInForce": "<Time in force>",
              "timestamp": "1594943491077",
              "remainQuantity": "1.5",
              "orderType": "LIMIT",
              "isTriggered": "False" 
            } ]
}
```

> **OrderClosed message format - STOP LIMIT order**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderClosed",
              "accountId": "<Your account ID>",
              "clientOrderId": "16",
              "orderId": "13",
              "quantity": "2",
              "side": "BUY",
              "status": "CANCELED_BY_USER",
              "marketCode": "BTC-flexUSD",
              "timeInForce": "<Time in force>",
              "timestamp": "1594943491077",
              "remainQuantity": "1.5",
              "stopPrice": "9100",
              "limitPrice": "9120",
              "orderType": "STOP_LIMIT",
              "isTriggered": "True" 
            } ]
}
```

There are multiple scenarios in which an order is closed as described by the **status** field in the OrderClosed message.  In summary orders can be closed by:-

* `CANCELED_BY_USER` - the client themselves initiating this action or the liquidation engine on the clients behalf if the clients account is below the maintenance margin threshold
* `CANCELED_BY_MAKER_ONLY` - if a maker-only order is priced such that it would actually be an agressing taker trade, the order is automatically canceled to prevent this order from matching as a taker
* `CANCELED_BY_FOK` - since fill-or-kill orders requires **all** of the order quantity to immediately take and match at the submitted limit price or better, if no such match is possible then the whole order quantity is canceled
* `CANCELED_ALL_BY_IOC` - since immediate-or-cancel orders also requires an immediate match at the specified limit price or better, if no such match price is possible for **any** of the submitted order quantity then the whole order quantity is canceled
* `CANCELED_PARTIAL_BY_IOC` - since immediate-or-cancel orders only requires **some** of the submitted order quantity to immediately take and match at the specified limit price or better, if a match is possible for only a **partial** quantity then only the remaining order quantity which didn't immediately match is canceled
* `CANCELED_BY_AMEND` - if a **Modify Order** command updated an order such that it changed its position in the order queue, the original order would be closed and an OrderClosed notice would be broadcasted in the Order Channel with this status

<sub>**Channel Update Fields**</sub>

Fields | Type | Description
-------------------------- | -----|--------- |
table | STRING | `order`
data | LIST of dictionary |
notice | STRING | `OrderClosed`
accountId | STRING  |  Account identifier
clientOrderId|STRING |  Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING  |  Unique order ID from the exchange
price|STRING |Limit price of closed order (only applicable for LIMIT order types)
quantity|STRING |Original order quantity of closed order
side|STRING |`BUY` or `SELL`
status|STRING | <ul><li>`CANCELED_BY_USER`</li><li>`CANCELED_BY_MAKER_ONLY`</li><li>`CANCELED_BY_FOK`</li><li>`CANCELED_ALL_BY_IOC`</li><li>`CANCELED_PARTIAL_BY_IOC`</li><li>`CANCELED_BY_AMEND`</li></ul>
marketCode|STRING |  Market code e.g. `BTC-flexUSD`
timeInForce|STRING |Time in force of closed order
timestamp|STRING |Current millisecond timestamp
remainQuantity|STRING |Remaining order quantity of closed order
stopPrice|STRING|Stop price of closed stop order (only applicable for STOP LIMIT order types)
limitPrice|STRING|Limit price of closed stop order (only applicable for STOP LIMIT order types)
ordertype|STRING  | `LIMIT` or `STOP_LIMIT`
isTriggered|STRING|`False` or `True` 


#### OrderClosed Failure

> **OrderClosed failure message format**

```json
{
  "event": "CANCEL", 
  "submitted": False, 
  "message": "Order request was rejected : REJECT_CANCEL_ORDER_ID_NOT_FOUND", 
  "code": "100004", 
  "timestamp": "0", 
  "data": {
            "clientOrderId": 3,
            "orderId": 3330802124194931673, 
            "displayQuantity": 0.0, 
            "lastMatchPrice": 0.0, 
            "lastMatchQuantity": 0.0, 
            "lastMatchedOrderId": 0, 
            "lastMatchedOrderId2": 0, 
            "matchedId": 0, 
            "matchedType": "MAKER", 
            "remainQuantity": 0.0, 
            "side": "BUY", 
            "status": "REJECT_CANCEL_ORDER_ID_NOT_FOUND", 
            "timeCondition": "GTC", 
            "marketCode": "BTC-flexUSD", 
            "timestampEpochMs": 1615377638518, 
            "orderType": "LIMIT",
            "price": 0.0, 
            "quantity": 0.0, 
            "isTriggered": False
          }
}
```

This order message can occur if:- 

* an order has already been matched by the time the cancel order command is recieved and processed by the exchange which means this order is no longer active and therefore cannot be closed.
* multiple cancel order commands for the **same** orderID have been sent in quick sucession to the exchange by mistake and only the first cancel order command is accepted and processed by the exchange which means this order is no longer active and therefore cannot be closed again.

<sub>**Channel Update Fields**</sub>

Fields | Type | Description
-------------------------- | ----- |--------
event | STRING |
submitted | BOOL |
message | STRING |
code | STRING | 
timestamp | STRING |
data | LIST of dictionary |
clientOrderId|STRING|  Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING|   Unique order ID from the exchange
displayQuantity | DECIMAL
lastMatchPrice | DECIMAL
lastMatchQuantity | DECIMAL
lastMatchedOrderId | DECIMAL
lastMatchedOrderId2 | DECIMAL
matchedId | DECIMAL
matchedType | STRING
remainQuantity | DECIMAL
side|STRING
status | STRING
timeCondition | STRING 
marketCode | STRING 
timestampEpochMs | LONG
orderType | STRING
price | DECIMAL
quantity | DECIMAL
isTriggered | BOOL


#### OrderMatched

> **OrderMatched message format**

```json
{
  "table": "order",
  "data": [ {
              "notice": "OrderMatched",
              "accountId": "<Your account ID>",
              "clientOrderId": "16",
              "orderId": "567531657",
              "price": "9300",
              "quantity": "20",
              "side": "BUY",
              "status": "<Matched status>",
              "marketCode": "BTC-flexUSD",
              "timeInForce": "GTC",
              "timestamp": "1592490254168",
              "matchId": "11568123",
              "matchPrice": "9300",
              "matchQuantity": "20",
              "orderMatchType": "MAKER",
              "remainQuantity": "0",
              "orderType": "<Order type>", 
              "stopPrice": "9000",
              "limitPrice": "9050", 
              "fees": "3.7",
              "feeInstrumentId": "FLEX",   
              "isTriggered": "False"
        }
    ]
}
```

<sub>**Channel Update Fields**</sub>

Fields | Type | Required
-------------------------- | -----|--------- |
table | STRING | `order`
data | LIST of dictionary |
notice | STRING | `OrderMatched`
accountId | STRING | Account identifier
clientOrderId|STRING|  Client assigned ID to help manage and identify orders with max value `9223372036854775807`
orderId | STRING|   Unique order ID from the exchange
price|STRING| Limit price submitted (only applicable for LIMIT order types)
stopPrice|STRING| Stop price submitted (only applicable for STOP LIMIT order types)
limitPrice|STRING| Limit price submitted (only applicable for STOP LIMIT order types)
quantity|STRING|Order quantity submitted
side|STRING|`BUY` or `SELL`
status|STRING|`FILLED` or `PARTIAL_FILL`
marketCode|STRING| Market code i.e. `BTC-flexUSD`
timeInForce|STRING|Client submitted time in force (only applicable for LIMIT and STOP LIMIT order types)
timestamp|STRING|Millisecond timestamp of order match
matchID|STRING|Exchange match ID
matchPrice|STRING|Match price of order from this match ID
matchQuantity|STRING|Match quantity of order from this match ID
orderMatchType|STRING|`MAKER` or `TAKER`
remainQuantity|STRING|Remaining order quantity
orderType|STRING|<ul><li>`LIMIT`</li><li>`MARKET`</li><li>`STOP_LIMIT`</li></ul>
fees|STRING|Amount of fees paid from this match ID 
feeInstrumentId|STRING|Instrument ID of fees paid from this match ID 
isTriggered|STRING|`False` (or `True` for STOP LIMIT order types)


## Subscriptions - Public

All subscriptions to public channels do **not** require an authenticated websocket connection.

Multiple subscriptions to different channels both public and private can be made within a single subscription command:

`{"op": "subscribe", "args": ["<value1>", "<value2>",.....]}`

### Orderbook Depth

> **Request format**

```json
{
  "op": "subscribe",
  "tag": 103,
  "args": ["depth:BTC-flexUSD"]
}
```
```python
import websockets
import asyncio
import json

orderbook_depth = \
{
  "op": "subscribe",
  "tag": 103,
  "args": ["depth:BTC-flexUSD"]
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(orderbook_depth))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())

```


> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": "depth:BTC-flexUSD",
  "success": True, 
  "tag": "103", 
  "timestamp": "1607985371601"
}
```

> **Orderbook depth channel format**

```json
{
  "table": "depth",
  "data": [ {
      "instrumentId": "BTC-flexUSD",
      "seqNum": 1608898592006137237,
      "timestamp": "1609350022785",
      "checksum": 1139901235, 
      "asks": [ [5556.82, 11, 0, 0],    //price, quantity, 0, 0
                [5556.84, 98.13, 0, 0],
                [5556.92, 1.582, 0, 0],
                [5557.6, 4.291, 0, 0],
                [5557.85, 2.54, 0, 0]
              ],
      "bids": [ [5556.81, 1.92, 0, 0],  //price, quantity, 0, 0
                [5556.8, 2.1, 0, 0],
                [5556.79, 1.9, 0, 0],
                [5556.19, 100, 0, 0],
                [5556.08, 2.972, 0, 0]
              ]
          } ]
}
```

**Channel Update Frequency:** 50ms

This orderbook depth channel sends a snapshot of the entire orderbook every 50ms.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe` |
tag | INTEGER or STRING | No | If given it will be echoed in the reply |
args | LIST | Yes | List of individual markets `<depth>:<marketCode>` e.g: `[depthL10:BTC-flexUSD]`, the `depth` can be `depthL5` `depthL10` `depthL25` `depth`(includes all) |

<sub>**Channel Update Fields**</sub>

Fields | Type | Description|
-------------------------- | -----| -------------|
table | STRING | `depth` |
data | LIST of dictionary |
instrumentId | STRING |Instrument ID |
seqNum | INTEGER | Sequence number of the order book snapshot |
timestamp| STRING | Millisecond timestamp |
asks| LIST of floats | Sell side depth; <ol><li>price</li><li>quantity</li><li>0</li><li>0</li></ol> |
bids| LIST of floats | Buy side depth; <ol><li>price</li><li>quantity</li><li>0</li><li>0</li></ol> |


### Trade

> **Request format**

```json
{
  "op": "subscribe",
  "tag": 1,
  "args": ["trade:BTC-flexUSD"]
}
```
```python
import websockets
import asyncio
import json

trade = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["trade:BTC-flexUSD"]
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(trade))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())

```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": ["trade:BTC-flexUSD"], 
  "success": True, 
  "tag": "1", 
  "timestamp": "1594299886880"
}
```

> **Trade channel format**

```json
{
  "table": "trade",
  "data": [ {
              "side": "buy",
              "tradeId": "2778148208082945",
              "price": "5556.91",
              "quantity": "5",
              "marketCode": "BTC-flexUSD",
              "timestamp": "1594299886890"
            } ]
}
```

**Channel Update Frequency:** real-time, with every order matched event

This trade channel sends public trade information whenever an order is matched on the orderbook. 

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe` |
tag | INTEGER or STRING | No | If given it will be echoed in the reply
args | LIST | Yes | list of individual markets `trade:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `trade`
data | LIST of dictionary |
tradeId   | STRING    | Transaction Id|
price | STRING    | Matched price|
quantity|STRING   | Matched quantity|
side    |STRING   | Matched side|
timestamp| STRING | Matched timestamp|
marketCode| STRING | Market code |


### Ticker

> **Request format**

```json
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["ticker:all"]
}

OR

{
  "op": "subscribe", 
  "tag": 1,
  "args": ["ticker:FLEX-USD", ........]
}
```
```python
import websockets
import asyncio
import json

ticker = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["ticker:all"]
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(ticker))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": "<args value>",
  "success": True,
  "tag": "1",
  "timestamp": "1594299886890"
}
```

> **Channel update format**

```json
{
    "table": "ticker",
    "data": [
        {
            "last": "0",
            "open24h": "2.80500000",
            "high24h": "3.39600000",
            "low24h": "2.53300000",
            "volume24h": "0",
            "currencyVolume24h": "0",
            "openInterest": "0",
            "marketCode": "1INCH-USD",
            "timestamp": "1622020931049",
            "lastQty": "0",
            "markPrice": "3.304",
            "lastMarkPrice": "3.304"
        },
        {
            "last": "0",
            "open24h": "2.80600000",
            "high24h": "3.39600000",
            "low24h": "2.53300000",
            "volume24h": "0",
            "currencyVolume24h": "0",
            "openInterest": "0",
            "marketCode": "1INCH-flexUSD",
            "timestamp": "1622020931046",
            "lastQty": "0",
            "markPrice": "3.304",
            "lastMarkPrice": "3.304"
        },
        ...
    ]
}
```

**Channel Update Frequency:** 500 ms

The ticker channel pushes live price and volume information about the contract.

The websocket will reply with the shown success response format for **each** ticker channel which has been successfully subscribed to.

The data array in the message from this ticker channel will contain a single JSON corresponding to one ticker subscription.

If you subcribe "ticker:all", you would get one whole message containing all markets.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
args | LIST | Yes | `ticker:all` or a list of individual markets `ticker:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `ticker`
data | LIST of dictionary |
marketCode    | STRING   | Market code |
last          | STRING   | Last traded price|
markPrice     | STRING   | Mark price|
open24h       | STRING   | 24 hour rolling opening price|
volume24h     | STRING   | 24 hour rolling trading volume in counter currency |
currencyVolume24h     | STRING   | 24 hour rolling trading volume in base currency|
high24h     | STRING   | 24 hour highest price|
low24h     | STRING   | 24 hour lowest price|
openInterest     | STRING   | Open interest|
lastQty     | STRING   | Last traded price amount|
timestamp   | STRING   | Millisecond timestamp|


### Candles

> **Request format**

```json
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["candles60s:BTC-flexUSD"]
}
```
```python
import websockets
import asyncio
import json

candles = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["candles60s:BTC-flexUSD"]
}

url= 'wss://stgapi.coinflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(candles))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": ["candles60s:BTC-flexUSD"], 
  "success": True, 
  "tag": "1", 
  "timestamp": "1594313762698"
}
```

> **Channel update format**

```json
{
  "table": "candle60s",
  "data": [ {
              "marketCode": "BTC-flexUSD",
              "candle": [
                "1594313762698", //timestamp
                "9633.1",        //open
                "9693.9",        //high
                "9238.1",        //low
                "9630.2",        //close
                "45247",         //volume in counter currency
                "5.3"            //volume in base currency
              ]
          } ]
}
```

**Channel Update Frequency**: 500ms

**Granularity**: 60s, 180s, 300s, 900s, 1800s, 3600s, 7200s, 14400s, 21600s, 43200s, 86400s

The candles channel pushes candlestick data for the current candle.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
args | LIST | Yes | list of individual candle granularity and market `candles<granularity>:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `candles<granularity>`
data | LIST of dictionary |
marketCode | STRING   | Market code |
candle | LIST of strings  | <ol><li>timestamp</li><li>open</li><li>high</li><li>low</li><li>close</li><li>volume in counter currency</li><li>volume in base currency</li></ol>


### Market

> **Request format**

```json
{
  "op": "subscribe", 
  "tag": 1,
  "args": ["market:all"]
}

OR

{
  "op": "subscribe", 
  "tag": 1,
  "args": ["market:FLEX-USD", ........]
}
```
```python
import websockets
import asyncio
import json

market = \
{
  "op": "subscribe",
  "tag": 1,
  "args": ["market:all"]
}

url= 'wss://stgapi.comflex.us/v1/websocket'
async def subscribe():
    async with websockets.connect(url) as ws:
        while True:
            if not ws.open:
                print("websocket disconnected")
                ws = await websockets.connect(url)
            response = await ws.recv()
            data = json.loads(response)
            print(data)
            if 'nonce' in data:
                await ws.send(json.dumps(market))
            elif 'success' in data and data['success'] == 'True':
                continue
asyncio.get_event_loop().run_until_complete(subscribe())
```

> **Success response format**

```json
{
  "event": "subscribe", 
  "channel": "<args value>",
  "success": True,
  "tag": "1",
  "timestamp": "1594299886890"
}
```

> **Channel update format**

```json
{
    "table":"market",
    "data": [
        {
            "marketCode":"USDC-flexUSD",
            "name":"USDC/flexUSD",
            "referencePair":"USDC/flexUSD",
            "base":"USDC",
            "counter":"flexUSD",
            "type":"SPOT",
            "tickSize":"0.0001",
            "minSize":"0.01",
            "listedAt":"1645430673095",
            "upperPriceBound":"1.0800",
            "lowerPriceBound":"0.9200",
            "markPrice":"1.000000000",
            "lastUpdatedAt":"1645430673204"
        }
    ]
}
```

**Channel Update Frequency:** 1s

The market channel pushes live information about the market such as the current market price and the lower & upper sanity bounds as well as reference data related to the market.

The websocket will reply with the shown success response format for **each** market which has been successfully subscribed to.

If a subscription has been made to **market:all**, the data array in the message from this channel will contain a JSON **list** of all markets. Each JSON will contain information for each market seperately. Otherwise the data array will contain a single JSON corresponding to one market per market channel subscription.

<sub>**Request Parameters**</sub> 

Parameters |Type| Required| Description |
--------|-----|---|-----------|
op | STRING| Yes | `subscribe`
tag | INTEGER or STRING | No | If given it will be echoed in the reply
args | LIST | Yes | `market:all` or a list of individual markets `market:<marketCode>`

<sub>**Channel Update Fields**</sub>

Fields |Type | Description|
-------------------------- | -----|--------- |
table | STRING | `market`
data | LIST of dictionaries |
marketCode   | STRING  | 
name   | STRING   | 
referencePair     | STRING   | 
base     | STRING   | 
counter     | STRING   | 
type     | STRING   | 
tickSize     | STRING   | Tick size |
minSize | STRING | Minimum size |
listedAt | STRING   | Millisecond timestamp |
upperPriceBound | STRING   | Upper sanity price bound|
lowerPriceBound     | STRING   | Lower sanity price bound |
marketPrice | STRING   | Mark price|
marketId | STRING | |
lastUpdatedAt | STRING | Millisecond timestamp|
qtyIncrement | STRING   | Quantity increment |
contractValCurrency     | STRING   | 
marginCurrency   | STRING |

## Other Responses

By subscribing to an authenticated websocket there may be instances when a REST method will also generate a websocket reponse in addition to the REST reply.  There are also some GUI commands which will generate a websocket reponse.

### Cancel All Open Orders

> **Success response format**

```json
{
  "event": "CANCEL",
  "submitted": True,
  "timestamp": "1612476498953"
}
```

Documentation for the REST method for cancelling **all** open orders for an account can be found here [Cancel All Orders](#rest-api-methods-private-delete-v1-cancel-orders).

Documentation for the REST method for cancelling **all** open orders **by market** for an account can be found here [Cancel All Orders By Market](#rest-api-methods-private-delete-v1-cancel-orders-marketcode).

In both these instances a successful action will generate the shown repsonse in an authenticated websocket.

This action can also be executed via the trading GUI using the **Cancel All** button on the **Open Orders** blotter for both Spot and Derivative markets.

## Error Codes

> **Failure response format**

```json
{
  "event": "<opValue>",
  "message": "<errorMessage>",
  "code": "<errorCode>",
  "success": False
}
```

Both subscription and order command requests sent via websocket can be rejected and the failure response will return an error code and a corresponding error message explaining the reason for the rejection.

Code | Error Message 
-----------| -------- 
05001| Your operation authority is invalid
20000| Signature is invalid 
20001| Operation failed, please contact system administrator 
20002| Unexpected error, please check if your request data complies with the specification. 
20003| Unrecognized operation
20005| Already logged in
20006| Quantity must be greater than zero 
20007| You are accessing server too rapidly 
20008| clientOrderId must be greater than zero if provided 
20009| JSON data format is invalid 
20010| Either clientOrderId or orderId is required 
20011| marketCode is required 
20012| side is required 
20013| orderType is required 
20014| clientOrderId is not long type 
20015| marketCode is invalid 
20016| side is invalid 
20017| orderType is invalid 
20018| timeInForce is invalid 
20019| orderId is invalid 
20020| stopPrice or limitPrice is invalid 
20021| price is invalid 
20022| price is required for LIMIT order 
20023| timestamp is required 
20024| timestamp exceeds the threshold 
20025| API key is invalid 
20026| Token is invalid or expired 
20027| The length of the message exceeds the maximum length 
20028| price or stopPrice or limitPrice must be greater than zero 
20029| stopPrice must be less than limitPrice for Buy Stop Order 
20030| limitPrice must be less than stopPrice for Sell Stop Order
100005| Open order not found with id
100006| Open order does not match to the given account
200050| The market is not active
710002| FAILED sanity bound check as price (.....) < lower bound (.....)
710003| FAILED sanity bound check as price (.....) > upper bound (.....)
710004| FAILED net position check as position (.....) > threshold (.....)
710005| FAILED margin check as collateral (.....) < var (.....)   
710006| FAILED balance check as balance (.....) < value (.....)
