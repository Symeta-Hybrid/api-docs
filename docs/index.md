# Introduction

## Introduction

Apart from our Mail-IT-Wize web application, we also offer a RESTful API which provides endpoints for processing, printing documents and envelopes, and finally sending the document to your recipient. The API also has options in place for handling return mail.

In addition to the main document, we also allow attachments, backgrounds and custom envelope designs to be used. If a background is used, it will be printed on the **first** page of your main document.

Our API can be divided into 2 large components: sending documents along with metadata (Intake request) and consulting previously sent documents (Resource request).

## Authentication and security

All HTTP requests and responses use SSL. Any unencrypted API requests will **not** be honoured. These requests will not receive any response, and will time out.
For authentication, we use an implementation of OAuth2.

## Components

### Intake request
Contains the main document, background (optional), envelope (optional), attachments (optional) together with all required metadata (address, recipient, document name, ...)

*Important: each intake request contains exactly 1 main document and 1 recipient.*

### Resource request
A resource request can be used to consult previously sent Intake requests.

## API base URL and versioning
There is currently a single version available for the API. This is v1.

Base URL for PROD: https://api.mailitwize.be/api/v1
 
Base URL for UAT: https://api-test.mailitwize.be/api/v1