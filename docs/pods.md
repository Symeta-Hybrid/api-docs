# Proof of Delivery

## Introduction

For registered mail, we have implemented bpost's Proof of Delivery (POD) API. This allows you to track your registered
mail, and optionally download the POD PDF document.

## Request header

| Header        | Value              |
|---------------|--------------------|
| Accept        | `application/json` |
| Authorization | `Bearer [token]`   |

## Endpoint and method

There are 2 endpoints available for the POD's: retrieving POD information and downloading the POD document (if
available).

### Request POD information

| API version | v1                                            |
|:------------|:----------------------------------------------|
| Endpoint    | `base_url/api_version/proof-of-delivery/{id}` |
| Method      | `GET`                                         |

#### Response

``` json
{
    "podGuid": "string",
    "podBusinessKey": "miw2_",
    "requestDateTime": "2024-03-20T14:00:00+00:00",
    "barcode": "JJBEA000000000000012345",
    "currentState": {
        "stateCode": "PodDownloadRequestPodInfoSucceeded",
        "stateDisplayText": "Pod download request, pod info succeeded",
        "transitionDateTime": "2023-08-01T10:00:00+02:00"
    },
    "stateHistory": [
        {
            "stateCode": "PodDownloadRequestPodInfoSucceeded",
            "stateDisplayText": "Pod download request, pod info succeeded",
            "transitionDateTime": "2023-08-01T10:00:00+02:00"
        }
    ],
    "documentInfo": {
        "hasPodAvailable": true,
        "hasOriginalDocumentAvailable": true
    }
}
```

### Download POD document

!!! attention
    This will only work if the `hasPodAvailable` parameter from the information request is `true`.

| API version | v1                                                     |
|:------------|:-------------------------------------------------------|
| Endpoint    | `base_url/api_version/proof-of-delivery/{id}/document` |
| Method      | `POST`                                                 |

#### Response

The POD document will be returned as `binary`, with an `application/pdf` header, as a download
(`Content-Disposition: attachment`).

## Possible errors

Both POD endpoints can return:

| HTTP code | Description | Code                       | Reason                                                                                                     |
|-----------|-------------|----------------------------|------------------------------------------------------------------------------------------------------------|
| 403       | Forbidden   | `FORBIDDEN`                | The Intake does not belong to the authenticated user                                                       |
| 404       | Not Found   | `EPOD_NOT_REGISTERED`      | The Intake was not sent registered, so it will never have a POD                                            |
| 404       | Not Found   | `EPOD_NOT_YET_AVAILABLE`   | No POD yet: the Intake has not been shipped (no bpost barcode), or bpost has not provided the POD information (information request) or the POD document (download) yet |
| 502       | Bad Gateway | `EPOD_ARCHIVE_UNAVAILABLE` | Download only: the POD document could not be fetched from the archive. Try again later                     |

Every error body is `{"code": "...", "message": "..."}`, for example:

``` json
{
    "code": "EPOD_NOT_YET_AVAILABLE",
    "message": "The proof of delivery for this shipment is not available yet. Please try again later."
}
```

An Intake id that does not exist also answers 404, but with a `message` only and no `code`.
