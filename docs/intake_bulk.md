# Bulk Intake Request

!!! warning
    This API endpoint is still in development, and only available on our UAT environment.

    As such, this documentation is subject to change in the final version.

## Endpoint and method

| API version | v1                                 |
|:------------|:-----------------------------------|
| Endpoint    | `base_url/api_version/intake/bulk` |
| Method      | `POST`                             |

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |


## Multipart form-data Structure
``` form-data
document[name] => 'string',
document[mime-type] => 'mime-type',
document[contents] => file,
document[multiplex] => boolean (default 1),

address[coordinates][startX] => integer,
address[coordinates][startY] => integer,
address[coordinates][width] => integer,
address[coordinates][height] => integer,

splitString[coordinates][startX] => integer,
splitString[coordinates][startY] => integer,
splitString[coordinates][width] => integer,
splitString[coordinates][height] => integer,
splitString[text] => string,

attachments[0][name] => 'string',
attachments[0][mime-type] => 'mime-type',
attachments[0][contents] => file,

attachments[1][name] => ...,

background[name] => 'string',
background[mime-type] => 'mime-type',
background[contents] => file,

envelope[name] => 'string',
envelope[mime-type] => 'mime-type',
envelope[contents] => file,

postalService[registered] => boolean,
postalService[prior] => boolean,
postalService[nonPrior] => boolean,

correlation[costId] => 'string,optional',
correlation[lang] => 'required, nl,en or fr',
correlation[returnAddress] => 'optional string, single line return address',
```

## Multipart Form Data Values
### Document (required)
Each Bulk Intake request must contain exactly one main document. The document is a PDF file that contains 1 or more recipients.

| Key                  | Description                                                             | Required | Type        | Default |
|----------------------|-------------------------------------------------------------------------|----------|-------------|---------|
| document\[name]      | Name of the original document, including extension                      | YES      | string      |         |
| document\[mime-type] | MIME type of the document. Currently only "application/pdf" is accepted | YES      | string      |         |
| document\[contents]  | Document PDF file                                                       | YES      | file/binary |         |
| document\[multiplex] | Print setting of the document. 0 = recto, 1 = recto/verso               | NO       | boolean     | 1       |

### Address (required)
Each Bulk Intake request must contain exactly one address location.
Coordinates must be specified in points (pts).
An address must be present on each page that designates a new recipient, along with a split string (see below).

| Key                           | Description                                  | Required | Type    | Default |
|-------------------------------|----------------------------------------------|----------|---------|---------|
| address\[coordinates][startX] | Horizontal starting point of the address box | YES      | integer |         |
| address\[coordinates][startY] | Vertical starting point of the address box   | YES      | integer |         |
| address\[coordinates][width]  | Width of the address box                     | YES      | integer |         |
| address\[coordinates][height] | Height of the address box                    | YES      | integer |         |

### Split string (required)
Each Bulk Intake request must contain exactly one split string location and text.
Coordinates must be specified in points (pts).
The text in splitString[text] must appear once, on each page that designates a new recipient.

| Key                               | Description                                                | Required | Type    | Default |
|-----------------------------------|------------------------------------------------------------|----------|---------|---------|
| splitString\[coordinates][startX] | Horizontal starting point of the split string location     | YES      | integer |         |
| splitString\[coordinates][startY] | Vertical starting point of the split string location       | YES      | integer |         |
| splitString\[coordinates][width]  | Width of the split string location                         | YES      | integer |         |
| splitString\[coordinates][height] | Height of the split string location                        | YES      | integer |         |
| splitString\[text]                | The unique text per recipient to split the PDF document on |          |         |         |

### Attachments (optional)
Each Bulk Intake request may contain multiple attachments, using a consecutive numerical index (see JSON structure).

| Key                         | Description                                                               | Required                 | Type        | Default |
|-----------------------------|---------------------------------------------------------------------------|--------------------------|-------------|---------|
| attachments\[n]\[name]      | Name of the attachment                                                    | YES, if using attachment | string      |         |
| attachments\[n]\[mime-type] | MIME type of the attachment. Currently only "application/pdf" is accepted | YES                      | string      |         |
| attachments\[n]\[contents]  | Attachment PDF file                                                       | YES                      | file/binary |         |
| attachments\[n]\[multiplex] | Print setting of the attachment. 0 = recto, 1 = recto/verso               | NO                       | boolean     | 1       |

### Background (optional)
Each Bulk Intake request may contain exactly one background. The background will only be printed on the first page of the main document.

| Key                    | Description                                                               | Required                 | Type        | Default |
|------------------------|---------------------------------------------------------------------------|--------------------------|-------------|---------|
| background\[name]      | Name of the background                                                    | YES, if using background | string      |         |
| background\[mime-type] | MIME type of the background. Currently only "application/pdf" is accepted | YES                      | string      |         |
| background\[contents]  | Background PDF file                                                       | YES                      | file/binary |         |

### Envelope (optional)
Each Bulk Intake request may contain exactly one envelope.

| Key                  | Description                                                             | Required               | Type        | Default |
|----------------------|-------------------------------------------------------------------------|------------------------|-------------|---------|
| envelope\[name]      | Name of the envelope                                                    | YES, if using envelope | string      |         |
| envelope\[mime-type] | MIME type of the envelope. Currently only "application/pdf" is accepted | YES                    | string      |         |
| envelope\[contents]  | Envelope PDF file                                                       | YES                    | file/binary |         |

### Carrier (optional) for C4 envelopes
Each Bulk Intake request may contain exactly one carrier. An address carrier is only used when sending C4 envelopes, which use a separate sheet of paper to print the address on.

| Key                 | Description                                                            | Required              | Type        | Default |
|---------------------|------------------------------------------------------------------------|-----------------------|-------------|---------|
| carrier\[name]      | Name of the address carrier                                            | YES, if using carrier | string      |         |
| carrier\[mime-type] | MIME type of the carrier. Currently only "application/pdf" is accepted | YES                   | string      |         |
| carrier\[contents]  | Address carrier PDF file                                               | YES                   | file/binary |         |

### Correlation (required)
Each Bulk Intake request may contain additional 

| Key                         | Description                                        | Required | Type   | Default                                        |
|-----------------------------|----------------------------------------------------|----------|--------|------------------------------------------------|
| correlation\[costId]        | Cost ID, for internal reference                    | NO       | string |                                                |
| correlation\[lang]          | Document's language, must be one of nl, fr, en, de | YES      | string |                                                |
| correlation\[returnAddress] | Return address in case of undeliverable mail       | NO       | string | Symeta Hybrid, Interleuvenlaan 50, 3001 Leuven |

### Postal service
Each Bulk Intake request may contain additional postal service instructions

| Key                        | Description                               | Required | Type    | Default |
|----------------------------|-------------------------------------------|----------|---------|---------|
| postalService\[registered] | Registered shipment with tracking (bpost) | NO       | boolean | 0       |
| postalService\[prior]      | Prior shipment                            | NO       | boolean | 0       |
| postalService\[nonPrior]   | Non-prior (standard) shipment             | NO       | boolean | 1       |

### Hook (required)
Each Bulk Intake request must specify a webhook URL and method to send status 
updates and possible address validation errors to

| Key           | Description                                                        | Required                   | Type   | Default |
|---------------|--------------------------------------------------------------------|----------------------------|--------|---------|
| hook\[uri]    | URI where the webhook should be sent to                            | YES, if webhook is desired | string |         |
| hook\[method] | HTTP method to be used for the webhook, must be either GET or POST | YES, if webhook is desired | string |         |

For more detailed information on the webhook functionality, refer to the [Webhooks](webhooks.md) page.

## Multipart Intake Request Response

### Success
A successful request will receive the following JSON response (with an HTTP 200 status code):

```json
{
    "bulk_intake_id": 145,
    "status": "received"
}
```
This indicates the request was well-formed. The file will then be further processed, and updates will be sent via webhooks.

### Webhooks

#### Document split result
After a bulk PDF document has been split successfully, we will send back a webhook detailing how many recipients were parsed, and on which pages they were found.

The cost of the entire bulk will also be included, in a `float` data type.

```json
{
  "bulk_intake_id": 145,
  "cost": "5.000",
  "created_intakes": [
    {
      "intake_id": 1948,
      "first_page": "1",
      "last_page": "2"
    }
  ],
  "total_recipients": 1
}
```

#### Address validation errors
If any of the parsed addresses in a bulk intake fail validation, these will be sent in a bundled webhook.
This webhook will contain the following:

```json
{
  "bulk_intake_id": 145,
  "status": "rejected",
  "data": [
    {
      "first_page": 2,
      "last_page": 2,
      "errors": [
        "houses"
      ],
      "parsed_text": "Kerkstraat 10\n9800 Deinze"
    },
    {
      "first_page": 3,
      "last_page": 3,
      "errors": [
        "roads",
        "postcode",
        "road"
      ],
      "parsed_text": "John Doe\n Deinze"
    },
    {
      "first_page": 4,
      "last_page": 4,
      "errors": [
        "city",
        "postcode"
      ],
      "parsed_text": "John Doe\nKerkstraat 25"
    }
  ],
  "type": "address_validation_errors"
}
```
In the `data` property, we will find all the addresses that failed. The webhook will always include which component of the address caused the failure.
`parsedText` will always contain the raw text that was parsed on the document.

Note: for documents that fail address validation, we will **not** persist the Intake model to the database. These are only kept in memory at runtime to provide feedback.

### Possible error codes

| HTTP code | Description          | Reason                                                                                                                 |
|-----------|----------------------|------------------------------------------------------------------------------------------------------------------------|
| 422       | Unprocessable Entity | Request is invalid - either missing or incorrectly formatted parameters.<br/>Refer to validation errors for specifics. |
| 413       | Entity Too Large     | Request size exceeds limit                                                                                             |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal                                                         |

### Possible error webhooks

#### PDF document is not readable

If the transferred PDF document cannot be read, this error will be returned via webhook.

This can occur if the PDF file has been flattened/rasterized, or if the file format has been corrupted.

#### Amount of split documents does not match count_recipients

If the indicated expected amount of recipients does not match the amount of documents we have after splitting using the identifier, this error will be returned via webhook.

```json
{
    "status": "error",
    "data": {
        "count_recipients": 2,
        "split_count": 6
    },
    "code": "indicatedCountNotEqualsSplitCount",
    "error_message": "Indicated number of recipients does not match the number of intakes. This usually indicates an issue with the splitting identifier."
}
```