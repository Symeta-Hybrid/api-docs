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

## When an Intake can be deleted
Only an Intake with status `printing` can be deleted, and never between 22:00 and 05:00 (Europe/Brussels time).
Otherwise a 422 is returned, with the reason in `error`:

| Situation                                                                                                                                         | Error                                                                                                          |
|---------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| Between 22:00 and 05:00 (Europe/Brussels)                                                                                                         | `Intakes can not be deleted between 22:00 and 05:00 local time (Europe/Brussels)`                              |
| Sent to eBox and still waiting there                                                                                                              | `Intake was sent to eBox and can only be deleted once it falls back to paper.`                                 |
| Still being prepared: status `dispatched`, `pickedup` or `processed`, or `received` for a Bulk Intake whose bulk is not dispatched yet            | `Intake is currently in pre-processing and cannot be deleted at this stage. Please try again in a few minutes.` |
| Any later status                                                                                                                                  | `Intake has status {status} and can no longer be deleted.`                                                     |

## JSON response
If the file is successfully deleted, the API will return the following response, along with an HTTP 200 status code:
```json
{
  "success": true,
  "message": "Intake with ID {id} was deleted and will not be printed."
}
```

## Possible errors

| HTTP code | Description          | Reason                                                                                        |
|-----------|----------------------|-----------------------------------------------------------------------------------------------|
| 422       | Unprocessable Entity | Intake cannot be deleted, see [When an Intake can be deleted](#when-an-intake-can-be-deleted) |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal                                |
| 403       | Forbidden            | The Intake does not belong to the authenticated user                                          |
| 400       | Bad Request          | Confirmation hash does not match                                                              |