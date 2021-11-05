# Client

## Order status changed webhook

This webhook is sent when an order status is changed.

### Notes

- The vendor has to request enabaling the status update webhook from the UCS.
- The webhook triggered any time an order status POST endpoint is called.
    - Thus the webhook is not triggered when the order is created.
    - Thus the webhook is triggered even if the status is not changed, but the status POST endpoint was called with the same status.
- The webhooks are bound to <restaurant_id>s.
    - If you have muptliple restaurants, we need to set up multiple webhooks.
    - These webhooks though can post to the same endpoint on the vendro's side.
- The webhooks communicates via TLS and with POST method.

### Vendor Parameters & Requirements

- The vendor has to provide an endpoint URL for the webhook.
    - The URL can be an IP instead of a domain name.
    - The URL can contain a PORT number.
- The vendor's endpoint has to answer with a HTTP 200 or 202 status code.
- The vendor's endpoint has to be reachable from the UCS's server.
- The vendor's endpoint's response size has to be less than 2000000 bytes.
- The vendor's endpoint's response time has to be less than 25 seconds. The webhook system closes the connection after 30 seconds (including resolving, networking).
- The webhook system sends its requests with the following user agent: `ucs-endpoint-bot`.

### Vendor side validation

The webhook either POSTS to an unprotected endpoint or to one using basich auth. If the vendor's endpoint have a basich auth protection, please provide the username and password for the UCS to validate the connection.

### Payload example

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