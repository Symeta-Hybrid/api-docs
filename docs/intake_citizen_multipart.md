# eBox Citizen Multipart Intake Request

!!! info "Eligibility and access"
    
    If you wish to make use of our eBox Citizen service, please contact us at [support@mailitwize.be](mailto:support@mailitwize.be).

    Before you can start using this endpoint to send messages to eBox a check needs to be done by BOSA (our eBox Citizen partner) 
    to determine if your organisation is eligible for sending eBox messages.
    
    If you are eligible, we will go through the process together to complete the audit process and enable eBox functionality.

    Do note that we have different partners for Citizen and Enterprise, and there are separate procedures to be completed.

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

recipient_identification_number => 'string, length 11',
subject_nl => 'string',
subject_en => 'string',
subject_fr => 'string',

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

hook[uri] => 'string',
hook[method] => 'string',

hook_uri => 'Full webhook path for eBox updates',
hook_method => 'POST or GET for eBox updates',

correlation[costId] => 'string,optional',
correlation[lang] => 'required, nl,en or fr',
correlation[returnAddress] => 'optional string, single line return address',

duplicityCheck => 'boolean, optional'
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

### Recipient identification number (required)
Each Intake request must contain exactly one identification number. This is the national registration number of the recipient.
The string provided must contain only numeric characters and adhere to the standards of the Belgian national registration number.

| Key                             | Description                                   | Required | Type   | Default |
|---------------------------------|-----------------------------------------------|----------|--------|---------|
| recipient_identification_number | National registration number of the recipient | YES      | string |         |

### Subject for eBox message (required)
Each Intake request must contain at least one localised subject.

Multiple subject localisations can be provided. These will be included in the eBox message. The eBox Portal will
decide which one to display based on the recipient's eBox locale settings.

| Key        | Description               | Required                                 | Type   | Default |
|------------|---------------------------|------------------------------------------|--------|---------|
| subject_nl | Message subject in Dutch  | YES, if subject_fr AND subject_de absent | string |         |
| subject_fr | Message subject in French | YES, if subject_nl AND subject_de absent | string |         |
| subject_de | Message subject in German | YES, if subject_nl AND subject_fr absent | string |         |

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

### Attachments (optional)
Each Multipart Intake request may contain multiple attachments, using a consecutive numerical index.

| Key                         | Description                                                               | Required                 | Type        | Default |
|-----------------------------|---------------------------------------------------------------------------|--------------------------|-------------|---------|
| attachments\[n]\[name]      | Name of the attachment                                                    | YES, if using attachment | string      |         |
| attachments\[n]\[mime-type] | MIME type of the attachment. Currently only "application/pdf" is accepted | YES                      | string      |         |
| attachments\[n]\[contents]  | Attachment PDF file                                                       | YES                      | file/binary |         |
| attachments\[n]\[multiplex] | Print setting of the attachment. 0 = recto, 1 = recto/verso               | NO                       | boolean     | 1       |

### Background (optional)
Each Multipart Intake request may contain exactly 1 background. The background will only be printed on the first page 
of the main document.

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

### eBox Hook (optional)
Each Intake request may specify an eBox webhook URL and method to send status updates to.
These updates relate to the eBox message processing by BOSA.

| Key         | Description                                                        | Required                   | Type   | Default |
|-------------|--------------------------------------------------------------------|----------------------------|--------|---------|
| hook_uri    | URI where the webhook should be sent to                            | YES, if webhook is desired | string |         |
| hook_method | HTTP method to be used for the webhook, must be either GET or POST | YES, if webhook is desired | string |         |

For more detailed information on the webhook functionality, refer to the [Webhooks](webhooks.md) page.

### Correlation (required)

| Key                         | Description                                        | Required | Type   | Default                                        |
|-----------------------------|----------------------------------------------------|----------|--------|------------------------------------------------|
| correlation\[costId]        | Cost ID, for internal reference                    | NO       | string |                                                |
| correlation\[lang]          | Document's language, must be one of nl, fr, en, de | YES      | string |                                                |
| correlation\[returnAddress] | Return address in case of undeliverable mail       | NO       | string | Symeta Hybrid, Interleuvenlaan 50, 3001 Leuven |

### Duplicity check (optional)
Each Intake request may specify whether a duplicity check should be run.

The check will determine if another Intake exists with the same document name. If attachments are present, the names of these will also be considered in the check.

If any duplicates are found, a validation error will be returned with a message containing the Intake ID('s) of the duplicate(s).

| Key            | Description                                     | Required | Type          | Default |
|----------------|-------------------------------------------------|----------|---------------|---------|
| duplicityCheck | Whether or not a duplicity check should be done | NO       | string 'true' |         |

## Multipart Intake Request Response

### Success

| Key              | Value                                                       |
|------------------|-------------------------------------------------------------|
| HTTP status code | 202 ACCEPTED                                                |
| JSON body        | `{ "intake": integer, "cost": integer, "message": string }` |

### Possible error codes

| HTTP code | Description           | Reason                                                                   |
|-----------|-----------------------|--------------------------------------------------------------------------|
| 422       | Unprocessable Entity  | Request is invalid - either missing or incorrectly formatted parameters  |
| 400       | No eBox configuration | There was no valid eBox configuration found associated with your account |
| 401       | Unauthorized          | Access token is not valid or expired - refer to the web portal           |
