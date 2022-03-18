# Client

## POST order

|Name | data|
--- | ---
|Endpoint| `https://endpoint.ucs.hu/wp-json/vendor/v1/postorder/<restaurant_id>`|
|Method| POST|
|Authentication| basich auth|
|User-Agent (header)| has to be defined, suggested: `restaurant_name-bot`|
|referer| has to be defined, suggested: the restaurant's URL|
|Payload| JSON|

### Important Notes:
- **Always use HTTPS connection!**
- **Always add `User-Agent` header to the request**
- **Always add `referer` to the request**

### Authentication
**Use basich auth!** The UCS support will create the user *(11-17 characters, /^[0-9_]+/)* and the password *(30 characters, /^[0-9a-f]{30}$/)* and set the available restaurant_ids for the givven user and password combination.
User example: `409_16_475_105`
Password example: `813dfdrr2d597rktfefd0766b70b61`
Expected authentication header format in the POST request:
`authorization: Basic NDA5XzE2XzQ3NV8xMDU6MGYzN2M1MTc0MGM34DNmMDyiODFlYlf1NDIkdmI5`

### Payload (post order)

#### Data
- objectid - object identifier (required, same as the restaurant_id in the URL)
- delivery_time - delivery date and time, in the format 'YYYY-MM-DDTHH:mm:ss' (required)
- comment - a comment to the order (optional, max 255 character)
- order_type - order type: 0 - delivery order, 1 - self-pickup order (required)
- pay_type - type of payment: 0 - cash to the courier, 1 - by bank card to the courier (required)
- pay_online_type - payed online: 1 -- will be payed offline: 0
- client - client data
    - phone - phone number (required)
    - email - email (optional)
    - ln - last name (required)
    - fn - name (optional)
    - mn - middle name (optional)
- address - delivery address (required if the order is for delivery)
    - country - country
    - city - city
    - street - street
    - house - house
    - building - building
    - entry - entrance
    - floor - floor
    - apartments - apartment
- order - order composition (list)
    - id - dish identifier
    - type - type. 'd' - dish, 'c' - combo dish, 'm' - modifier
    - qnt - quantity in thousands (if 1 piece, then the value will be 1000)
    - items - a list of components (modifiers)
        - id - identifier
        - type - type. "M" - modifier, "d" - dish (if the main course is a combo dish)
        - qnt - quantity
        - items - list of modifiers / combo components

#### Example
```
{
    "objectid": "101150001",
    "delivery_time": "2021-11-26T23:26:01",
    "comment": "testcomment",
    "order_type": 0,
    "pay_type": 0,
    "pay_online_type": 0,
    "client": {
        "phone": "+361110000",
        "email": "test@test.hu",
        "ln": "Testln",
        "fn": "Testfn"
    },
    "order": [
        {
            "id": "1002217",
            "qnt": 5000,
            "type": "d",
            "items": []
        },
        {
            "id": "1001552",
            "qnt": 1000,
            "type": "d",
            "items": []
        }
    ]
}
```

### Success
|Name | data|
--- | ---
|Reponse code| 200|
|Reponse body| json|
|Reponse body| 👇|

```
{
    "success": true,
    "data": {
        "remoteResponse": {
            "remoteOrderId": 237
        },
        "data": {
            "seq_number": 4,
            "visit_id": "1618091731-589-109150003"
        },
        "source": "endpoint_server",
        "status": "Ok"
    }
}
```

### Failed

**Expected authentication/validation error reponses**

|Error code | Error message | Example
--- | --- | ---
|404 | No route was found matching the URL and request method. | `{"code":"rest_no_route","message":"No route was found matching the URL and request method.","data":{"status":404}}`|
|400 | Missing authorization header. | `{"success":false,"data":"Missing authorization header."}` |
|400 | Malformed authentication header. | `{"success":false,"data":"Malformed authentication header."}` |
|403 | Invalid vendor id or token | `{"success":false,"data":"Invalid vendor id or token"}` |
|400 | Missing `<x>` from the request. | `{"success":false,"data":"Missing <x> from the request."}` |
|400 | Mallformed POST data: `<x>`. | `{"success":false,"data":"Mallformed POST data: <x>."}` |
|400 | If custom `order_number` is enabled, but missing from the payload | `{"success":false,"data":"Missing `order_number` from the request."}` |
|400 | If custom `order_number` is enabled, and the provided one is not numeric and between 0-99999 | `{"success":false,"data":"Invalid `order_number` format."}` |
|500 | Server error occured. Please try again later or get in touch with the server provider. | `{"success":false,"data":"Server error occured. Please try again later or get in touch with the server provider."}` |

### 🔌 Use custom order_number
If it's enabled on the endpoint server it's possible to use custom order_number. The order_number is a unique identifier for the order and will be used on the restaurant terminal side.

#### Notes
- **Important** It has to be  enabled on the endpoint server. Please ask the server provider to make it avilable for you.
- If it's not enabled, the server will ignore the extra field and fall back the default order_number.
- If the custom `order_number` option is enabled but the POST request doesn't contain the `order_number` field, the server will fall back the default order_number.
- The `order_number` must be numeric and between 0 and 99999.

#### Data / payload
The payload of the POST request is the same as the one for the POST order. But you can provide an extra field `order_number` with the order_number you want to use.

#### Example payload
```
{
    "objectid": "101150001",
    "delivery_time": "2021-11-26T23:26:01",
    "comment": "testcomment",
    "order_type": 0,
    "pay_type": 0,
    "pay_online_type": 0,
    "client": {
        "phone": "+361110000",
        "email": "test@test.hu",
        "ln": "Testln",
        "fn": "Testfn"
    },
    "order": [
        {
            "id": "1002217",
            "qnt": 5000,
            "type": "d",
            "items": []
        },
        {
            "id": "1001552",
            "qnt": 1000,
            "type": "d",
            "items": []
        }
    ],
    "order_number": "12345"
}
```

#### Example success response

The response's `data>data>seq_number` has to be the same as the custom `order_number` you provided.

```
{
    "success": true,
    "data": {
        "remoteResponse": {
            "remoteOrderId": 237
        },
        "data": {
            "seq_number": 12345,
            "visit_id": "1618091731-589-109150003"
        },
        "source": "endpoint_server",
        "status": "Ok"
    }
}
```



---

## POST order payed

When an order where the `pay_online_type` value is 1 is posted, it will be created in a "pending" state. It stays in the pendign state while the endpoint recieves a payed order post for it. Orders in pending state are not forwarded to the terminal in the restaurant.

|Name | data|
--- | ---
|Endpoint| `https://endpoint.ucs.hu/wp-json/vendor/v1/postorder/payed/<restaurant_id>/<visit_id>`|
|Method| POST|
|Authentication| basich auth|
|Payload| NONE|

- `visit_id` is in the return Json of the pos order

### Success
|Name | data|
--- | ---
|Reponse code| 200|
|Reponse body| json|
|Reponse body| 👇|

```
{
    "success": true,
    "data": {
        "remoteResponse": {
            "remoteOrderId": 237
        },
        "data": {
            "seq_number": 4,
            "visit_id": "1618091731-589-109150003"
        },
        "source": "endpoint_server",
        "status": "Ok"
    }
}
```
### Failed

**Expected authentication/validation error reponses**

|Error code | Error message | Example
--- | --- | ---
|404 | No route was found matching the URL and request method. | `{"code":"rest_no_route","message":"No route was found matching the URL and request method.","data":{"status":404}}`|
|400 | Missing authorization header. | `{"success":false,"data":"Missing authorization header."}` |
|400 | Malformed authentication header. | `{"success":false,"data":"Malformed authentication header."}` |
|403 | Invalid vendor id or token | `{"success":false,"data":"Invalid vendor id or token"}` |
|404 | Order with this visit id can't be found. | `{"success":false,"data":"Order with this visit id can't be found."}` |


## POST order status

|Name | data|
--- | ---
|Endpoint| `https://endpoint.ucs.hu/wp-json/vendor/v1/order/status/<restaurant_id>/<visit_id>/<new_status>`|
|Method| POST|
|Authentication| basich auth|
|Payload| NONE|

### Important Notes:
- **Always use HTTPS connection!**
- **Always add `User-Agent` header to the request**
- **Always add `referer` to the request**
- **The new status <new_status> options are provided by the UCS team. Only the pre-registered status options (eg.: received, in_delivery, delivered, ...) can be used!**

### Authentication
**Use basich auth!** The UCS support will create the user *(11-17 characters, /^[0-9_]+/)* and the password *(30 characters, /^[0-9a-f]{30}$/)* and set the available restaurant_ids for the givven user and password combination.
User example: `409_16_475_105`
Password example: `813dfdrr2d597rktfefd0766b70b61`
Expected authentication header format in the POST request:
`authorization: Basic NDA5XzE2XzQ3NV8xMDU6MGYzN2M1MTc0MGM34DNmMDyiODFlYlf1NDIkdmI5`

### Success
|Name | data|
--- | ---
|Reponse code| 200|
|Reponse body| json|
|Reponse body| 👇|

```
{
    "success": true,
    "data": {
        "remoteResponse": {
            "remoteOrderId": 237
        },
        "data": {
            "visit_id": "1618091731-589-109150003",
            "new_status": "<the passed new status>",
            "prev_status": "<the order's previous status. {string or null}>"
        },
        "source": "endpoint_server",
        "status": "Ok"
    }
}
```

### Failed

**Expected authentication/validation error reponses**

|Error code | Error message | Example
--- | --- | ---
|404 | No route was found matching the URL and request method. | `{"code":"rest_no_route","message":"No route was found matching the URL and request method.","data":{"status":404}}`|
|400 | Missing authorization header. | `{"success":false,"data":"Missing authorization header."}` |
|400 | Malformed authentication header. | `{"success":false,"data":"Malformed authentication header."}` |
|403 | Invalid vendor id or token | `{"success":false,"data":"Invalid vendor id or token"}` |
|404 | Order with this visit id can't be found. | `{"success":false,"data":"Order with this visit id can't be found."}` |
|400 | The posted order status is not allowed in the system. | `{"success":false,"data":"The posted order status is not allowed in the system."}` |
|400 | No available order status can be found. | `{"success":false,"data":"No available order status can be found."}` |
|500 | Server error occured. Please try again later or get in touch with the server provider. | `{"success":false,"data":"Server error occured. Please try again later or get in touch with the server provider."}` |

## GET available statuses for the restaurant

|Name | data|
--- | ---
|Endpoint| `https://endpoint.ucs.hu/wp-json/vendor/v1/statuses/<restaurant_id>`|
|Method| GET|
|Authentication| basich auth|
|Payload| NONE|

### Important Notes:
- **Always use HTTPS connection!**
- **Always add `User-Agent` header to the request**
- **Always add `referer` to the request**

### Authentication
**Use basich auth!** The UCS support will create the user *(11-17 characters, /^[0-9_]+/)* and the password *(30 characters, /^[0-9a-f]{30}$/)* and set the available restaurant_ids for the givven user and password combination.
User example: `409_16_475_105`
Password example: `813dfdrr2d597rktfefd0766b70b61`
Expected authentication header format in the POST request:
`authorization: Basic NDA5XzE2XzQ3NV8xMDU6MGYzN2M1MTc0MGM34DNmMDyiODFlYlf1NDIkdmI5`

### Success
|Name | data|
--- | ---
|Reponse code| 200|
|Reponse body| json|
|Reponse body| 👇|

```
{
    "success": true,
    "data": {
        "status_slug": "status Name",
        "status2_slug": "status2 Name"
    }
}
```

### Failed

**Expected authentication/validation error reponses**

|Error code | Error message | Example
--- | --- | ---
|404 | No route was found matching the URL and request method. | `{"code":"rest_no_route","message":"No route was found matching the URL and request method.","data":{"status":404}}`|
|400 | Missing authorization header. | `{"success":false,"data":"Missing authorization header."}` |
|400 | Malformed authentication header. | `{"success":false,"data":"Malformed authentication header."}` |
|403 | Invalid vendor id or token | `{"success":false,"data":"Invalid vendor id or token"}` |
|400 | No available order status can be found. | `{"success":false,"data":"No available order status can be found."}` |
|500 | Server error occured. Please try again later or get in touch with the server provider. | `{"success":false,"data":"Server error occured. Please try again later or get in touch with the server provider."}` |

## GET order status

|Name | data|
--- | ---
|Endpoint| `https://endpoint.ucs.hu/wp-json/vendor/v1/order/status/<restaurant_id>/<visit_id>`|
|Method| GET|
|Authentication| basich auth|
|Payload| NONE|

### Important Notes:
- **Always use HTTPS connection!**
- **Always add `User-Agent` header to the request**
- **Always add `referer` to the request**

### Authentication
**Use basich auth!** The UCS support will create the user *(11-17 characters, /^[0-9_]+/)* and the password *(30 characters, /^[0-9a-f]{30}$/)* and set the available restaurant_ids for the givven user and password combination.
User example: `409_16_475_105`
Password example: `813dfdrr2d597rktfefd0766b70b61`
Expected authentication header format in the POST request:
`authorization: Basic NDA5XzE2XzQ3NV8xMDU6MGYzN2M1MTc0MGM34DNmMDyiODFlYlf1NDIkdmI5`

### Success
|Name | data|
--- | ---
|Reponse code| 200|
|Reponse body| json|
|Reponse body| 👇|

```
{
    "success": true,
    "data": {
        "remoteResponse": {
            "remoteOrderId": 237
        },
        "data": {
            "visit_id": "1618091731-589-109150003",
            "order_status": "<the order status>",
        },
        "source": "endpoint_server",
        "status": "Ok"
    }
}
```

### Failed

**Expected authentication/validation error reponses**

|Error code | Error message | Example
--- | --- | ---
|404 | No route was found matching the URL and request method. | `{"code":"rest_no_route","message":"No route was found matching the URL and request method.","data":{"status":404}}`|
|400 | Missing authorization header. | `{"success":false,"data":"Missing authorization header."}` |
|400 | Malformed authentication header. | `{"success":false,"data":"Malformed authentication header."}` |
|403 | Invalid vendor id or token | `{"success":false,"data":"Invalid vendor id or token"}` |
|400 | No available order status can be found. | `{"success":false,"data":"No available order status can be found."}` |
|500 | Server error occured. Please try again later or get in touch with the server provider. | `{"success":false,"data":"Server error occured. Please try again later or get in touch with the server provider."}` |
|404 | Order with this visit id can't be found. | `{"success":false,"data":"Order with this visit id can't be found."}` |
