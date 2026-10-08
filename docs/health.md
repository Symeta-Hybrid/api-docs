# Health check request

## Endpoint and method

| API version | v1                            |
|:------------|:------------------------------|
| Endpoint    | `base_url/api_version/health` |
| Method      | `GET`                         |

## Request header

| Header | Value              |
|--------|--------------------|
| Accept | `application/json` |

!!! info
    This endpoint is publicly available and does not require an access token.

## JSON response
If the API is available, it will return the following response, along with an HTTP 200 status code:
```json
{
    "status": "OK"
}
```

## Possible errors

| HTTP code | Description           | Reason                                                       |
|-----------|-----------------------|--------------------------------------------------------------|
| 503       | Service Unavailable   | The API is currently down for maintenance                    |
| 500       | Internal Server Error | The API is experiencing technical issues                     |
| 429       | Too Many Requests     | Request limit of 6000 requests per minute has been exceeded  |
