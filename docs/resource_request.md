# JSON Resource Requests
## General
A resource request returns a data collection, formatted as JSON.

### Resource Request Header


| Header        | Value            |
|---------------|------------------|
| Accept        | `application/json` |
| Authorization | Bearer \[token\] |

## Available Resource Collections
### Intake Resource
With the Intake ID received when submitting an Intake, you can retrieve all information related to that Intake.

* Endpoint: BASE_URI/intake/show/{intake_id}
* Method: GET
* Example: https://api.mailitwize.be/intake/show/678

``` json
{
    "data": {
        "id": 78754,
        "firstname": "John",
        "lastname": "Doe",
        "fullname": null,
        "company": null,
        "street": "Fernand Scribedreef",
        "number": "1",
        "number_alpha": null,
        "box": null,
        "unstructured_street": "",
        "zip": "9000",
        "municipality": "Gent",
        "country_iso2": "BE",
        "cost_id": null,
        "status": "Printing",
        "created_at": "2022-12-08T13:16:15.827000Z",
        "updated_at": "2022-12-08T13:18:05.407000Z",
        "mailpack": {
            "id": 88715,
            "intake_id": "78754",
            "name": "miw2_78754_231ec460-b7bd-4da4-9bd5-ed54bc08cff0_mailpack.pdf",
            "pages": "2",
            "envelope_type": "US",
            "created_at": "2022-12-08T13:17:08.150000Z",
            "updated_at": "2022-12-08T13:17:08.150000Z"
        },
        "envelope": {
            "id": 17147,
            "intake_id": "78754",
            "name": "miw2_78754_231ec460-b7bd-4da4-9bd5-ed54bc08cff0_envelope.pdf",
            "original_name": "miw2_78754_231ec460-b7bd-4da4-9bd5-ed54bc08cff0_envelope.pdf",
            "created_at": "2022-12-08T13:16:15.857000Z",
            "updated_at": "2022-12-08T13:16:15.857000Z"
        },
        "shipping_barcode": false,
        "shipping_date": false
    }
}
```