# Client

## POST order

|Name | data|
--- | ---
|Endpoint| `https://endpoint.ucs.hu/wp-json/vendor/v1/postorder/<restaurant_id>`|
|Method| POST|
|Authentication| basich auth|
|Payload| JSON|

Notes:
**Always use HTTPS connection!**

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
    "objectid": "109150003",
    "delivery_time": "2020-11-26T23:26:01",
    "comment": "lacitest",
    "order_type": 0,
    "pay_type": 0,
    "pay_online_type": 0,
    "client": {
        "phone": "+36303857256",
        "email": "asdghfasd@djasd.hu",
        "ln": "Laci",
        "fn": "Farkas"
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
|500 | Server error occured. Please try again later or get in touch with the server provider. | `{"success":false,"data":"Server error occured. Please try again later or get in touch with the server provider."}` |

## POST order payed

When an order where the `pay_online_type` value is 1 is posted, it will be created in a "pending" state. It stays in the pendign state while the endpoint recieves a payed order post for it. Orders in pending state are not forwarded to the terminal in the restaurant.

|Name | data|
--- | ---
|Endpoint| `https://endpoint.ucs.hu/wp-json/vendor/v1/postorder/payed/<restaurant_id>/<visit_id>`|
|Method| POST|
|Authentication| basich auth|
|Payload| NONE|

- `visit_id` is in the resturn Json of the pos order

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
