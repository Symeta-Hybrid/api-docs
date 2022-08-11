# JSON Intake request

## Endpoint and method

| API version | v1                          |
|:------------|:----------------------------|
| Endpoint    | base_url/api_version/intake |
| Method      | POST                        |

## Request header

| Header        | Value            |
|---------------|------------------|
| Accept        | application/json |
| Authorization | Bearer [token]   |

## Request body and document format
All metadata and documents must be sent in a JSON format. The main document's content must be sent in a base-64 encoded string.

Examples for achieving the correct format:

### PHP
``` php
<?php
$doc = base64_encode(file_get_contents(‘document.pdf’));
```
### C\#
``` c#
String GetBase64FileContent(string fileName, string path)
{
    var file = Directory.GetFiles(path, $"{fileName}.*").FirstOrDefault();
    if (file != null) {
        Byte[] bytes = File.ReadAllBytes(file);
        String base64String = Convert.ToBase64String(bytes);
        return base64String;
    }

    return null;
}
```

!!! attention
    The maximum size of a JSON request is 4MB! If your payload exceeds this limit, 
    the server will return an HTTP error 413 (entity/payload too large). If you want to send larger documents, please use a Multipart Intake request!

## JSON Request Structure
``` json
{
    "document": {
        "name": "string",
        "mime-type": "mime-type",
        "contents": "base64encoded document content",
        "multiplex": boolean (default 1),
    },
    "attachments": {
        "0": {
            "name" : "string",
            "mime-type": "mime-type",
            "contents": "base64encoded content",
            "multiplex": boolean (default 1)
        },
        "1": {
            ...
        }
    },  
    "background" : {
        "name": "string",
        "mime-type": "mime-type",
        "contents": "base64encoded content",
    },
    "envelope": {
        "name": "string",
        "mime-type": "mime-type",
        "contents": "base64encoded content"
    },
    "address": {
        "firstname": "string",
        "lastname": "string",
        "fullname": "string",
        "street": "string",
        "houseNumber": integer,
        "houseNumberAlpha": "string",
        "box": "string"
        "unstructuredStreetNumberBox": "string",
        "zip": "string",
        "municipality": "string",
        "countryIso2": "string two letters",
        "company": "string",
        "area": "string"
    },
    "correlation": {
        "costId": "optional,string",
        "lang": "string nl,fr,en,de",
        "returnAddress: "string, single line address"
    },
    "postalService": {
        "registered": boolean default 0,
        "prior": boolean default 0,
        "nonPrior": boolean default 0,
    },
    "email": {
        "registered": boolean default 0,
        "email": "string email"
    },
    "hook": {
        "uri": "Full webhook path",
        "method": "POST or GET"
    }
}
```
## JSON Request Values
### Document (required)
Each Intake request must contain exactly one main document.

| Key                | Description                                                             | Required | Type    | Default |
|--------------------|-------------------------------------------------------------------------|----------|---------|---------|
| document.name      | Name of the document.                                                   | YES      | string  |         |
| document.mime-type | MIME type of the document. Currently only "application/pdf" is accepted | YES      | string  |         |
| document.contents  | Base64-encoded contents of the document                                 | YES      | string  |         |
| document.multiplex | Print setting of the document. 0 = recto, 1 = recto/verso               | NO       | boolean | 1       |

### Attachments (optional)
Each Intake request may contain multiple attachments, using a consecutive numerical index (see JSON structure).

| Key                       | Description                                                               | Required                 | Type    | Default |
|---------------------------|---------------------------------------------------------------------------|--------------------------|---------|---------|
| attachments.{n}.name      | Name of the attachment                                                    | YES, if using attachment | string  |         |
| attachments.{n}.mime-type | MIME type of the attachment. Currently only "application/pdf" is accepted | YES                      | string  |         |
| attachments.{n}.contents  | Base64-encoded contents of the attachment                                 | YES                      | string  |         |
| attachments.{n}.multiplex | Print setting of the attachment. 0 = recto, 1 = recto/verso               | NO                       | boolean | 1       |

### Background (optional)
Each Intake request may contain exactly 1 background. The background will only be printed on the first page of the main document.

| Key                  | Description                                                               | Required                 | Type   | Default |
|----------------------|---------------------------------------------------------------------------|--------------------------|--------|---------|
| background.name      | Name of the background                                                    | YES, if using background | string |         |
| background.mime-type | MIME type of the background. Currently only "application/pdf" is accepted | YES                      | string |         |
| background.contents  | Base64-encoded contents of the background                                 | YES                      | string |         |

### Envelope (optional)
Each Intake request may contain exactly one envelope.

| Key                | Description                                                             | Required               | Type   | Default |
|--------------------|-------------------------------------------------------------------------|------------------------|--------|---------|
| envelope.name      | Name of the envelope                                                    | YES, if using envelope | string |         |
| envelope.mime-type | MIME type of the envelope. Currently only "application/pdf" is accepted | YES                    | string |         |
| envelope.contents  | Base64-encoded contents of the envelope                                 | YES                    | string |         |

### Address (required)
Each Intake request must contain exactly one recipient with the required address parts.

| Key                                 | Description                                                             | Required                                                                | Type   | Default |
|-------------------------------------|-------------------------------------------------------------------------|-------------------------------------------------------------------------|--------|---------|
| address.firstname                   | Recipient's first name                                                  | YES, if address.fullname and address.company absent                     | string |         |
| address.lastname                    | Recipient's last name                                                   | YES, if address.fullname and address.company absent                     | string |         |
| address.fullname                    | Recipient's full name                                                   | YES, if address.firstname, address.lastname and address.company absent  | string |         |
| address.company                     | Company name                                                            | YES, if address.firstname, address.lastname and address.fullname absent | string |         |
| address.street                      | Street name without number                                              | YES, if address.unstructuredStreetNumberBox absent                      | string |         |
| address.houseNumber                 | House number                                                            | YES, if address.unstructuredStreetNumberBox absent                      | string |         |
| address.houseNumberAlpha            | House number alphanumeric, will be placed after houseNumber on envelope | NO                                                                      | string |         |
| address.box                         | Box number                                                              | NO                                                                      | string |         |
| address.unstructuredStreetNumberBox | Unstructured street, house number and box                               | YES, if address.street and address.houseNumber absent                   | string |         |
| address.zip                         | Zip code                                                                | YES                                                                     | string |         |
| address.municipality                | Municipality                                                            | YES                                                                     | string |         |
| address.countryIso2                 | Country code, ISO 3611-2 format (2 characters)                          | YES                                                                     | string |         |
| address.area                        | Area                                                                    | NO                                                                      | string |         |

### Correlation (required)

| Key                       | Description                                        | Required | Type   | Default                                        |
|---------------------------|----------------------------------------------------|----------|--------|------------------------------------------------|
| correlation.costId        | Cost ID, for internal reference                    | NO       | string |                                                |
| correlation.lang          | Document's language, must be one of nl, fr, en, de | YES      | string |                                                |
| correlation.returnAddress | Return address in case of undeliverable mail       | NO       | string | Symeta Hybrid, Interleuvenlaan 50, 3001 Leuven |

### Postal service (optional, default bpost non-prior)
Each Intake request may contain additional postal service instructions

| Key                      | Description                               | Required | Type    | Default |
|--------------------------|-------------------------------------------|----------|---------|---------|
| postalService.registered | Registered shipment with tracking (bpost) | NO       | boolean | 0       |
| postalService.prior      | Prior shipment                            | NO       | boolean | 0       |
| postalService.nonPrior   | Non-prior (standard) shipment             | NO       | boolean | 1       |

### hook (optional)
Each Intake request may specify a webhook URL and method to send status updates to

| Key         | Description                                                        | Required                   | Type   | Default |
|-------------|--------------------------------------------------------------------|----------------------------|--------|---------|
| hook.uri    | URI where the webhook should be sent to                            | YES, if webhook is desired | string |         |
| hook.method | HTTP method to be used for the webhook, must be either GET or POST | YES, if webhook is desired | string |         |


The webhook payload will contain a JSON body with the following keys:

``` json
{
    "intake": {id, integer},
    "status": {intake status, string},
    "data": {currently not implemented, empty string},
}
```
Possible Intake statuses are:
- Processed
- Printing
- Shipped

Payload json 

hook.uri (required if hook is set)

hook.method (required if hook is set)
Webhook
Header 	Signature 	hash_hmac('sha256', 'payloadjson', 'webhook key')
Body 	{"intake":"id","status":"status"}

If signature equals hash_hmac('sha256','json response','webhook key') payload is valid.
## JSON Request Response
Success
Status Code 	202 Accepted
JSON body 	{"intake":integer, "cost":integer} (intake = id of intake request)
Possible error codes
422 	Unprocessable Entity 	Request is invalid.
413 	Entity Too Large 	Request size too big (json max 4mb).
401 	Unauthorized 	Access token invalid.