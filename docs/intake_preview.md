# Intake preview request

## Endpoint and method

| API version | v1                                         |
|:------------|:-------------------------------------------|
| Endpoint    | `base_url/api_version/intake/preview/{id}` |
| Method      | `GET`                                      |

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |

## JSON response
The API will return a JSON response containing the following properties, where `previewBase64` contains the final 
composed PDF file's contents, in a base-64 encoded format.
```json
{
    "attachmentPages": 6,
    "documentPages": 6,
    "previewBase64": "{base64 encoded string}",
    "totalPages": "18"
}
```