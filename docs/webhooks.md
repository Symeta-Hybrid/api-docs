# Webhooks
## Introduction
Once a request has been accepted by the API, status updates can be sent to you via webhooks. There are three kinds:

| Kind                                            | Sent for                                                | Configured with                                                          |
|-------------------------------------------------|---------------------------------------------------------|--------------------------------------------------------------------------|
| [Intake status updates](#intake-status-updates) | Every Intake, including eBox ones once they are printed | `hook.uri` / `hook.method` (`hook[uri]` / `hook[method]`)                |
| [eBox status updates](#ebox-status-updates)     | eBox Citizen and eBox Enterprise messages               | `hook_uri` / `hook_method`                                               |
| Bulk Intake updates                             | Bulk Intakes                                            | `hook[uri]` / `hook[method]`, see [Bulk Intake](intake_bulk.md#webhooks) |

Webhooks are only sent when you specify a URI in the request. The method is `POST` unless you specify `GET` (`get`,
`post`, `GET` or `POST` are accepted).

## Webhook key
To make use of webhooks, you will need to generate a webhook key in the web portal first, under *Webhook*. Every webhook
is signed with this key. A request that specifies a webhook URI while your company has no webhook key is rejected with a
422 `No webhook key found, please set one in your dashboard.`

On the same page you can optionally set an API key. When set, it is sent along with every webhook in an `X-API-Key`
header, for receivers that require one.

## Structure
Each webhook is sent with the following headers:

| Header       | Value                                                                |
|--------------|----------------------------------------------------------------------|
| Content-Type | `application/json`                                                   |
| Signature    | HMAC-SHA256 of the JSON payload, using your webhook key, hex encoded |
| X-API-Key    | Your API key, only when one is set in the web portal                 |

With `POST` (the default), the payload is sent as the JSON request body.

With `GET`, the same payload is sent as query string parameters instead (nested values in `key[subkey]=value` notation),
and all values arrive as strings. We recommend `POST`: see the note on verifying `GET` webhooks below.

### Verifying the signature
Calculate the HMAC-SHA256 of the raw request body, using the webhook key as key. If it matches the value of the
`Signature` header, the payload is valid.

For example, in PHP:
``` php
<?php
$jsonPayload = file_get_contents('php://input');
$hash = hash_hmac('sha256', $jsonPayload, $webhookKey);
if (hash_equals($hash, $_SERVER['HTTP_SIGNATURE'])) {
    // Payload contents verified
    // ...
}
```

!!! warning
    Always calculate the signature over the body exactly as it was received. Decoding and re-encoding the JSON first
    will usually give a different result: slashes are sent escaped (`\/`) and non-ASCII characters as `\uXXXX`.

    For `GET` webhooks there is no body: the signature is calculated over the payload encoded as JSON in the same way,
    which you would have to reconstruct from the query string. Use `POST` if you want to verify webhooks.

### Delivery and retries
A webhook counts as delivered when your endpoint responds with an HTTP 2xx status code within 3 seconds. Otherwise it is
retried, up to 3 attempts in total: the second attempt follows after about 10 seconds, the third after about 100
seconds. After the third failed attempt the webhook is dropped and not sent again.

A webhook can arrive more than once (for example when your endpoint processed it but responded too late), and webhooks
are not guaranteed to arrive in the order they were sent. Respond quickly and process the payload afterwards, and make
your processing safe to repeat.

## Intake status updates
These are sent to `hook.uri` with `hook.method`, as specified in the Intake request.

Currently, there are 5 possible Intake status updates:

| Status    | Sent when                                                                                                          |
|-----------|--------------------------------------------------------------------------------------------------------------------|
| Processed | The Intake was received and processed by our server, and is waiting to be printed. Not sent for eBox Intakes       |
| Printing  | The Intake was processed and put on the print queue. For eBox Intakes, only after a [fallback](#fallback) to paper |
| Shipped   | The Intake was printed and handed to bpost for delivery                                                            |
| Rejected  | The Intake was rejected by our print production. Not enabled by default, see below                                 |
| Returned  | The postal item was returned to Symeta Hybrid. Not enabled by default, see below                                   |

!!! info
    The payloads below are exactly as sent. Note that the casing of `status` differs (`rejected` is lowercase), and that
    not every payload has `data` and `type`.

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

### Rejected

This webhook is not automatically enabled. If you wish to have this activated for your intakes, please contact us at
[support@mailitwize.be](mailto:support@mailitwize.be).

``` json
{
    "intake": 158056,
    "status": "rejected",
    "data": {
        "rejected_reason": "string"
    },
    "type": "status_update"
}
```

### Returned

This webhook is not automatically enabled. If you wish to have this activated for your intakes, please contact us at
[support@mailitwize.be](mailto:support@mailitwize.be).

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

## eBox status updates
These are sent to `hook_uri` with `hook_method`, as specified in the [eBox Citizen](intake_citizen_multipart.md) or
[eBox Enterprise](intake_enterprise_multipart.md) request. They report on the eBox message. The paper Intake that comes
with it reports separately, through the [Intake status updates](#intake-status-updates) sent to `hook[uri]`.

Currently, there are 4 possible eBox status updates:

| Status    | Sent when                                                            |
|-----------|----------------------------------------------------------------------|
| offered   | The message was published to the recipient's eBox                    |
| delivered | eBox confirmed the message is available to the recipient             |
| read      | The recipient opened the message                                     |
| fallback  | The message will be sent on paper instead, see [Fallback](#fallback) |

Every eBox status update contains the following keys:

| Key                 | Description                                                                                                            | Always present |
|---------------------|------------------------------------------------------------------------------------------------------------------------|----------------|
| message             | Unique identifier (GUID) of the eBox message: the `message` returned by the eBox request, or the `guid` you sent       | YES            |
| type                | Always `status_update`                                                                                                 | YES            |
| status              | One of `offered`, `delivered`, `read` or `fallback`                                                                    | YES            |
| data                | Details of the status, see the payloads below                                                                          | YES            |
| ebox_status         | The event as reported by eBox itself, when the update was triggered by one                                             | NO             |
| customer_identifier | The customer the message was sent for, when it was sent for one of your [company customers](company_customers.md#ebox) | NO             |

!!! info "Date and time formats"
    `offered_at` is sent in ISO 8601 format, in UTC. `must_be_read_by` is sent as `YYYY-MM-DD HH:MM:SS.mmm`. All other
    timestamps are sent as `YYYY-MM-DD HH:MM:SS` without a timezone offset: Belgian local time (Europe/Brussels), except
    for `expires_at` and the times of events reported by eBox, which keep the timezone eBox reported them in.

The payloads are as follows, for each status:

### Offered
``` json
{
    "message": "8f0c2a6e-2b8e-4a39-9d6f-3c1f2f6e9a10",
    "type": "status_update",
    "status": "offered",
    "data": {
        "expires_at": "2027-10-29 10:14:00",
        "must_be_read_by": "2026-11-28 10:14:00.000",
        "offered_at": "2026-10-29T09:14:00.000000Z"
    }
}
```

| Key             | Description                                                                                                                               |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| expires_at      | The expiration date eBox gave the message                                                                                                 |
| must_be_read_by | Deadline by which the message must be read before it falls back to paper (see [Fallback](#fallback)), or `null` when there is no deadline |
| offered_at      | When the eBox request was received by the API                                                                                             |

### Delivered
For a message sent for one of your company customers, `customer_identifier` is included, as in this example.

``` json
{
    "message": "8f0c2a6e-2b8e-4a39-9d6f-3c1f2f6e9a10",
    "type": "status_update",
    "status": "delivered",
    "data": {
        "delivered_at": "2026-10-29 10:15:00"
    },
    "ebox_status": "newDocument",
    "customer_identifier": "customer-001"
}
```

### Read
``` json
{
    "message": "8f0c2a6e-2b8e-4a39-9d6f-3c1f2f6e9a10",
    "type": "status_update",
    "status": "read",
    "data": {
        "read_at": "2026-10-30 18:02:11"
    },
    "ebox_status": "readDocument"
}
```

### Fallback
When a message cannot be delivered digitally, it falls back to paper: the Intake sent with the eBox request is printed
and posted like any other Intake, and reports through the [Intake status updates](#intake-status-updates) (`Printing`,
`Shipped`, ...) on `hook[uri]`. A message falls back at most once.

``` json
{
    "message": "8f0c2a6e-2b8e-4a39-9d6f-3c1f2f6e9a10",
    "type": "status_update",
    "status": "fallback",
    "data": {
        "fallback_at": "2026-11-28 10:20:00",
        "fallback_reason": "messageExpired"
    }
}
```

| fallback_reason          | ebox_status         | Reason                                                                                       |
|--------------------------|---------------------|----------------------------------------------------------------------------------------------|
| failedToSend             | (absent)            | The message could not be published to eBox                                                   |
| noDigitalCitizen         | noDigitalCitizen    | eBox Citizen: eBox reports the recipient cannot receive the message digitally                |
| noDigitalConsumer        | noDigitalConsumer   | eBox Citizen: eBox reports the recipient cannot receive the message digitally                |
| newDocumentOnPaper       | newDocumentOnPaper  | eBox Citizen: eBox reports the message is to be delivered on paper                           |
| noDigitalEnterprise      | noDigitalEnterprise | eBox Enterprise: eBox accepted the message, but the recipient enterprise has no digital eBox |
| enterpriseNotFullDigital | (absent)            | eBox Enterprise: the recipient enterprise does not receive its mail exclusively through eBox |
| messageExpired           | (absent)            | The message was delivered but not read before `must_be_read_by`                              |

The read deadline (`must_be_read_by`) is set by `fallback_days` in the eBox request, or else by your eBox configuration
(see [Read deadline](intake_citizen_multipart.md#read-deadline-optional)). When there is no deadline, a message that is
delivered but never read does not fall back.

### Message flow

#### eBox Citizen

1. `offered` once the message is published.
2. `delivered` once eBox reports the message as available (`newDocument`).
3. `read` once the recipient opens it (`readDocument`). The paper Intake is then discarded and will not be printed.

#### eBox Enterprise

1. `offered` once the message is published, immediately followed by `delivered`: for eBox Enterprise a published message
   is available to the recipient right away.
2. If the recipient enterprise does not receive its mail exclusively through eBox, a `fallback` with reason
   `enterpriseNotFullDigital` follows. The message stays in the recipient's eBox, and is also sent on paper.
3. `read` once the recipient opens it (`readDocument`). If no fallback happened, the paper Intake is discarded and will
   not be printed.

A `fallback` is not necessarily the last update: eBox may still report a message as delivered or read after it has
fallen back.
