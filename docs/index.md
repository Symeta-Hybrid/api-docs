# Introduction

## Purpose

Apart from our Mail-IT-Wize web application, we also offer a RESTful API which provides endpoints for processing, printing documents and envelopes, and finally sending the document to your recipient. The API also has options in place for handling return mail.

Our API can be divided into 2 large components: sending documents along with metadata (Intake request) and consulting previously sent documents (Resource request).

## Authentication and security

All HTTP requests and responses use SSL. Any API requests sent over unencrypted HTTP will **not** be honoured. These requests will not receive any response, and will time out.
For authentication, we use an implementation of OAuth2.

### Managing access tokens

Access tokens are managed by the users, via the API's web portal. These tokens always have a lifetime of 365 days, and are linked to one specific user.

Upon expiration, the API will return an "Unauthenticated" error upon subsequent requests with said token.

## Components

### Intake request
Contains the main document, background (optional), envelope (optional), attachments (optional) together with all required metadata (address, recipient, document name, ...)

*Important: each intake request contains exactly 1 main document and 1 recipient. Bulk sending in a single request is currently **not** supported.*

### Resource request
A resource request can be used to consult previously sent Intake requests.

## API base URL and versioning
There is currently a single version available for the API in both UAT and PROD. This is v1.

If a new version is introduced, the previous one will remain active for backwards compatibility. All API users will be notified of new releases.

Base URL for PROD: `https://api.mailitwize.be/api/v1`
 
Base URL for UAT: `https://api-test.mailitwize.be/api/v1`

## Technical specification

These docs serve as a general introduction and manual for usage of the API. 

For more detailed specs on all available API functionality, we suggest taking a look at our [Swagger](https://symeta-hybrid.github.io/api-swagger/).