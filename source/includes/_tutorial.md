# How To: Create a SmartVu from a 3D file

Using the VUFRAMEⓇ Public API, you can easily create a simple SmartVu from a 3D file. For a detailed explanation of the API's endpoints and how to use them, please refer to this article.

## 1. Creating the empty SmartVu

By making a `POST` request to the `/smartvus` endpoint, you can create an empty SmartVu in any VUFRAMEⓇ Studio project you have access to. You need to provide the following parameters:

Parameter | Type | Description
--------- | ---- | -----------
project_token | string | token of the project the SmartVu should be created in
title | string | title of the new SmartVu
description | string | (optional) description of the new SmartVu

Save the "token" value of the "smartvu" object from the response - you will need it for the next steps.

## 2. Uploading a 3D file

In order to upload a 3D file to VUFRAMEⓇ Studio, you will first need to obtain an upload URL. This can be achieved by calling `POST /items` with the following parameters:

Parameter | Type | Description
--------- | ---- | -----------
folder_token | string | the SmartVu token you obtained in the previous step
folder_type | string | this value should be set to "feature"
content_type | string | content type identifier describing the type of 3D file that should be uploaded (see [List of supported content types](#upload-types))

The response will contain two URLs: The presigned_url which will be used to upload the 3D file to our cloud storage and the callback_url which you will need to call after the file upload is complete.

First, you will need to upload your 3D file to the presigned S3-URL using a PUT-Request. When the upload process has finished, make a POST request to the callback URL you received. This will start the internal processing and conversion of the 3D file. Save the resource_token value of the response object as you will need it for the next step.

## 3. Processing the 3D file

Before you can add your 3D file to the SmartVu you created in the first step, it needs to be processed and converted to the VF3D-Format. This may take a couple of minutes, depending on the size and type of the file you uploaded. In order to check the current processing status of your file, simply call `GET /items/:resource_token/status` with the resource token you received in the previous step.

Keep calling this endpoint periodically until the value `finished_processing` is set to true. This means that your file has finished processing. Additionally, check the value for `success`. If it is `false`, your file could not be processed for some reason. If it is set to `true`, your file was processed successfully and you can now add it to the SmartVu you created in the first step. In order to do so, please save the value for `item_token`.

## 4. Adding the 3D file to the SmartVu

The final step in setting up your SmartVu is adding the file you uploaded in the last step. To do so, make a PATCH request to `/smartvus/:smartvu_token` using the SmartVu token you received in the first step. Additionally, pass the following parameters:

Parameter | Type | Description
--------- | ---- | -----------
item_token | string | the item token you acquired in the previous step
share | boolean | (optional) if you want your SmartVu to be publicly available, set this parameter to true (otherwise, do not set it at all)

You should now have successfully set up a SmartVu containing your 3D file. Log into VUFRAMEⓇ Studio with your account and the SmartVu should be right there. If you chose to share your SmartVu, you can additionally access its public share page by appending the share_code value from the response to the share page's base URL `https://share.vuframe.com/` (e.g. `https://share.vuframe.com/abcd1234`).