## The FormAssembly API

### Introduction

[FormAssembly.com](https://formassembly.com) is a web-based form management solution that allows anyone to easily create online forms and collect data. FormAssembly is available in different flavors, cloud-hosted (SaaS) or on-premises, and all editions provide a secure REST API for interacting with user accounts and exporting data.

Before you can start building an application using the FormAssembly API, check out the [FormAssembly Developer Hub](https://www.formassembly.com/api/) for information on how to register, obtain a CLIENT_ID, and get access to a sandbox.

***

### Endpoint

The **endpoint**, in API parlance, is the destination URL for your API requests. The endpoint will vary depending on the FormAssembly edition you're interacting with.

| Edition  | Endpoint | Note |
| -------- | -------- | ---- |
| FormAssembly Developer Sandbox | https://developer.formassembly.com/api_v1/ | For development and testing only |
| FormAssembly.com | https://app.formassembly.com/api_v1/ | |
| FormAssembly Essentials, Enterprise, Teams, Government | https://instance_name.tfaforms.net/api_v1/   | Adjust `instance_name` as needed |

From this point on, we'll use the FormAssembly.com endpoint. Just remember to adjust the URIs to match your own FormAssembly edition.

***

### Registration

Before you can interact with the API, you must obtain a `CLIENT_ID` and a `CLIENT_SECRET` code. This is done by completing the registration process on the particular instance you are targeting. 
+ For the Developer Sandbox and FormAssembly.com, you must register through the [FormAssembly Developer Hub](https://www.formassembly.com/api/). 
+ For Essentials, Team, Enterprise, and Government instances, the FormAssembly administrator can register your app [here](https://github.com/FormAssembly/formassembly-api#self-register-your-app-on-an-existing-enterprise-instance).

***

### Authorization

**Authorization** is the mechanism that lets FormAssembly users decide which applications can access their account through the API. Authorizations are handled using the [OAuth2](https://oauth.net/2/) protocol.

If you're not familiar with OAuth2 interactions, it can be helpful to look at the documentation of other APIs, [such as this one for the Google API](https://developers.google.com/accounts/docs/OAuth2), explaining how OAuth2 works in detail.

The key point of OAuth2 is that it allows a **user** of FormAssembly (let's call them Adam), to authorize a **client** (you) to take actions in FormAssembly as if you were Adam. For example, you could write an application, which when authorized by Adam, could download and display a list of all of Adam's forms along with associated metrics, e.g., number of responses, dropout rates, etc.  This data could then be used by you to generate custom reports for Adam.

***

### Step-by-Step Outline

To request a user's authorization, your application must follow these steps.

#### 1. Redirect the User to the Authorization URL

https://YOUR_FORMASSEMBLY_INSTANCE_URL/oauth/login?type=web&client_id=CLIENT_ID&redirect_uri=RETURN_URL&response_type=code 

  + `CLIENT_ID`: The unique client ID generated by FormAssembly for your client application (see [Registration](#registration) above).
  + `RETURN_URL`: The URL that your user will be returned to when they complete the authorization step on FormAssembly. This value must be URL-encoded.  

![authorize](https://user-images.githubusercontent.com/7387512/46624911-1fdfcf80-cb00-11e8-9052-4fa6845fa10c.png) 

#### 2. Retrieve the Authorization Code

FormAssembly will append the **code** query string parameter to your RETURN_URL when returning the user to the URL. For instance, `redirect_uri=https%3A%2F%2Fwww.yourapp.com` (note that the value is URL-encoded) will result in the user being taken to `https://www.yourapp.com?code=XXXXXXXXXX` after authorizing FormAssembly.

Retrieve the code from the query string. You will need it to obtain the access token.

![retrieve code](https://user-images.githubusercontent.com/7387512/46616121-d5eaef80-cae7-11e8-920a-f70fd57ddd8c.png)

#### 3. Request an Access Token

Execute a HTTP request *from your server* (do *not* direct the user) to request an **access token**.

 + URL: https://YOUR_FORMASSEMBLY_INSTANCE_URL/oauth/access_token
 + HTTP Method: POST
 + Post Data:


 | Parameter Name | Value  | Comment
 | ---------- | ---------- | ---------- |
 | grant_type | authorization_code | Use this value as-is | 
 | type | web_server | Use this value as-is |
 | client_id | CLIENT_ID | Use your unique client application ID issued by FormAssembly |
 | client_secret | CLIENT_SECRET | Use your unique client application secret issued by FormAssembly |
 | redirect_uri | RETURN_URL | The same `RETURN_URL` used in [Step 1](#1-redirect-the-user-to-the-authorization-url) |
 | code | CODE | The code parameter you obtained in [Step 2](#2-retrieve-the-authorization-code) |


The response to this request will be a JSON array, like so:
 
    { "access_token":"XXXXXXXXXXXXXX",
      "expires_in":XXXXX,
      "scope":XXXX,
      "refresh_token":"XXXXXXXXXXXX" }

![get token](https://user-images.githubusercontent.com/7387512/46624917-253d1a00-cb00-11e8-86e1-b1ef960c1985.png)

Store the `access_token` value. You'll need it for every API request on the user account. Since the access token is valid for only one user account, you will typically obtain multiple access_token from multiple users, and you must store and manage those tokens accordingly.

The `refresh_token` is used in a similar way to other OAuth2 implementations - it's used to obtain a renewed access token. Expiration time for access tokens is set to 10 years. [Here](https://auth0.com/learn/refresh-tokens/) is a bit more documentation on refresh tokens.



***

### Example

####  Request

A HTTP GET request to: 

https://app.formassembly.com/api_v1/forms/index.json?access_token=ACCESS_TOKEN

The URL is composed of the following parts:

 + `https://app.formassembly.com/`: The FormAssembly instance.
 + `api_v1`: The API version in use. As changes are made to the API, new versions can be released (e.g., under api_v2, api_v3, etc.).
 + `forms/index`: The requested data (see [API Reference](#api-reference) below).
 + `json`: The requested format for the response.
 + `ACCESS_TOKEN`: The token received in [Step 3](#3-request-an-access-token) of the authorization process.


#### Response 

The response from the request above will result in a JSON-formatted list of the forms in the user's account:

    {"Forms":[{
    "Form":{
    "id":"XXXX", "version_id":"XXXX", "name":"XXXXXXXX",
    "category":"XXXX", "subcategory":"XXXX", "is_template":"XXXX",
    "display_status":"XXXX", "moderation_status":"XXXX", "expired":"XXXX",
    "use_ssl":"XXXX", "user_id":"XXXX",
    "created":"XXXXXXXX", "modified":"XXXXXXXX",
    "Aggregate_metadata":{
    "id":"XXXX", "response_count":"XXXX", "submitted_count":"XXXX",
    "saved_count":"XXXX", "unread_count":"XXXX", "dropout_rate":"XXXX",
    "average_completion_time":"XXXX", "is_uptodate":"XXXX"
        }}
    }]}

For an explanation of each field, please see [Object Reference](#object-reference).

Several output formats are accepted. See [Formats](#formats) for more details.

***

### API Reference

#### Formats

FormAssembly supports returning data in two main formats:

  + [json](https://en.wikipedia.org/wiki/Json): A lightweight data exchange format supported by newer languages.  Directly parsable by JavaScript.
  + [xml](https://en.wikipedia.org/wiki/XML): An industry standard data exchange format, parsable by almost all languages.

Some endpoints support additional formats, including:
  + [plist](https://en.wikipedia.org/wiki/Plist): Used to provide Apple consumable data for Objective-C applications.
  + [csv](https://en.wikipedia.org/wiki/Comma-separated_values): Used as a standard record data exchange format.
  + [zip](https://en.wikipedia.org/wiki/ZIP_(file_format\)): A binary data container format.

***

#### Forms [Returned Fields Reference]

##### Admin Index (Enterprise Plan only)
 + https://app.formassembly.com/admin/api_v1/forms/index.json
 + https://app.formassembly.com/admin/api_v1/forms/index.xml

Returns a list of all forms in the FormAssembly instance. Only accessible if using FormAssembly Enterprise and an access token from an admin-level user.

##### Index
+ https://app.formassembly.com/api_v1/forms/index.json
+ https://app.formassembly.com/api_v1/forms/index.xml

Returns a list of the forms in the user's account, along with associated metadata.

##### View
 + https://app.formassembly.com/api_v1/forms/view/#FORMID#.json
 + https://app.formassembly.com/api_v1/forms/view/#FORMID#.xml

##### Additional Parameters
+ `raw`: Bypass form XML element whitelisting.

Return an encoded copy of the form's definition in XML or JSON.  `Raw` parameter will cause 
the returned output to be exactly the content stored by FormAssembly including obsolete or 
internal fields.  Generally you should not use the `raw` parameter.

##### Create
 + https://app.formassembly.com/api_v1/forms/create.json
 + https://app.formassembly.com/api_v1/forms/create.xml

##### Additional Parameters (POST only)
+ `xml_data`: xml data formatted per FormAssembly's schema.  See output of api_v1/forms/view.
+ `builder_version`: string identifier, e.g. "3.4.2","4.0.0","4.0.1","4.1.0","4.2.0".
+ `language`: iso language code, e.g. "en-US","fr","zh-CN"
+ `name`: string text of the preferred internal application display name.

Create a new form.  Expects the additional parameters listed above to be sent as POST.

##### Edit
 + https://app.formassembly.com/api_v1/forms/edit/#FORMID#.json
 + https://app.formassembly.com/api_v1/forms/edit/#FORMID#.xml

##### Additional Parameters (POST only)
+ `id`: id for existing form.
+ `xml_data`: xml data formatted per FormAssembly's schema.  See output of api_v1/forms/view.
+ `builder_version`: string identifier, e.g. "3.4.2","4.0.0","4.0.1","4.1.0","4.2.0".
+ `language`: iso language code, e.g. "en-US","fr","zh-CN"
+ `name`: string text of the preferred internal application display name.

Send update to form code.  Expects the additional parameters listed above to be sent as POST.

##### Delete
 + https://app.formassembly.com/api_v1/forms/delete/#FORMID#.json
 + https://app.formassembly.com/api_v1/forms/delete/#FORMID#.xml

##### Additional Parameters (POST only)
+ `id`: id for existing form.

Delete form.  Expects the additional parameters listed above to be sent as POST.



##### Code Examples

 + PHP: https://github.com/formassembly/formassembly-api/blob/master/php/fa_api.php
 + Python (command-line): https://github.com/formassembly/formassembly-api/blob/master/python/fa_api.py
 + Bash (command-line): https://github.com/formassembly/formassembly-api/blob/master/curl/fa_api.sh
 + Salesforce: https://github.com/drewbuschhorn/sfdc-oauth-playground/tree/oauth2_drew

***

#### Responses [Returned Fields Reference]

##### Export
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.csv
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.json
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.xml
+ https://app.formassembly.com/api_v1/responses/export/#FORMID#.zip

additional parameters:
+ date_from: start date for export range
+ date_to: end date for export range
+ filter: if set to 'all', export will include both completed and incomplete responses. If not present, only completed responses will be returned.
+ response_ids: set of comma delimited response ids to retrieve
+ flag: starred is `1`, unstarred is `` (blank)
+ unlock: For Enterprise instance, `1` to unlock sensitive data fields from report.


##### Examples
+ https://app.formassembly.com/api_v1/responses/export/1.csv?date_from=01/01/2012&date_to=01/01/2013&filter=all
  * would retrieve all responses (including incompletes) created between January 1, 2012 and January 1, 2013.
+ https://app.formassembly.com/api_v1/responses/export/1.xml?response_ids=10,11,12
  * would retrieve only responses with IDs: 10, 11, 12.

***

#### Connectors

##### Index
+ https://app.formassembly.com/api_v1/connectors/index/#FORMID#.json
+ https://app.formassembly.com/api_v1/connectors/index/#FORMID#.xml

Returns a list of the connectors associated with form #FORMID#.

##### View
 + https://app.formassembly.com/admin/api_v1/connectors/view/#CONNECTORID#.json
 + https://app.formassembly.com/admin/api_v1/connectors/view/#CONNECTORID#.xml

Return an encoded copy of the connectors's definition in XML or JSON. #CONNECTORID# can be retrieved from the api_v1/connectors/index/#FORMID# call.

##### Create
 + https://app.formassembly.com/admin/api_v1/connectors/create/#FORMID#/#CONNECTORNAME#.json
 + https://app.formassembly.com/admin/api_v1/connectors/create/#FORMID#/#CONNECTORNAME#.xml

##### Additional Parameters (POST only)
+ `event`: string, indicating the stage the connector is run: "beforerender" - before form is displayed to user,"before_save" - when form is saved by user, "interactive" - after form is completed, "background" - after user is shown the form's thank-you page. 

Create a new connector.  Expects the additional parameters listed above to be sent as POST. #CONNECTORID# can be retrieved from the api_v1/connectors/index/#FORMID# call.

##### Edit
 + https://app.formassembly.com/admin/api_v1/connectors/edit/#CONNECTORID#.json
 + https://app.formassembly.com/admin/api_v1/connectors/edit/#CONNECTORID#.xml

##### Additional Parameters (POST only)
+ `mapping`: data to set as connector mapping.
+ `login`: login data for the connector service.
+ `password`: password data for the connector service.

Send update to connector code.  Expects the additional parameters listed above to be sent as POST.

##### Delete
 + https://app.formassembly.com/admin/api_v1/connectors/delete/#CONNECTORID#.json
 + https://app.formassembly.com/admin/api_v1/connectors/delete/#CONNECTORID#.xml

Delete connector.

***

#### Themes

##### Index
+ https://app.formassembly.com/api_v1/themes/index.json

Returns a list of the themes associated user account.

##### View
 + https://app.formassembly.com/admin/api_v1/themes/view/#THEMEID#.json

Return the theme's definition in raw CSS. #THEMEID# can be retrieved from the api_v1/themes/index/#FORMID# call.  Note this data may contain links to local urls that will not match a new environment.

##### Create
 + https://app.formassembly.com/admin/api_v1/themes/create.json

##### Additional Parameters (POST only)
+ `name`: string, indicating name of theme
+ `css_data` : string, CSS payload for the theme

Create a new theme.  Expects the additional parameters listed above to be sent as POST.

##### Edit
 + https://app.formassembly.com/admin/api_v1/themes/edit/#THEMEID#.json

##### Additional Parameters (POST only)
+ `id`: int, id of theme to be edited. 
+ `name`: string, indicating name of theme
+ `css_data` : string, CSS payload for the theme

Send update to theme code.  Expects the additional parameters listed above to be sent as POST.

##### Delete
 + https://app.formassembly.com/admin/api_v1/themes/delete/#THEMEID#.json

Delete the theme.

***

#### Form Elements ( in flux, may change without warning )

##### Index
+ https://app.formassembly.com/api_v1/form_elements/index.xml

Returns a list of the form elements available to the user.

##### View
 + https://app.formassembly.com/admin/api_v1/form_elements/view/#ELEMENTID#.xml

Return an XML copy of the element's definition in raw CSS. #ELEMENTID# can be retrieved from the api_v1/form_elements/index.xml call.  Note this data may contain links to local urls that will not match a new environment.

##### Create
 + https://app.formassembly.com/admin/api_v1/form_elements/create.json

##### Additional Parameters (POST only)
+ `xml_data`: string, XML payload for FormAssembly FormBuilder
+ `name` : string, name for element
+ `comments` : string, TBD
+ `category` : string, system level category, (leave blank by default)
+ `subcategory` : string, user defined category
+ `batch` : int, TBD - leave blank

Create a new form element.  Expects the additional parameters listed above to be sent as POST.

##### Edit
 + https://app.formassembly.com/admin/api_v1/form_elements/edit/#ELEMENTID#.json

##### Additional Parameters (POST only)
+ `id`: int, id of element to be edited. 
+ `xml_data`: string, XML payload for FormAssembly FormBuilder
+ `name` : string, name for element
+ `comments` : string, TBD
+ `category` : string, system level category, (leave blank by default)
+ `subcategory` : string, user defined category
+ `batch` : int, TBD - leave blank

Send update to element code.  Expects the additional parameters listed above to be sent as POST.

##### Delete
 + https://app.formassembly.com/admin/api_v1/form_elements/delete/#THEMEID#.json

Delete the element.

***

#### Aggregates

##### Reset_counters
 + https://app.formassembly.com/admin/api_v1/aggregates/reset_counters/#FORMID#.json

Trigger a reset of the aggregate data counters (unread, read, saved, etc.) that may be out of sync after import runs.

***

#### Object Reference

##### Form

Object | Description | Example
---: | --- | :---
"id":"XXXX" | Unique integer value identifying the form within the FormAssembly instance. Every form has a single unique ID in the form of an integer. Can be used to construct a valid form URL, e.g., https://app.formassembly.com/forms/view/XXXX | 1
"version_id":"XXXX" | Unique integer ID identifying the current version (revision) of the form. | 1
"name":"XXXXXXXX" | HTML-encoded string representing the form's name as displayed in the user's FormAssembly form index list. Not to be confused with the Form Title, found in the form's XML definition. | "Mine &amp; Yours Form"
"category":"XXXX", | String representing one of the system-wide default form organizational categories. Can be an empty string for uncategorized forms. | "Contact Forms"
"subcategory":"XXXX", | String representing one of the user-created organizational categories. Can be an empty string for uncategorized forms. | "IBM Contact Forms"
"is_template":"XXXX", | Integer specifying whether or not the form is shared as a template publicly. Contact Support for more details.<br/><br/>**Values**<br/>`0`: Not a template<br/>`>1`: Is a template | 0 
"display_status":"XXXX", |Integer specifying whether or not the form is active or archived.<br/><br/>**Values**<br/>`0`:  Archived<br/>`2`: Active | 2
"moderation_status":"XXXX", | Integer specifying whether or not the form is moderated (under review for suspicious content).<br/><br/>**Values**<br/>`0`: Not checked<br/>`2`: Reviewed and approved<br/>`3`: Reviewed and denied | 0
"use_ssl":"XXXX", | Integer representing if the form must be displayed over HTTPS. If true, form URL must contain `https://`. | |
"created":"XXXXXXXX" | String timestamp representing the date the form was created. | "created":"1983-01-01 23:59:59"
"modified":"XXXXXXXX" | String timestamp representing the last time the form or any of its settings were modified. | "modified":"2012-06-17 17:50:12"
"expired":"XXXX" | String timestamp representing the date the form was marked for deletion, if any. | "expired":null

***

### Self-Register Your App on an Existing Enterprise Instance

If you are developing for an existing Essentials, Team, Enterprise, and Government instances, you will need to self-register your app by following the instructions below.

1. Login to your Enterprise account as an admin.

2. Navigate to the **Admin Dashboard** tab. <div></div> ![Step1](content/AdminDashboardMenu.png) 

3. In the Admin Dashboard, navigate to **Settings > Third Party Apps** on the sidebar menu.  <div></div> ![Step2](content/SettingThirdParty.gif)

4. Click "Register a new application." <div></div> ![Step3](content/RegisterNewApplication.png)

5. Pick a name for your new application, and take note of your OAuth credentials. <div></div> ![Step4](content/ApplicationTitle.gif)

***

### FAQ

#### 1. How can we access file uploads from a form?
The user can use the export zip API call to obtain the file upload path then append this to the instance name.

e.g https://app.formassembly.com/api_v1/responses/export/5040999.zip

which returns uploads/get_zip/a2124d28e77f7b2da80ed12301cfcb17-form_5040999.zip,

then appended to instance will download the upload file. 

https://app.formassembly.com/uploads/get_zip/a2124d28e77f7b2da80ed12301cfcb17-form_5040999.zip

#### 2. What happens with fields marked as sensitive? 
Fields marked as sensitive will show as redacted when you access the endpoint until you log into the FormAssembly account and unlock the report.

#### 3. Can the Response PDF be sent through the API? 
No

#### 4. Can we prefill data to the form using the FormAssembly API?
No

#### 5. Is there something to get a token via Oauth without a web based sign in?
API call from Postman APP

#### 6. Is our API token user based or app based?
User based

#### 7. How can someone have it pull data on a time schedule?
There is no scheduling functionality, To accomplish this the user can use third-party scheduling applications that will call [API](https://github.com/FormAssembly/formassembly-api#export) based on their preferred schedule or create their own application.

#### 8. Can we pull data from workflow in the API?
No