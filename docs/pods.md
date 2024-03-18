# Proof of Delivery

## Introduction

For registered mail, we have implemented bpost's Proof of Delivery API.
This allows you to track your registered mail, and optionally download the POD PDF document.

## Endpoint and method

There are 2 endpoints available for the POD's: retrieving POD information and downloading the POD document (if available).

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
    "deliveryStates": [
        {
            "stateCode": "PodDownloadRequestPodInfoSucceeded",
            "stateDisplaytext": "Pod download request, pod info succeeded",
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

The POD document will be returned as `binary`, with an `application/pdf` header.