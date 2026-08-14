# Intake preview request

## Endpoint and method

| API version | v1                                         |
|:------------|:-------------------------------------------|
| Endpoint    | `base_url/api_version/intake/preview/{id}` |
| Method      | `GET`                                      |

## Request header

| Header        | Value                                  |
|---------------|-----------------------------------------|
| Accept        | `application/json` or `application/pdf` |
| Authorization | `Bearer [token]`                        |

The `Accept` header determines the response format: `application/json` returns a summary with the PDF embedded as
base64, while `application/pdf` returns the raw PDF file directly. Any value other than `application/pdf` is
treated as `application/json`.

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

## PDF response
If the request was sent with an `Accept: application/pdf` header, the API returns the composed PDF file directly,
along with a `Content-Type: application/pdf` header, instead of the JSON response above.

## Possible errors

| HTTP code | Description  | Reason                                                           |
|-----------|--------------|-------------------------------------------------------------------|
| 401       | Unauthorized | Access token is not valid or expired - refer to the web portal    |
| 403       | Forbidden    | The Intake does not belong to the authenticated user               |
| 404       | Not Found    | The Intake file could not be found - it may still be processing    |