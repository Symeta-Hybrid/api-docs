# Multipart Intake Request
## Multipart vs JSON
The Multipart Intake Request is the recommended way to submit intakes. Files are uploaded as-is, without base64
encoding, and the 4MB request size limit of the JSON Intake Request does not apply. For the most part, the requests are
very similar.

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

carrier[name] => 'string',
carrier[mime-type] => 'mime-type',
carrier[contents] => file,

address[firstname] => 'string',
address[lastname] => 'string',
address[fullname] => 'string',
address[street] => 'string',
address[houseNumber] => 'string',
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

hook[uri] => 'string',
hook[method] => 'string',

correlation[costId] => 'string,optional',
correlation[customerIdentifier] => 'string,optional',
correlation[lang] => 'required, nl, fr, de or en',
correlation[returnAddress] => 'optional string, single line return address',

duplicityCheck => "string 'true', optional"
```

## Multipart Form Data Values
### Document (required)
Each Multipart Intake request must contain exactly one main document.

| Key                  | Description                                                             | Required | Type        | Default |
|----------------------|-------------------------------------------------------------------------|----------|-------------|---------|
| document\[name]      | Name of the original document, including extension                      | YES      | string      |         |
| document\[mime-type] | MIME type of the document. Currently only "application/pdf" is accepted | YES      | string      |         |
| document\[contents]  | Document PDF file                                                       | YES      | file/binary |         |
| document\[multiplex] | Print setting of the document. 0 = recto, 1 = recto/verso               | NO       | boolean     | 1       |

### Attachments (optional)
Each Multipart Intake request may contain multiple attachments, using a consecutive numerical index (see JSON
structure).

| Key                         | Description                                                               | Required                 | Type        | Default |
|-----------------------------|---------------------------------------------------------------------------|--------------------------|-------------|---------|
| attachments\[n]\[name]      | Name of the attachment                                                    | YES, if using attachment | string      |         |
| attachments\[n]\[mime-type] | MIME type of the attachment. Currently only "application/pdf" is accepted | YES                      | string      |         |
| attachments\[n]\[contents]  | Attachment PDF file                                                       | YES                      | file/binary |         |

Attachments are always printed recto/verso.

### Background (optional)
Each Multipart Intake request may contain exactly 1 background. The background will only be printed on the first page of
the main document.

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
Each Multipart Intake request may contain exactly one carrier. An address carrier is only used when sending C4
envelopes, which use a separate sheet of paper to print the address on.

| Key                 | Description                                                            | Required              | Type        | Default |
|---------------------|------------------------------------------------------------------------|-----------------------|-------------|---------|
| carrier\[name]      | Name of the address carrier                                            | YES, if using carrier | string      |         |
| carrier\[mime-type] | MIME type of the carrier. Currently only "application/pdf" is accepted | YES                   | string      |         |
| carrier\[contents]  | Address carrier PDF file                                               | YES                   | file/binary |         |

### PDF requirements
Every file must be a valid PDF, and the envelope may be at most 219 × 104 mm (621 × 295 pt). Otherwise a 422 is
returned. Page sizes of the other files are not checked by the Multipart Intake request; see the [JSON Intake
request](intake_json.md#pdf-requirements) for the sizes we print.

### Address (required)
Each Multipart Intake request must contain exactly one recipient with the required address parts.

| Key                                   | Description                                                            | Required                                                                      | Type   | Default |
|---------------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------------------|--------|---------|
| address\[firstname]                   | Recipient's first name                                                 | YES, if address\[fullname] and address\[company] absent                       | string |         |
| address\[lastname]                    | Recipient's last name                                                  | YES, if address\[fullname] and address\[company] absent                       | string |         |
| address\[fullname]                    | Recipient's full name                                                  | YES, if address\[firstname], address\[lastname] and address\[company] absent  | string |         |
| address\[company]                     | Company name                                                           | YES, if address\[firstname], address\[lastname] and address\[fullname] absent | string |         |
| address\[street]                      | Street name without number                                             | YES, if address\[unstructuredStreetNumberBox] absent                          | string |         |
| address\[houseNumber]                 | House number                                                           | YES, if address\[unstructuredStreetNumberBox] absent                          | string |         |
| address\[houseNumberAlpha]            | House number suffix, letters and digits only, placed after houseNumber | NO                                                                            | string |         |
| address\[box]                         | Box number                                                             | NO                                                                            | string |         |
| address\[unstructuredStreetNumberBox] | Unstructured street, house number and box                              | YES, if address\[street] and address\[houseNumber] absent                     | string |         |
| address\[zip]                         | Zip code                                                               | YES                                                                           | string |         |
| address\[municipality]                | Municipality                                                           | YES                                                                           | string |         |
| address\[countryIso2]                 | Country code, ISO 3166-1 alpha-2 format (2 letters)                    | YES                                                                           | string |         |

The recipient's name (`address[fullname]`, or `address[firstname]` and `address[lastname]` joined by a space) may be at
most 50 bytes long. An accented character such as `é` counts as 2 bytes. A longer name returns a 422 with key
`addresseeTooLong`.

For a Belgian address (`address[countryIso2]` is `BE`), `address[zip]` must be a zip code that exists in Belgium: 4
digits, otherwise a 422 is returned (`A Belgian zip code must only contain numbers.`, `A Belgian zip code must be 4
characters exactly.` or `This zip code does not exist in Belgium.`).

### Postal service (optional, default bpost non-prior)
Each Intake request may contain additional postal service instructions

| Key                        | Description                               | Required | Type    | Default |
|----------------------------|-------------------------------------------|----------|---------|---------|
| postalService\[registered] | Registered shipment with tracking (bpost) | NO       | boolean | 0       |
| postalService\[prior]      | Prior shipment                            | NO       | boolean | 0       |
| postalService\[nonPrior]   | Non-prior (standard) shipment             | NO       | boolean | 1       |

Non-prior is used whenever prior is not set: `postalService[nonPrior]` is accepted but has no effect.

### Hook (optional)
Each Intake request may specify a webhook URL and method to send status updates to

| Key           | Description                                                                                                                                                                                               | Required                   | Type   | Default |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------|--------|---------|
| hook\[uri]    | URI where the webhook should be sent to. Must be a valid URL. Requires a webhook key (see [Webhooks](webhooks.md)), otherwise a 422 `No webhook key found, please set one in your dashboard.` is returned | YES, if webhook is desired | string |         |
| hook\[method] | HTTP method to be used for the webhook: `get`, `post`, `GET` or `POST`                                                                                                                                    | NO                         | string | POST    |

For more detailed information on the webhook functionality, refer to the [Webhooks](webhooks.md) page.

### Correlation (required)

| Key                              | Description                                                                       | Required | Type   | Default                                        |
|----------------------------------|-----------------------------------------------------------------------------------|----------|--------|------------------------------------------------|
| correlation\[costId]             | Cost ID, for internal reference                                                   | NO       | string |                                                |
| correlation\[customerIdentifier] | Identifier of the [company customer](company_customers.md) the Intake is sent for | NO       | string |                                                |
| correlation\[lang]               | Document's language, must be one of nl, fr, en, de                                | YES      | string |                                                |
| correlation\[returnAddress]      | Return address in case of undeliverable mail                                      | NO       | string | Symeta Hybrid, Interleuvenlaan 50, 3001 Leuven |

### Duplicity check (optional)
Each Intake request may specify whether a duplicity check should be run.

The check will determine if another Intake exists with the same document name. If attachments are present, the names of
these will also be considered in the check.

If any duplicates are found, a validation error will be returned with a message containing the Intake ID('s) of the
duplicate(s).

| Key            | Description                                     | Required | Type          | Default |
|----------------|-------------------------------------------------|----------|---------------|---------|
| duplicityCheck | Whether or not a duplicity check should be done | NO       | string 'true' |         |

## Multipart Intake Request Response

### Success

| Key              | Value                                                      |
|------------------|------------------------------------------------------------|
| HTTP status code | 202 ACCEPTED                                               |
| JSON body        | `{ "intake": integer, "cost": string, "pricing": object }` |

`cost` is the total cost of the Intake, as a string with 2 decimals (e.g. `"1.23"`). `pricing` is the breakdown it is
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
| 413       | Entity Too Large     | Request size exceeds limit                                              |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal          |
