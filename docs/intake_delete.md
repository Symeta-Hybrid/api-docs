# Intake delete request

## Endpoint and method

| API version | v1                                                            |
|:------------|:--------------------------------------------------------------|
| Endpoint    | `base_url/api_version/intake/delete/{id}?confirmation={hash}` |
| Method      | `POST`                                                        |

`{id}` is the numeric ID of the Intake to be deleted.

`{hash}` should be replaced by an MD5 hash of the zip code of the Intake to be deleted.

!!! example
    For zip code 9000, this would result in the following URL:

    `https://api.mailitwize.be/api/v1/intake/delete/1450?confirmation=d5ab8dc7ef67ca92e41d730982c5c602`

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |


## JSON response
If the file is successfully deleted, the API will return the following response, along with an HTTP 200 status code:
```json
{
  "success": true,
  "message": "Intake with ID {id} was deleted and will not be printed."
}
```

## Possible errors

| HTTP code | Description          | Reason                                                                    |
|-----------|----------------------|---------------------------------------------------------------------------|
| 422       | Unprocessable Entity | Request could not be fulfilled.<br/>Refer to error message for specifics. |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal            |
| 400       | Bad Request          | Confirmation hash does not match                                          |