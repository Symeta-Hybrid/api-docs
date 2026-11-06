# JSON Estimate request

## Endpoint and method

| API version | v1                                     |
|:------------|:---------------------------------------|
| Endpoint    | `base_url/api_version/intake/estimate` |
| Method      | `POST`                                 |

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |

## Request body and document format

The request body and format for an Estimate request are the exact same as the JSON Intake Request.

Please refer to the [JSON Intake documentation](intake_json.md).

The request is validated exactly like the Intake request, including the PDF checks, `duplicityCheck`, the webhook and
`correlation.customerIdentifier`, so an Estimate can return the same 422 errors. Nothing is stored or sent.

## Response

| Key              | Value                    |
|------------------|--------------------------|
| HTTP status code | 202 ACCEPTED             |
| JSON body        | `{ "estimate": string }` |

`estimate` is the total cost, as a string with 4 decimals (e.g. `"1.2345"`).

Partner companies also receive `estimate_partner`, the total at partner rates, and `detail` and `detail_partner`: the
cost breakdown at both rates, with the same keys as `pricing` in the [JSON Intake
response](intake_json.md#json-request-response).

## Possible errors

| HTTP code | Description          | Reason                                                                  |
|-----------|----------------------|-------------------------------------------------------------------------|
| 422       | Unprocessable Entity | Request is invalid - either missing or incorrectly formatted parameters |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal          |
