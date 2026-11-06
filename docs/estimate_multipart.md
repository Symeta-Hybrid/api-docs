# Multipart Estimate request

## Endpoint and method

| API version | v1                                               |
|:------------|:-------------------------------------------------|
| Endpoint    | `base_url/api_version/intake/estimate/multipart` |
| Method      | `POST`                                           |

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |

## Request body and document format

The request body and format for an Estimate request are the exact same as the Multipart Intake Request.

Please refer to the [Multipart Intake documentation](intake_multipart.md).

The request is validated exactly like the Intake request, including the PDF checks, `duplicityCheck`, the webhook and
`correlation[customerIdentifier]`, so an Estimate can return the same 422 errors. Nothing is stored or sent.

## Response

| Key              | Value                    |
|------------------|--------------------------|
| HTTP status code | 200 OK                   |
| JSON body        | `{ "estimate": string }` |

`estimate` is the total cost, as a string with 2 decimals (e.g. `"1.23"`).

Partner companies receive HTTP status code 202 ACCEPTED and also `estimate_partner`, the total at partner rates.

## Possible errors

| HTTP code | Description          | Reason                                                                  |
|-----------|----------------------|-------------------------------------------------------------------------|
| 422       | Unprocessable Entity | Request is invalid - either missing or incorrectly formatted parameters |
| 401       | Unauthorized         | Access token is not valid or expired - refer to the web portal          |
