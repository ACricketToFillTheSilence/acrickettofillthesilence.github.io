<!--# Removing Auth Tokens from URL Parameters-->
## High-Level Description

The way that API calls are authenticated is changing on 31 Jan 2022. 
Placing auth tokens in the URL parameters pose a significant security risk. 
The change is being requested by Fiserv. 
Developers will need to move from using URL parameters to using Authorization headers when sending in JSON files. 
As they do so, they will need to move any client-side API calls to become server-side calls because Clover's CORS protocols do not allow JSON files authenticated with the Auth headers to be sent in from front-end API calls.

This high-level view will be unpacked below.

## Background

When a third-party developer sends in a request for data, they do so using JSON files. The HTTP for JSON includes many things; the two important elements here are the body and the header.

![]()

The body includes all the data going back and forth between computers. 
If we're creating a Customer, for instance, the body would include details like the customer's name, phone number, email address, and birthday.

The header includes external information, such as:

- What kind of file should be returned from the request (i.e. the dev should be asking for a JSON file because that's the file format we use)
- Cookies
- Required authentication keys

The JSON body uses braces (`{}`) and key-value pairs to organize data. 
Braces surround each JSON object. 
Key-value pairs help regularize the data between the computers sharing information. 
If a developer wants to create a customer with a specific email address, for example, they would use the following key-value pair:

```
{
  'emailAddress': 'abc@clover.com'
}
```
When a developer interacts with the Clover REST API, they call an endpoint (a specific kind of URL meant for sending and retrieving data).

Historically, developers have been able to authenticate their JSON files by placing the auth token either in the header or as a variable in the URL (called a URL parameter). 
Here is a sample call using an Authorization header: 

```
https://apisandbox.dev.clover.com/v3/merchants/<<merchantUuid>>/customers/
```

Because the header belongs to HTTP, it uses different formatting.

Header:

```
Authorization: Bearer <<authToken>>
Cookie: _ga=GA1.2.400426771.1588356098; _hp2_id.12
22pageviewId%22%3A%222946885056905627%22%2C%22ses
22trackerVersion%22%3A%224.0%22%7D; wm-cseu-id=%22
be50-46e3-88f6-dbe2e4166503%22; wm_ct_0_a3ae97b8b
[{%22t%22:175445%2C%22td%22:null%2C%22c%22:0%2C%2
```

Body:

```
{
  "emailAddress": "abc123@clover.com"
}
```



If the developer decided that they'd rather send in the auth token using a URL parameter, the URL would become this: 

```
https://apisandbox.dev.clover.com/v3/merchants/<<merchantUuid>>/customers/?access_token=<<authToken>>
```

## Security Concerns

For security reasons, Fiserv is requiring that we remove auth tokens from URL parameters. 
We will no longer allow URL parameter authentication because it gives too much information away for comfort. 
Because our auth tokens include

* information about the app,
* access to a specific merchant's information,
* no expiration date,

the security risk is fairly large when a bad actor finds a token.

We sent out a communication on 30 Jul 2021 informing developers of the coming changes. 
Any devs making calls with URL parameters will need to update their calls so that they use Auth headers by 31 Jan 2022. 
We have given them six months' warning to prepare for this change.

## CORS Conflict

When we first scoped this project, we ran into an issue with CORS: developers who used CORS would be unable to use Authorization headers and would therefore be put in a no-win situation. 
We addressed this issue and Authorization headers should now work in conjunction with CORS.

## Developer FAQs

### Why is Clover removing the access_token query parameter option?
Authorization via URL query parameter can be riskier from a security standpoint. URLs are more public than headers and can be captured in logs.

### Other places use URL authorization. Why isn't it secure for Clover?
URL authorization is more secure when using one-time access tokens or other tokens that expire quickly. Clover tokens are currently too long-lived for us to be comfortable continuing to offer URL authorization.

### What will happen if an app is still using URL authorization once the access_token parameter is sunset?
Any API request that still uses the access_token query parameter after the sunset date will return a 401 Unauthorized response.

### "By the end of January" is a little vague. When's the actual sunset date?
(This has been updated. If developers still mention the "end of January" date from our earlier communications, you can let them know there's been an extension to provide them a little more time.)

Our current timeline is as follows:

- **January 12, 2022**: access_token sunset in Sandbox
- **February 28, 2022**: access_token sunset in Production

### How can I switch to approved authorization methods?
Ensure you are using server-side requests as outlined in our Using API tokens documentation. Instead of using the access_token URL parameter, use the Authorization header on your requests.

### I use CORS and can't use Authorization headers. How can my app continue to make requests after this deprecation goes through?
We identified issues with using the Authorization header with CORS. Our engineering teams have deployed a fix to address this in all environments now. You should be good to go ahead and use Authorization: Bearer headers on your CORS requests.

### Your docs/example app/YouTube video/other resource still uses the access_token parameter! Why's that?
Thank you for bringing this to our attention. We're in the process of updating our developer resources and will make sure this one is on the list.

### Can I be an exception for this deprecation?
We are not able to offer case-by-case exceptions for this deprecation. 
If you feel you would be negatively impacted by this deprecation or would not be able to meet the cutoff date, we'd be interested in hearing details about your use case. 
We will let our engineering and security teams know about your situation to help inform their product decisions.

## Internal FAQs

### How many developers will be affected by this change?
We were able to find 251 unique apps using the `access_token` parameter over a month's timespan. 
There may be more developers using merchant tokens that we're unaware of.

### How can I tell if a specific developer is affected by this change?
You can check in Kibana whether any of the developer's apps have used the `access_token` URL parameter recently. 
To do so, you can:

1. Grab the app's ID; you can insert the app's UUID into this SQL statement and run it to find the app ID:
  ```
  select id from developer_app where uuid = "APP_UUID";
  ```
2. Use this Kibana query as a template: {URL Removed}.
3. Select the `mdc.developerAppId` filter.
4. Select **edit filter**.
5. Replace the example value with the application ID from the first step.

It's not foolproof and developers should still test their apps thoroughly to check for `access_token` usage, but this can provide a good at-a-glance idea on whether a developer will be affected.
