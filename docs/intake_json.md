# JSON Intake request

!!! tip
    We recommend the [Multipart Intake Request](intake_multipart.md) for new integrations. It avoids base64 encoding and
    is not bound by the 4MB request size limit.

## Endpoint and method

| API version | v1                            |
|:------------|:------------------------------|
| Endpoint    | `base_url/api_version/intake` |
| Method      | `POST`                        |

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |

## Request body and document format
All metadata and documents must be sent in a JSON format. PDF content must be sent in a base-64 encoded string.

Examples for achieving the correct format:

!!! example
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

!!! warning
    The maximum size of a JSON request is 4MB! If your payload exceeds this limit, the server will return an HTTP error
    413 (entity/payload too large). If you want to send larger documents, use the [Multipart Intake
    request](intake_multipart.md).

## JSON Request Structure
``` json
{
    "document": {
        "name": "string",
        "mime-type": "mime-type",
        "contents": "base64encoded document content",
        "multiplex": boolean (default 1)
    },
    "attachments": {
        "0": {
            "name" : "string",
            "mime-type": "mime-type",
            "contents": "base64encoded content"
        },
        "1": {
            ...
        }
    },
    "background" : {
        "name": "string",
        "mime-type": "mime-type",
        "contents": "base64encoded content"
    },
    "envelope": {
        "name": "string",
        "mime-type": "mime-type",
        "contents": "base64encoded content"
    },
    "carrier": {
        "name": "string",
        "mime-type": "mime-type",
        "contents": "base64encoded content"
    },
    "address": {
        "firstname": "string",
        "lastname": "string",
        "fullname": "string",
        "street": "string",
        "houseNumber": "string",
        "houseNumberAlpha": "string",
        "box": "string",
        "unstructuredStreetNumberBox": "string",
        "zip": "string",
        "municipality": "string",
        "countryIso2": "string two letters",
        "company": "string"
    },
    "postalService": {
        "registered": boolean default 0,
        "prior": boolean default 0,
        "nonPrior": boolean default 1
    },
    "hook": {
        "uri": "Full webhook path",
        "method": "POST or GET"
    },
    "correlation": {
        "costId": "optional,string",
        "customerIdentifier": "optional,string",
        "lang": "string nl,fr,en,de",
        "returnAddress": "string, single line address"
    },
    "duplicityCheck": "string 'true', optional"
}
```
## JSON Request Values
### Document (required)
Each Intake request must contain exactly one main document.

| Key                | Description                                                             | Required | Type    | Default |
|--------------------|-------------------------------------------------------------------------|----------|---------|---------|
| document.name      | Name of the document                                                    | YES      | string  |         |
| document.mime-type | MIME type of the document. Currently only "application/pdf" is accepted | YES      | string  |         |
| document.contents  | Base64-encoded contents of the document                                 | YES      | string  |         |
| document.multiplex | Print setting of the document. 0 = recto, 1 = recto/verso               | NO       | boolean | 1       |

### Attachments (optional)
Each Intake request may contain multiple attachments, using a consecutive numerical index (see JSON structure).

| Key                       | Description                                                               | Required                 | Type   | Default |
|---------------------------|---------------------------------------------------------------------------|--------------------------|--------|---------|
| attachments.{n}.name      | Name of the attachment                                                    | YES, if using attachment | string |         |
| attachments.{n}.mime-type | MIME type of the attachment. Currently only "application/pdf" is accepted | YES                      | string |         |
| attachments.{n}.contents  | Base64-encoded contents of the attachment                                 | YES                      | string |         |

Attachments are always printed recto/verso.

### Background (optional)
Each Intake request may contain exactly 1 background. The background will only be printed on the first page of the main
document.

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

### Carrier (optional) for C4 envelopes
Each Intake request may contain exactly one carrier. An address carrier is only used when sending C4 envelopes, which
use a separate sheet of paper to print the address on.

| Key               | Description                                                            | Required              | Type   | Default |
|-------------------|------------------------------------------------------------------------|-----------------------|--------|---------|
| carrier.name      | Name of the address carrier                                            | YES, if using carrier | string |         |
| carrier.mime-type | MIME type of the carrier. Currently only "application/pdf" is accepted | YES                   | string |         |
| carrier.contents  | Base64-encoded contents of the carrier                                 | YES                   | string |         |


### PDF requirements
Every file must be a valid PDF. The JSON Intake request also checks the page size of each file, and returns a 422 when
a file does not meet these requirements:

| File                  | Maximum page size           | Other requirements                |
|-----------------------|-----------------------------|-----------------------------------|
| document, attachments | 600 × 847 pt (about A4)     |                                   |
| background, carrier   | 595 × 842 pt (A4)           | exactly 1 page, PDF version ≥ 1.3 |
| envelope              | 621 × 295 pt (219 × 104 mm) |                                   |

### Address (required)
Each Intake request must contain exactly one recipient with the required address parts.

| Key                                 | Description                                                            | Required                                                                | Type   | Default |
|-------------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------------|--------|---------|
| address.firstname                   | Recipient's first name                                                 | YES, if address.fullname and address.company absent                     | string |         |
| address.lastname                    | Recipient's last name                                                  | YES, if address.fullname and address.company absent                     | string |         |
| address.fullname                    | Recipient's full name                                                  | YES, if address.firstname, address.lastname and address.company absent  | string |         |
| address.company                     | Company name                                                           | YES, if address.firstname, address.lastname and address.fullname absent | string |         |
| address.street                      | Street name without number                                             | YES, if address.unstructuredStreetNumberBox absent                      | string |         |
| address.houseNumber                 | House number                                                           | YES, if address.unstructuredStreetNumberBox absent                      | string |         |
| address.houseNumberAlpha            | House number suffix, letters and digits only, placed after houseNumber | NO                                                                      | string |         |
| address.box                         | Box number                                                             | NO                                                                      | string |         |
| address.unstructuredStreetNumberBox | Unstructured street, house number and box                              | YES, if address.street and address.houseNumber absent                   | string |         |
| address.zip                         | Zip code                                                               | YES                                                                     | string |         |
| address.municipality                | Municipality                                                           | YES                                                                     | string |         |
| address.countryIso2                 | Country code, ISO 3166-1 alpha-2 format (2 letters)                    | YES                                                                     | string |         |

The recipient's name (`address.fullname`, or `address.firstname` and `address.lastname` joined by a space) may be at
most 50 bytes long. An accented character such as `é` counts as 2 bytes. A longer name returns a 422 with key
`addresseeTooLong`.

For a Belgian address (`address.countryIso2` is `BE`), `address.zip` must be a zip code that exists in Belgium: 4
digits, otherwise a 422 is returned (`A Belgian zip code must only contain numbers.`, `A Belgian zip code must be 4
characters exactly.` or `This zip code does not exist in Belgium.`).

### Postal service (optional, default bpost non-prior)
Each Intake request may contain additional postal service instructions

| Key                      | Description                               | Required | Type    | Default |
|--------------------------|-------------------------------------------|----------|---------|---------|
| postalService.registered | Registered shipment with tracking (bpost) | NO       | boolean | 0       |
| postalService.prior      | Prior shipment                            | NO       | boolean | 0       |
| postalService.nonPrior   | Non-prior (standard) shipment             | NO       | boolean | 1       |

Non-prior is used whenever prior is not set: `postalService.nonPrior` is accepted but has no effect.

### Hook (optional)
Each Intake request may specify a webhook URL and method to send status updates to

| Key         | Description                                                                                                                                                                                               | Required                   | Type   | Default |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------|--------|---------|
| hook.uri    | URI where the webhook should be sent to. Must be a valid URL. Requires a webhook key (see [Webhooks](webhooks.md)), otherwise a 422 `No webhook key found, please set one in your dashboard.` is returned | YES, if webhook is desired | string |         |
| hook.method | HTTP method to be used for the webhook: `get`, `post`, `GET` or `POST`                                                                                                                                    | NO                         | string | POST    |

For more detailed information on the webhook functionality, refer to the [Webhooks](webhooks.md) page.

### Correlation (required)

| Key                            | Description                                                                       | Required | Type   | Default                                        |
|--------------------------------|-----------------------------------------------------------------------------------|----------|--------|------------------------------------------------|
| correlation.costId             | Cost ID, for internal reference                                                   | NO       | string |                                                |
| correlation.customerIdentifier | Identifier of the [company customer](company_customers.md) the Intake is sent for | NO       | string |                                                |
| correlation.lang               | Document's language, must be one of nl, fr, en, de                                | YES      | string |                                                |
| correlation.returnAddress      | Return address in case of undeliverable mail                                      | NO       | string | Symeta Hybrid, Interleuvenlaan 50, 3001 Leuven |

### Duplicity check (optional)
Each Intake request may specify whether a duplicity check should be run.

The check will determine if another Intake exists with the same document name. If attachments are present, the names of
these will also be considered in the check.

If any duplicates are found, a validation error will be returned with a message containing the Intake ID('s) of the
duplicate(s).

| Key            | Description                                     | Required | Type          | Default |
|----------------|-------------------------------------------------|----------|---------------|---------|
| duplicityCheck | Whether or not a duplicity check should be done | NO       | string 'true' |         |


## JSON Request Response

### Success

| Key              | Value                                                      |
|------------------|------------------------------------------------------------|
| HTTP status code | 202 ACCEPTED                                               |
| JSON body        | `{ "intake": integer, "cost": string, "pricing": object }` |

`cost` is the total cost of the Intake, as a string with 4 decimals (e.g. `"1.2345"`). `pricing` is the breakdown it is
calculated from:

| Key                  | Type   | Description                                       |
|----------------------|--------|---------------------------------------------------|
| recto_pages          | number | Printed sheet fronts                              |
| verso_pages          | number | Printed sheet backs                               |
| total_pages          | number | recto_pages + verso_pages                         |
| total_sheets         | number | Sheets of paper                                   |
| envelope_type        | string | `US`, `C5` or `C4`                                |
| recto_cost           | number | Cost of the fronts                                |
| verso_cost           | number | Cost of the backs                                 |
| print_cost           | number | recto_cost + verso_cost                           |
| send_cost            | number | Postage                                           |
| prior_cost           | number | Prior surcharge (registered mail is always prior) |
| registered_cost      | number | Registered surcharge                              |
| envelope_cost        | number | Custom envelope                                   |
| address_carrier_cost | number | Address carrier (C4)                              |
| international        | string | `yes` or `no`                                     |
| total_cost           | number | Total, `cost` before rounding                     |

Partner companies also receive `partner_pricing`, the same breakdown at partner rates.

### Possible error codes

| HTTP code | Description          | Reason                                                                  |
|-----------|----------------------|-------------------------------------------------------------------------|
| 422       | Unprocessable Entity | Request is invalid - either missing or incorrectly formatted parameters |
| 413       | Entity Too Large     | Request size exceeds 4MB limit - use the Multipart method instead       |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal          |
