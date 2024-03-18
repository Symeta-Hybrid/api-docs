# Webhooks
## Introduction
Once an Intake has been accepted by the API, it will receive status updates via webhooks.

Currently, there are 4 possible status updates:

 * Processed (Intake received successfully, currently being processed by our server)
 * Printing (Intake processed and put on the print queue)
 * Shipped (Intake printed and delivered to bpost for mail delivery)
 * Returned (Postal item was returned to Symeta Hybrid)

These webhooks will be sent to the URI you specify in the Intake, as well as using the specified method (hook.uri and hook.method).

To make use of webhooks, you will need to generate a webhook key in the API interface first.
This key can then be used to verify the contents of the webhook's payload as follows.

Calculate the SHA-256 hash of the JSON payload, using the webhook key as key. If this hash matches the hash that is sent in the header value, the payload is valid.

For example, in PHP:
``` php
<?php
$hash = hash_hmac('sha256', $jsonPayload, $webhookKey);
if ($hash === $responseHeader) {
    // Payload contents verified
    // ...
}
```

## Structure
Each webhook will be sent as an `application/json` payload.

The webhook payload will contain a JSON body with the following keys and header:

| Header    | Value        |
|-----------|--------------|
| Signature | SHA-256 hash |

The payloads are as follows, for each status:

### Processed
``` json
{
    "intake": 158056,
    "status": "Processed",
    "data": ""
}
```

### Printing
``` json
{
    "intake": 158056,
    "status": "Printing"
}
```

### Shipped
``` json
{
    "intake": 158056,
    "status": "Shipped",
    "data": {
        "bpost": "JJBEA000000000000012345", 
        "shipping_date": "15-03-2024"
    },
    "type": "status_update"
}
```

### Returned
``` json
{
    "intake": 158056,
    "status": "Returned",
    "data": {
        "barcode": "JJBEA000000000000012345",
        "reasonCode": "S11",
        "reasonDescription": "Ontvangt de briefwisseling niet op aangeduid adres",
        "scanned_at": "2024-03-17 12:00:00"
    },
    "type": "status_update"
}
```