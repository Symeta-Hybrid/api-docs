# Token lifetime request

## Endpoint and method

| API version | v1                              |
|:------------|:--------------------------------|
| Endpoint    | `base_url/api_version/lifetime` |
| Method      | `POST`                          |

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |

## JSON response
The API will return the expiration date of the token used in the request as a JSON string, along with an HTTP 200 status code:
```json
"2027-10-07T09:30:00.000000Z"
```

## Possible errors

| HTTP code | Description  | Reason                                                         |
|-----------|--------------|----------------------------------------------------------------|
| 401       | Unauthorized | Access token is not valid or expired - refer to the web portal |
