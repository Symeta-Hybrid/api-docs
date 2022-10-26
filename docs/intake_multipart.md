# Multipart Intake Request
## Multipart vs JSON
Using a Multipart Request allows to upload larger documents to the API. However, we do advise to use the JSON Intake Request as much as possible and avoid large filesizes.
With the Multipart Request, the 4MB filesize limit does not apply. For the most part, the requests are very similar.

## Endpoint and method

| API version | v1                                    |
|:------------|:--------------------------------------|
| Endpoint    | base_url/api_version/intake/multipart |
| Method      | POST                                  |

## Request header

| Header        | Value            |
|---------------|------------------|
| Accept        | application/json |
| Authorization | Bearer [token]   |


## Multipart form-data Structure
``` form-data
    document[name] => 'string',
    document[mime-type] => 'mime-type',
    document[contents] => 'contents', ! not base64encoded !
    document[multiplex] => integer,

    // attachments

    attachments[0][name] => 'string',
    attachments[0][mime-type] => 'mime-type',
    attachments[0][contents] => 'content' ! not base64encoded !
    attachments[0][multiplex] => integer

    attachments[1][name] => ...,

    // background

    background[name] => 'string',
    background[mime-type] => 'mime-type',
    background[contents] => 'content' ! not base64encoded !

    // envelope

    envelope[name] => 'string',
    envelope[mime-type] => 'mime-type',
    envelope[contents] => 'content' ! not base64encoded !

    // address

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
    address[company] => 'string',
    address[area] => 'string',

    // postalService

    postalService[registered] => boolean,
    postalService[prior] => boolean,
    postalService[nonPrior] => boolean,

    // email

    email[registered] => boolean,
    email[email] => 'string',

    // correlation
    correlation[costId] => 'string,optional',
    correlation[lang] => 'required, nl,en or fr',
    correlation[returnAddress] => 'optional string, single line return address'

    // add request type multipart
    requestType => 'multipart'

]
```

Multipart Form Data Values
requestType (required)
Add parameter requestType and set value as 'multipart' to indicate it is a multipart form.

requestType (required, string value: multipart)

document (required)
Bevat nodige data mbt het document, en bevat onderstaande componenten.

document.name (required, string)

Naam van het originele document inclusief file extensie

document.mime-type (required, string)

Mime-type van document. Momenteel enkel application/pdf

document.contents (required, file contents)

De content van het document.

document.multiplex (optional, integer)

Printwijze:
0 = recto
1 = recto / verso

attachments (optional)
Een intake request kan meerdere bijlagen bevatten met een opeenvolgende numerieke index (zie ook json structure)

attachments.(index).name (string, required if attachments)

Naam van het originele bestand inclusief extensie.

attachments.(index).mime-type (mime-type, required if attachments)

Mime-type van document. Momenteel enkel application/pdf

attachments.(index).contents (required if attachments)

Content van bestand.

attachments.(index).multiplex (integer, optional, default = 1)

Printwijze:
0 = recto
1 = recto / verso

background (optional)
Elke intake request kan maximaal 1 achtergrond bevatten. We plaatsen deze achtergrond enkel op de eerste pagina van het hoofddocument.

background.name (string, required if background)

Naam van het originele bestand inclusief extensie.

background.mime-type (mime-type, required if background)

Mime-type van document. Momenteel enkel application/pdf

background.contents (required if background)

Content van bestand.

envelope (optional)
Elke intake request kan maximaal 1 envelop bedrukking bevatten.

envelope.name (string, required if envelope)

Naam van het originele bestand inclusief extensie.

envelope.mime-type (mime-type, required if envelope)

Mime-type van document. Momenteel enkel application/pdf

envelope.contents (required if envelope)

Content van bestand.

address (required)
Elke intake request moet een adres component bezitten.

address.firstname (string, required if no address.fullname and no address.company)

address.lastname (string, required if no address.fullname and no address.company)

address.fullname (string, required if no address.firstname and/or no address.lastname, and no address.company)

address.street (string, required if no address.unstructuredStreetNumberBox)

address.houseNumber (integer, required if no address.unstructuredStreetNumberBox)

address.houseNumberAlpha (string, optional)

address.box (string, optional)

address.unstructuredStreetNumberBox (string, required if no address.street or no address.houseNumber)

address.zip (string, required)

address.municipality (string, required)

address.countryIso2 (string, max two characters, required)

address.company (string, required if no address.firstname or no address.lastname or no address.fullname)

address.area (string, optional)

correlation (required)
correlation.costId (string, optional)

correlation.lang (required, must be one of nl,fr,en,de)

correlation.returnAddress (optional, ex. Death Star Inc, Darth Vader Avenue 1, 1111 Space)

postalService (optional, default bpost non-prior)
postalService.registered (optional, boolean, default 0)

postalService.prior (optional,boolean, default 0)

postalService.nonPrior (optional, boolean, default 1)

email (optional)
email.registered (optional, boolean, default 0)

email.email (required if email or email.registered, string)

hook (optional)
To receive status updates about the intake processing you can add a webhook path.

Payload json {"intake":"id","status":"status"}

hook.uri (required if hook is set)

hook.method (required if hook is set)

Webhook
Header	Signature	hash_hmac('sha256', 'payloadjson', 'webhook key')
Body	{"intake":"id","status":"status"}
Als signature gelijk is aan hash_hmac('sha256','json response','webhook key') payload is valid.

Multipart Request Response
Success
Status Code	202 Accepted
JSON body	{"intake":integer, "cost":integer} (intake = id van intake request)
Errors
422	Unprocessable Entity	Request voldoet niet aan structuur en waarden.
413	Entity Too Large	Request size te groot.
401	Unauthorized	Access token ongeldig of vervallen.
Ook andere standaard errors zijn mogelijk vb 500 server error