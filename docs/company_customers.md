# Company Customers

## Introduction
If you send mail on behalf of your own customers, you can register them as company customers. Each Intake request can
then specify the customer it is sent for.

Once we have approved the customer, its name is used as the sender of the printed mail instead of your company's name.
For eBox, the eBox configuration set up for that customer is used (see [eBox](#ebox)).

## Managing company customers
Company customers are managed by the users, via the API's web portal, under *Company Customers*.

| Key         | Description                                                                           | Required | Type   | Default |
|-------------|---------------------------------------------------------------------------------------|----------|--------|---------|
| Identifier  | The value to send in Intake requests. Max. 255 characters, unique within your company | YES      | string |         |
| Name        | The customer's name, used as sender. Max. 255 characters                              | YES      | string |         |
| Description | Free text for your own reference                                                      | NO       | string |         |

Once Intakes have been sent for a customer, its identifier can no longer be changed and the customer can no longer be
deleted. A customer with an eBox configuration cannot be deleted either.

### Approval
New customers are inactive until we approve them. When you change the identifier or name of an approved customer, it
becomes inactive again until we approve the change.

Intakes can be sent for a customer that awaits approval. They are accepted and linked to the customer as usual, but your
company's name is used as sender until the customer is approved. For eBox, see [eBox](#ebox).

## Customer identifier (optional)
Each Intake request may specify the company customer it is sent for, in its correlation data. This applies to the
[JSON](intake_json.md), [Multipart](intake_multipart.md), [Bulk](intake_bulk.md), [eBox
Citizen](intake_citizen_multipart.md) and [eBox Enterprise](intake_enterprise_multipart.md) Intake requests, and to the
Estimate requests.

| Key                              | Description                                                           | Required | Type   | Default |
|----------------------------------|-----------------------------------------------------------------------|----------|--------|---------|
| correlation.customerIdentifier   | Identifier of the company customer the Intake is sent for (JSON)      | NO       | string |         |
| correlation\[customerIdentifier] | Identifier of the company customer the Intake is sent for (multipart) | NO       | string |         |

The identifier must be one of your company's customers, approved or not. For a Bulk Intake, the customer applies to
every Intake created from it. When omitted, the Intake is sent for your company itself.

!!! attention
    The identifier must be sent as a string. In a JSON request, send `"123"` rather than `123`.

## eBox
eBox messages are sent with an eBox configuration that we set up for you. If you send eBox messages on behalf of your
customers, each customer can have its own eBox configuration, which we link to the customer once it is approved.

The eBox requests also accept the identifier as `customer_identifier`. It is the same as
`correlation[customerIdentifier]`, and either one may be used. When both are sent, they must be equal apart from case
(`ACME` and `acme` match).

When a customer is specified, the eBox configuration linked to that customer is used. When no customer is specified,
your company's own eBox configuration is used, never one linked to a customer.

A customer whose identifier or name was changed keeps its eBox configuration while the change awaits approval, so its
messages are still sent. Only a paper fallback uses your company's name as sender until the change is approved.

[eBox status updates](webhooks.md#ebox-status-updates) for a message sent for a customer carry the customer's identifier
in `customer_identifier`.

## Possible errors

| HTTP code | Description           | Reason                                                                                                                         |
|-----------|-----------------------|--------------------------------------------------------------------------------------------------------------------------------|
| 422       | Unprocessable Entity  | `correlation.customerIdentifier` is not one of your company's customers, or differs from `customer_identifier` apart from case |
| 400       | No eBox configuration | eBox only: no eBox configuration is linked to the customer, for example because it is not approved yet                         |
