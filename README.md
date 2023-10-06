# Client API Documentation

API consists of two components:
* [**json-rpc**](#json-rpc) to work with orders, watch available banks/currencies registries, and check reports;
* [**sse (server-side-events)**](#sse) to subscribe on orders events in real time.

---

Access protocol: **https**

For client authentication, we use **mutual tls** and **ip address whitelists** on PoP (Point-of-Present) gateway-instances to control access at the network level.

---

### JSON-RPC

endpoint: **/rpc**

#### Methods

##### Fetch Registries (read-only)
* [client/registry/banks](#fetch-list-of-available-banks) - returns list of available banks for the client;
* [client/registry/currencies](#fetch-list-of-available-currencies) - returns list of available currencies for the client.

##### Create/Cancel Orders
* [client/order/withdraw/create](#create-new-order-to-withdraw-funds) - creates new order to withdraw funds;
* [client/order/withdraw/cancel](#cancel-withdraw-order) - cancels withdraw order specified by identifier;
* [client/order/deposit/create](#create-new-order-to-deposit-funds) - creates new order to deposit funds;
* [client/order/deposit/cancel](#cancel-deposit-order) - cancels deposit order specified by identifier.

##### Get Order Status (read-only)
* [client/order/status](#get-order-status) - returns current state of the order specified by identifier.

##### Get Reports (read-only)
* [client/report/today](#get-report-for-today) - returns statistics from 00:00 of today to the current time;
* [client/report/yesterday](#get-report-for-yesterday) - returns statistics from 00:00 to 23:59:59 yesterday;
* [client/report/24hr](#get-report-for-24hr) - returns statistics for the last 24 hours.

#### JSON-RPC Request Object
| key      | description       | value type | value example                   |
| -------- | ----------------- | ---------- | ------------------------------- |
| `method` | method name       | string     | `"client/order/deposit/create"` |
| `params` | method parameters | object     | `{"client_tx_id": "tx_01"}`     |

#### JSON-RPC Response Object
| key      | description     | value type | value example                                        |
| -------- | --------------- | ---------- | ---------------------------------------------------- |
| `result` | on success only | object     | `{"result":{"code":200,"text":"ok"}}`                |
| `error`  | on failure only | object     | `{"validation":{"field":"uuid","reason":"required"}` |

### Examples

#### Fetch List of Available Banks
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/registry/banks**

params: omit

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data '{"method": "client/registry/banks"}'
```

response:
```json
{
  "result": {
    "count": 2,
    "banks": [
      {
        "uuid": "a8234838-db88-4476-b32b-49640899e685",
        "title": "emirates bank"
      },
      {
        "uuid": "52c50471-a179-4c6d-9d53-393f5d35cb82",
        "title": "capital bank"
      }
    ]
  }
}
```

#### Fetch List of Available Currencies
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/registry/currencies**

params: omit

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data '{"method": "client/registry/currencies"}'
```

response:
```json
{
  "result": {
    "count": 1,
    "currencies": [
      {
        "uuid": "4b96405b-654c-4f16-9527-eb42e650c8bb",
        "title": "uae dirham",
        "code": "aed"
      }
    ]
  }
}
```

#### Create New Order to Withdraw Funds
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/order/withdraw/create**

params:

| key              | description          | value type     | value example                            | mandatory |
| ---------------- | -------------------- | -------------- | ---------------------------------------- | --------- |
| `client_tx_id`   | client-side tx id    | string         | `"clientside_uniq_tx_id_01"`             | required  |
| `client_user_id` | client-side user id  | string         | `"clientside_uniq_user_id_01"`           | optional  |
| `currency_uuid`  | transaction currency | string         | `"4b96405b-654c-4f16-9527-eb42e650c8bb"` | required  |
| `bank_uuid`      | transaction bank     | string         | `"52c50471-a179-4c6d-9d53-393f5d35cb82"` | optional  |
| `sum`            | order sum            | decimal string | `"5000"`                                 | required  |
| `pan`            | card pan             | string         | `"1111222233334444"`                     | required  |
| `name`           | card holder name     | string         | `"John Doe"`                             | optional  |

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data @- <<EOF
{
  "method": "client/order/withdraw/create",
  "params": {
    "client_tx_id": "tx7438041958",
    "client_user_id": "user110",
    "currency_uuid": "4b96405b-654c-4f16-9527-eb42e650c8bb",
    "bank_uuid": "52c50471-a179-4c6d-9d53-393f5d35cb82",
    "sum": "1570.50",
    "pan": "3030333305057070",
    "name": "John Doe"
  }
}
EOF
```

response:
```json
{
  "result": {
    "uuid": "0e810a3c-93ad-4472-bc05-8dd56d9c531f",
    "status": "created",
    "created_ts": 1696489762,
    "expire_ts": 1696490362,
    "updated_ts": 1696489762,
    "client_tx_id": "tx7438041958",
    "client_user_id": "user110",
    "sum": "1570.5",
    "pan": "3030333305057070",
    "name": "John Doe",
    "bank": {
      "uuid": "52c50471-a179-4c6d-9d53-393f5d35cb82",
      "title": "capital bank"
    },
    "currency": {
      "uuid": "4b96405b-654c-4f16-9527-eb42e650c8bb",
      "title": "uae dirham",
      "code": "aed"
    }
  }
}
```

#### Cancel Withdraw Order
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/order/withdraw/cancel**

params:

| key            | description          | value type | value example                            | mandatory |
| -------------- | -------------------- | ---------- | ---------------------------------------- | --------- |
| `uuid`         | order uuid           | string     | `"0e810a3c-93ad-4472-bc05-8dd56d9c531f"` | optional  |
| `client_tx_id` | client-side tx id    | string     | `"clientside_uniq_tx_id_01"`          | optional  |

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data @- <<EOF
{
  "method": "client/order/withdraw/cancel",
  "params": {
    "client_tx_id": "tx7438041958",
  }
}
EOF
```

response:
```json
{
  "result": {
    "code": 200,
    "text": "ok"
  }
}
```

#### Create New Order to Deposit Funds
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/order/deposit/create**

params:

| key              | description          | value type     | value example                            | mandatory |
| ---------------- | -------------------- | -------------- | ---------------------------------------- | --------- |
| `client_tx_id`   | client-side tx id    | string         | `"clientside_uniq_tx_id_01"`             | required  |
| `client_user_id` | client-side user id  | string         | `"clientside_uniq_user_id_01"`           | optional  |
| `currency_uuid`  | transaction currency | string         | `"4b96405b-654c-4f16-9527-eb42e650c8bb"` | required  |
| `bank_uuid`      | transaction bank     | string         | `"52c50471-a179-4c6d-9d53-393f5d35cb82"` | optional  |
| `sum`            | order sum            | decimal string | `"5000"`                                 | required  |
| `pan`            | card pan             | string         | `"1111222233334444"`                     | optional  |
| `name`           | card holder name     | string         | `"John Doe"`                             | optional  |

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data @- <<EOF
{
  "method": "client/order/deposit/create",
  "params": {
    "client_tx_id": "tx7438041959",
    "client_user_id": "user110",
    "currency_uuid": "4b96405b-654c-4f16-9527-eb42e650c8bb",
    "bank_uuid": "52c50471-a179-4c6d-9d53-393f5d35cb82",
    "sum": "2570.50",
    "pan": "4040333305057070",
    "name": "John Doe"
  }
}
EOF
```

response:
```json
{
  "result": {
    "uuid": "bc29825c-5207-46c0-a0c2-cc9253bbf4ca",
    "status": "created",
    "created_ts": 1696490746,
    "expire_ts": 1696491346,
    "updated_ts": 1696490746,
    "client_tx_id": "tx7438041959",
    "client_user_id": "user110",
    "sum": "2570.5",
    "pan": "4040333305057070",
    "name": "John Doe",
    "bank": {
      "uuid": "52c50471-a179-4c6d-9d53-393f5d35cb82",
      "title": "capital bank"
    },
    "currency": {
      "uuid": "4b96405b-654c-4f16-9527-eb42e650c8bb",
      "title": "uae dirham",
      "code": "aed"
    }
  }
}
```

#### Cancel Deposit Order
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/order/deposit/cancel**

params:

| key            | description          | value type | value example                            | mandatory |
| -------------- | -------------------- | ---------- | ---------------------------------------- | --------- |
| `uuid`         | order uuid           | string     | `"bc29825c-5207-46c0-a0c2-cc9253bbf4ca"` | optional  |
| `client_tx_id` | client-side tx id    | string     | `"clientside_uniq_tx_id_01"`             | optional  |

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data @- <<EOF
{
  "method": "client/order/deposit/cancel",
  "params": {
    "uuid": "bc29825c-5207-46c0-a0c2-cc9253bbf4ca"
  }
}
EOF
```

response:
```json
{
  "result": {
    "code": 200,
    "text": "ok"
  }
}
```

#### Get Order Status
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/order/status**

params:

| key            | description          | value type | value example                            | mandatory |
| -------------- | -------------------- | ---------- | ---------------------------------------- | --------- |
| `uuid`         | order uuid           | string     | `"bc29825c-5207-46c0-a0c2-cc9253bbf4ca"` | optional  |
| `client_tx_id` | client-side tx id    | string     | `"clientside_uniq_tx_id_01"`             | optional  |

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data @- <<EOF
{
  "method": "client/order/status",
  "params": {
    "uuid": "bc29825c-5207-46c0-a0c2-cc9253bbf4ca"
  }
}
EOF
```

response:
```json
{
  "result": {
    "uuid": "bc29825c-5207-46c0-a0c2-cc9253bbf4ca",
    "direction": "deposit",
    "status": "canceled",
    "created_ts": 1696490746,
    "expire_ts": 1696491346,
    "updated_ts": 1696490934,
    "client_tx_id": "tx7438041959",
    "client_user_id": "user110",
    "sum": "2570.5",
    "pan": "4040333305057070",
    "name": "John Doe",
    "bank": {
      "uuid": "a8234838-db88-4476-b32b-49640899e685",
      "title": "emirates bank"
    },
    "card": {
      "uuid": "aa788504-14e6-4f60-86fc-e33463e1e1d9",
      "number": "4242999933335555"
    },
    "currency": {
      "uuid": "4b96405b-654c-4f16-9527-eb42e650c8bb",
      "title": "uae dirham",
      "code": "aed"
    }
  }
}
```

#### Get Report For Today
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/report/today**

params: omit

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data '{"method": "client/report/today"}'
```

response:
```json
{
  "result": {
    "total": 5,
    "total_in": 2,
    "total_out": 3,
    "success": 0,
    "success_in": 0,
    "success_out": 0,
    "canceled": 2,
    "canceled_in": 1,
    "canceled_out": 1,
    "expired": 3,
    "failed": 0,
    "failed_in": 0,
    "failed_out": 0
  }
}
```

#### Get Report For Yesterday
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/report/yesterday**

params: omit

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data '{"method": "client/report/yesterday"}'
```

response:
```json
{
  "result": {
    "total": 5,
    "total_in": 2,
    "total_out": 3,
    "success": 0,
    "success_in": 0,
    "success_out": 0,
    "canceled": 2,
    "canceled_in": 1,
    "canceled_out": 1,
    "expired": 3,
    "failed": 0,
    "failed_in": 0,
    "failed_out": 0
  }
}
```

#### Get Report For 24hr
```
endpoint: /rpc
content-type: application/json
method: POST
```

method: **client/report/24hr**

params: omit

request:
```bash
curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --header 'content-type: application/json' \
  --request 'POST' \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/rpc' \
  --data '{"method": "client/report/24hr"}'
```

response:
```json
{
  "result": {
    "total": 5,
    "total_in": 2,
    "total_out": 3,
    "success": 0,
    "success_in": 0,
    "success_out": 0,
    "canceled": 2,
    "canceled_in": 1,
    "canceled_out": 1,
    "expired": 3,
    "failed": 0,
    "failed_in": 0,
    "failed_out": 0
  }
}
```

### SSE

endpoint prefix: **/sse**

#### Methods
* [/sse/client/events](#subscribe-on-order-events) - sends events, such as order creation and status changes.

**IMPORTANT:** alternatively, you can provide us with the callback url
and receive notifications that way.

### Examples

#### Subscribe on Order Events
```
endpoint: /sse/client/events
content-type: text/event-stream
method: GET
```

request:
```bash
$ curl \
  --cacert './server_cert.pem' \
  --cert './client_cert.pem' \
  --key './client_key.pem' \
  --request 'GET' \
  --no-buffer \
  --resolve "api.payone:{PORT}:{IP_ADDRESS}" \
  --url 'https://api.payone:{PORT}/sse/client/events' \
```

response stream:
```bash
: events stream

retry: 5000
event: order
data: {"uuid":"90dcb27e-cba9-4307-b889-ec81d8152d56","status":"created", …}

event: order
data: {"uuid":"90dcb27e-cba9-4307-b889-ec81d8152d56","status":"pending", …}

event: order
data: {"uuid":"90dcb27e-cba9-4307-b889-ec81d8152d56","status":"success", …}

…
```
