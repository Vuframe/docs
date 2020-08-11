---
title: VUFRAME® Public API

toc_footers:
  - <a href='mailto:info@vuframe.com'>Request an access token</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - tutorial
  - content_types

search: true
---

# Introduction

The VUFRAME® Public API offers a number of endpoint to view, create and edit different types of content within the VUFRAME® platform. The public API can be accessed by any VUFRAME® Studio user.

The base URL for all the endpoints listed below is 

`https://api.vuframe.com/public/v1`

or alternatively

`https://api-beta.vuframe.studio/public/v1`

for the Beta environment.

# Authentication

In order to integrate the VUFRAME® Public API into your own application, you will first need to register an SSO app by contacting us directly. After your app is set up, you will receive a unique SSO app token that allows you to communicate with the VUFRAME® Public API.

In addition to the SSO app token, all requests to the VUFRAME® Public API need to be authenticated with an authentication token that is bound to an active VUFRAME® Studio account. The required authentication mechanism is Basic Auth with the following credentials:

`Username: SSO_APP_TOKEN
Password: USER_AUTH_TOKEN`

# SSO Tokens

## Create an SSO Auth Token for a VUFRAME® Studio User

> Returns the newly created auth token in the following format:

```json
{
  success: true,
  token: "<auth token>"
}
```

Creates an authentication token for a VUFRAME® Studio user. This token can then be used to access the various endpoints of the VUFRAME® Public API. If the user already has a valid auth token for the given SSO app, the existing token will be returned instead of creating a new one.

### HTTP Request

`POST https://api.vuframe.com/public/v1/:app_token/create_token`

<aside class="notice">
Replace <code>:app_token</code> with your SSO App Token.
</aside>

### Authentication:

This endpoint uses Basic Auth with the following Credentials:

`Username: STUDIO_EMAIL
Password: STUDIO_PASSWORD`

<aside class="notice">
Replace <code>STUDIO_EMAIL</code> with the e-mail address of a valid VUFRAME® Studio user and <code>STUDIO_PASSWORD</code> with the password associated with that account.
</aside>



# Projects

## Get a list of all projects

> Returns a list of all accessible projects in the following format:

```json
[
    {
        "platform_version": 1,
        "token": "<project token>",
        "name": "My Project"
    }, 
    {
        ...
    }
]
```

Returns a list of all VUFRAME® Studio projects the user has access to.

### HTTP Request

`GET https://api.vuframe.com/public/v1/projects`



# Folders/SmartVu's

## Get a list of all folders and SmartVu's

> Returns a list of all accessible folders and SmartVu's in the following format:

```json
[
    {
        "platform_version": 1,
        "name": "Folder 1",
        "token": "<folder token>",
        "team": "<project token>",
        "type": "folder"
    },
    {
        "platform_version": 1,
        "name": "SmartVu 1",
        "token": "<SmartVu token>",
        "team": "<project token>",
        "type": "feature"
    },
    {
        ...
    }
]
```
> **Notes:**<br/>
> - The value "team" contains the token of the VUFRAME® Studio project the folder or SmartVu belongs to<br/>
> - The value "type" can be either "folder" or "feature" for folders and SmartVus respectively

Returns a list of all SmartVu's and folders the current user has access to.

### HTTP Request

`GET https://api.vuframe.com/public/v1/folders`

# SmartVu's

These endpoints allow the user to create, edit and share SmartVu's.

## Create a new SmartVu

> Returns the newly created SmartVu in the following format:

```json
{
    "success": true,
    "smartvu": {
        "token": "<token of the new SmartVu>",
        "title": "New SmartVu Title",
        "item_token": "<item token>" | null,
        "description": "New SmartVu Description",
        "version": 1,
        "shared": true,
        "share_code": "<share code>" | null,
        "shared_version": 1 | null
    }
}
```
> **Notes:**<br/>
> - The values "share_code" and "shared_version" are only present if the request param "share" was set to true (null otherwise)<br/>
> - Using the share code from the response, the user is able to access the share page for the newly created SmartVu under https://share.vuframe.com/<share_code>

Create a new SmartVu in one of the project the current user has access to.

### HTTP Request

`POST https://api.vuframe.com/public/v1/smartvus`

### Request Parameters

Parameter | Type | Description
--------- | ---- | -----------
project_token | string | token of the project the SmartVu should be created in
title | string | title of the new SmartVu
description | string | (optional) description of the new SmartVu
item_token | string | (optional) token of the 3D item that the SmartVu should display
share | boolean | (optional) if true, the new SmartVu will be shared immediately

<aside class="notice">If no item token is specified, an empty SmartVu will be created.</aside>

## Update an existing SmartVu

> Returns the updated SmartVu in the following format:

```json
{
    "success": true,
    "smartvu": {
        "token": "<token of the SmartVu>",
        "title": "New SmartVu Title",
        "item_token": "<item token>" | null,
        "description": "New SmartVu Description",
        "version": 2,
        "shared": true,
        "share_code": "<share code>" | null,
        "shared_version": 2 | null
    }
}
```
> **Notes:**<br/>
> - The values "version" and "shared_version" show the latest and the public version revision of the SmartVu respectively<br/>
> - If the SmartVu is not shared, "share_code" and "shared_version" are set to null

Edit and/or share an existing SmartVu.

### HTTP Request

`PATCH https://api.vuframe.com/public/v1/smartvus/:smartvu_token`

### Request Parameters

Parameter | Type | Description
--------- | ---- | -----------
smartvu_token | string | (URL param) token of the SmartVu that should be changed
title | string | (optional) new title of the SmartVu
description | string | (optional) new description of new SmartVu
item_token | string | (optional) token of the 3D item that the SmartVu should display
share | boolean |- if true, the new version of the SmartVu is shared<br>- if false, the SmartVu is unshared (no longer publicly accessible)<br>- if not specified or null, the share status of the SmartVu remains unchanged

# Item Upload & Processing

These Endpoints allow uploading new 3D files to VUFRAME® Studio and processing them using the AURA Conversion Pipeline.

## Create a new item

> Returns the following:<br>
> - A presigned Amazon S3 URL that allows the user to upload their 3D file to the VUFRAME® cloud storage.<br>
> A callback URL pointing to another Public API endpoint that the user needs to call after the file has been uploaded successfully. As soon as this callback is received, VUFRAME® Studio will start processing the uploaded file.

```json
{
    "presigned_url": "<S3 upload URL>",
    "callback_url": "<Public API callback URL>",
    "upload_action": "upload as new item",
    "upload_filename": "generated unique filename",
    "resolved_content_type": "recognized content type"
}
```

Prepare the upload of a new 3D file to the specified folder or SmartVu.

### HTTP REQUEST

`POST https://api.vuframe.com/public/v1/items`

### Request Parameters

Parameter | Type | Description
--------- | ---- | -----------
folder_token | string | token of the folder or SmartVu the new 3D item should be created in
folder_type | string | "folder" or "feature" depending on whether the item should be created in a folder or a SmartVu
filename | string | filename with file type (see [List of supported file types](#upload-types)) of the 3D file that should be uploaded
content_type | string | content type identifier describing the type of 3D file that should be uploaded (see [List of supported content types](#upload-types))

## Process a new item

> Returns the following:<br>
> - A direct link to the VUFRAME® Studio folder or SmartVu that contains the newly uploaded file<br>
> A resource token that can be used for tracking the processing status of the file

```json
{
  entity_url: "<direct link to VUFRAME® Studio>",
  resource_token: "<resource token>"
}
```

The callback URL obtained via POST /items points to this endpoint. Sending a request to this URL will start the processing task for a newly uploaded 3D file. 

### HTTP REQUEST

`POST https://api.vuframe.com/public/v1/items/:upload_id/upload_complete`

<aside class="notice">This endpoint is not intended to be requested manually. It should only be used in conjunction with <code>POST /items</code>.</aside>

## Get file processing status

> Returns the tokens of all VUFRAME® Studio Entities associated with this file as well as its current processing status:

```json
{
    "resource_token": "<resource token>",
    "project_token": "<token of the VUFRAME® Studio project the file was uploaded to>",
    "folder_token": "<token of the VUFRAME® Studio folder or SmartVu the file was uploaded to>",
    "item_token": "<token of the the item that was created for this file>" | null,
    "ready": true,
    "finished_processing": true,
    "success": true,
    "status": "Complete."
}
```
> **Notes:**<br/>
> - As the item for the file is only created after the file has finished processing, the value "item_token" is null until then. After the item has been created, you can assign it to a new or existing SmartVu using its token<br/>
> - The value "ready" is true only if the item has finished processing and the processing task was completed successfully<br/>
> - While the processing task is running, the value "success" is null
> - The value "status" offers a short description of the processing step the file is currently undergoing

Get the current processing status of a 3D file that was uploaded using the Public API.

### HTTP Request

`GET https://api.vuframe.com/public/v1/items/:resource_token/status`

#### Request Params

Parameter | Type | Description
--------- | ---- | -----------
resource_token | string | (URL param) resource token obtained from the response of `POST /items/:upload_id/upload_complete`
