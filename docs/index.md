# Introduction

## Purpose

Apart from our Mail-IT-Wize web application, we also offer a RESTful API which provides endpoints for processing,
printing documents and envelopes, and finally sending the document to your recipient. The API also has options in place
for handling return mail.

Our API can be divided into 2 large components: sending documents along with metadata (Intake request) and consulting
previously sent documents (Resource request).

## Authentication and security

All HTTP requests and responses use SSL. Any API requests sent over unencrypted HTTP will **not** be honoured. These
requests will not receive any response, and will time out. For authentication, we use bearer tokens. Send your access
token in the `Authorization: Bearer [token]` header of every request.

### Managing access tokens

Access tokens are managed by the users, via the API's web portal. These tokens always have a lifetime of 365 days, and
are linked to one specific user.

Upon expiration, the API will return an "Unauthenticated" error upon subsequent requests with said token.

Tokens cannot be refreshed or renewed through the API. Use the [Token lifetime request](token_lifetime.md) to check when
your token expires.

## Components

### Intake request
Contains the main document, background (optional), envelope (optional), attachments (optional) together with all
required metadata (address, recipient, document name, ...)

*Important: each intake request contains exactly 1 main document and 1 recipient. To send one PDF to multiple recipients
in a single request, use the [Bulk Intake](intake_bulk.md) request.*

### Resource request
A resource request can be used to consult previously sent Intake requests.

## API base URL and versioning
There is currently a single version available for the API in both UAT and PROD. This is v1.

If a new version is introduced, the previous one will remain active for backwards compatibility. All API users will be
notified of new releases.

Base URL for PROD: `https://api.mailitwize.be/api/v1`

Base URL for UAT: `https://api-acc.mailitwize.be/api/v1`

## Errors
Every response is JSON, also for errors, with one exception: the [Preview](intake_preview.md) answers its errors
in JSON only when the request carries `Accept: application/json`. The error bodies most endpoints share are:

| HTTP code | Description          | Body                                                                                                 |
|-----------|----------------------|------------------------------------------------------------------------------------------------------|
| 401       | Unauthorized         | `{"message": "Unauthenticated.", "error": "Token is invalid or expired."}`                           |
| 403       | Forbidden            | `{"code": "FORBIDDEN", "message": "You are not allowed to perform this action."}`                    |
| 422       | Unprocessable Entity | An object with one key per invalid field, each holding a list of messages                            |
| 429       | Too Many Requests    | `{"code": "THROTTLED", "message": "Too many requests. Please try again in 30 seconds."}`             |

An endpoint that answers an error differently says so on its own page.

The API accepts at most 6000 requests per minute. Above that it returns the 429, with a `Retry-After` header giving the
number of seconds to wait.

Example of a 422 body:
```json
{
    "address.zip": ["This zip code does not exist in Belgium."],
    "correlation.lang": ["The selected correlation.lang is invalid."]
}
```

## Technical specification

These docs serve as a general introduction and manual for usage of the API.

For more detailed specs on all available API functionality, we suggest also taking a look at our
[Swagger](https://symeta-hybrid.github.io/api-swagger/).