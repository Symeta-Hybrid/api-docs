# Multipart Intake Request
## Multipart vs JSON
Using a Multipart Request allows to upload larger documents to the API. However, we do advise to use the JSON Intake Request as much as possible and avoid large filesizes.
With the Multipart Request, the 4MB filesize limit does not apply. For the most part, the requests are very similar.

## Endpoint and method

| API version | v1                                      |
|:------------|:----------------------------------------|
| Endpoint    | `base_url/api_version/intake/multipart` |
| Method      | `POST`                                  |

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

address[firstname] => 'string',
address[lastname] => 'string',
address[fullname] => 'string',
address[street] => 'string',
address[houseNumber] => integer,
address[houseNumberAlpha] => 'string',
address[box] => 'string',
address[unstructuredStreetNumberBox] => 'string',
address[zip] => 'string',
address[municipality] => 'string',
address[countryIso2] => 'string max 2',
address[company] => 'string'

postalService[registered] => boolean,
postalService[prior] => boolean,
postalService[nonPrior] => boolean,

correlation[costId] => 'string,optional',
correlation[lang] => 'required, nl,en or fr',
correlation[returnAddress] => 'optional string, single line return address',

requestType => 'multipart'
```

## Multipart Form Data Values
### Request type (required)
Add parameter requestType and set value as 'multipart' to indicate it is a multipart form.

| Key         | Description                       | Required          | Type   | Default |
|-------------|-----------------------------------|-------------------|--------|---------|
| requestType | Type of Intake request being sent | YES, if multipart | string |         |


### Document (required)
Each Multipart Intake request must contain exactly one main document.

| Key                  | Description                                                             | Required | Type        | Default |
|----------------------|-------------------------------------------------------------------------|----------|-------------|---------|
| document\[name]      | Name of the original document, including extension                      | YES      | string      |         |
| document\[mime-type] | MIME type of the document. Currently only "application/pdf" is accepted | YES      | string      |         |
| document\[contents]  | Document PDF file                                                       | YES      | file/binary |         |
| document\[multiplex] | Print setting of the document. 0 = recto, 1 = recto/verso               | NO       | boolean     | 1       |

### Attachments (optional)
Each Multipart Intake request may contain multiple attachments, using a consecutive numerical index (see JSON structure).

| Key                         | Description                                                               | Required                 | Type        | Default |
|-----------------------------|---------------------------------------------------------------------------|--------------------------|-------------|---------|
| attachments\[n]\[name]      | Name of the attachment                                                    | YES, if using attachment | string      |         |
| attachments\[n]\[mime-type] | MIME type of the attachment. Currently only "application/pdf" is accepted | YES                      | string      |         |
| attachments\[n]\[contents]  | Attachment PDF file                                                       | YES                      | file/binary |         |
| attachments\[n]\[multiplex] | Print setting of the attachment. 0 = recto, 1 = recto/verso               | NO                       | boolean     | 1       |

### Background (optional)
Each Multipart Intake request may contain exactly 1 background. The background will only be printed on the first page of the main document.

| Key                    | Description                                                               | Required                 | Type        | Default |
|------------------------|---------------------------------------------------------------------------|--------------------------|-------------|---------|
| background\[name]      | Name of the background                                                    | YES, if using background | string      |         |
| background\[mime-type] | MIME type of the background. Currently only "application/pdf" is accepted | YES                      | string      |         |
| background\[contents]  | Background PDF file                                                       | YES                      | file/binary |         |

### Envelope (optional)
Each Multipart Intake request may contain exactly one envelope.

| Key                  | Description                                                             | Required               | Type        | Default |
|----------------------|-------------------------------------------------------------------------|------------------------|-------------|---------|
| envelope\[name]      | Name of the envelope                                                    | YES, if using envelope | string      |         |
| envelope\[mime-type] | MIME type of the envelope. Currently only "application/pdf" is accepted | YES                    | string      |         |
| envelope\[contents]  | Envelope PDF file                                                       | YES                    | file/binary |         |

### Carrier (optional) for C4 envelopes
Each Multipart Intake request may contain exactly one carrier. An address carrier is only used when sending C4 envelopes, which use a separate sheet of paper to print the address on.

| Key                 | Description                                                            | Required              | Type        | Default |
|---------------------|------------------------------------------------------------------------|-----------------------|-------------|---------|
| carrier\[name]      | Name of the address carrier                                            | YES, if using carrier | string      |         |
| carrier\[mime-type] | MIME type of the carrier. Currently only "application/pdf" is accepted | YES                   | string      |         |
| carrier\[contents]  | Address carrier PDF file                                               | YES                   | file/binary |         |

### Address (required)
Each Multipart Intake request must contain exactly one recipient with the required address parts.

| Key                                   | Description                                                             | Required                                                                    | Type   | Default |
|---------------------------------------|-------------------------------------------------------------------------|-----------------------------------------------------------------------------|--------|---------|
| address\[firstname]                   | Recipient's first name                                                  | YES, if address\[fullname] and address\[company] absent                     | string |         |
| address\[lastname]                    | Recipient's last name                                                   | YES, if address\[fullname] and address\[company] absent                     | string |         |
| address\[fullname]                    | Recipient's full name                                                   | YES, if address\[firstname], address.lastname and address\[company] absent  | string |         |
| address\[company]                     | Company name                                                            | YES, if address\[firstname], address.lastname and address\[fullname] absent | string |         |
| address\[street]                      | Street name without number                                              | YES, if address\[unstructuredStreetNumberBox] absent                        | string |         |
| address\[houseNumber]                 | House number                                                            | YES, if address\[unstructuredStreetNumberBox] absent                        | string |         |
| address\[houseNumberAlpha]            | House number alphanumeric, will be placed after houseNumber on envelope | NO                                                                          | string |         |
| address\[box]                         | Box number                                                              | NO                                                                          | string |         |
| address\[unstructuredStreetNumberBox] | Unstructured street, house number and box                               | YES, if address\[street] and address\[houseNumber] absent                   | string |         |
| address\[zip]                         | Zip code                                                                | YES                                                                         | string |         |
| address\[municipality]                | Municipality                                                            | YES                                                                         | string |         |
| address\[countryIso2]                 | Country code, ISO 3611-2 format (2 characters)                          | YES                                                                         | string |         |

### Correlation (required)

| Key                         | Description                                        | Required | Type   | Default                                        |
|-----------------------------|----------------------------------------------------|----------|--------|------------------------------------------------|
| correlation\[costId]        | Cost ID, for internal reference                    | NO       | string |                                                |
| correlation\[lang]          | Document's language, must be one of nl, fr, en, de | YES      | string |                                                |
| correlation\[returnAddress] | Return address in case of undeliverable mail       | NO       | string | Symeta Hybrid, Interleuvenlaan 50, 3001 Leuven |

### Postal service (optional, default bpost non-prior)
Each Intake request may contain additional postal service instructions

| Key                        | Description                               | Required | Type    | Default |
|----------------------------|-------------------------------------------|----------|---------|---------|
| postalService\[registered] | Registered shipment with tracking (bpost) | NO       | boolean | 0       |
| postalService\[prior]      | Prior shipment                            | NO       | boolean | 0       |
| postalService\[nonPrior]   | Non-prior (standard) shipment             | NO       | boolean | 1       |

### Hook (optional)
Each Intake request may specify a webhook URL and method to send status updates to

| Key           | Description                                                        | Required                   | Type   | Default |
|---------------|--------------------------------------------------------------------|----------------------------|--------|---------|
| hook\[uri]    | URI where the webhook should be sent to                            | YES, if webhook is desired | string |         |
| hook\[method] | HTTP method to be used for the webhook, must be either GET or POST | YES, if webhook is desired | string |         |

For more detailed information on the webhook functionality, refer to the [Webhooks](webhooks.md) page.

## Multipart Intake Request Response

### Success

| Key              | Value                                    |
|------------------|------------------------------------------|
| HTTP status code | 202 ACCEPTED                             |
| JSON body        | `{ "intake": integer, "cost": integer }` |

### Possible error codes

| HTTP code | Description          | Reason                                                                  |
|-----------|----------------------|-------------------------------------------------------------------------|
| 422       | Unprocessable Entity | Request is invalid - either missing or incorrectly formatted parameters |
| 413       | Entity Too Large     | Request size exceeds limit                                              |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal          |
