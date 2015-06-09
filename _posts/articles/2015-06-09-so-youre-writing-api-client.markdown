---
layout: post
title: So You're Writing Api Client
modified:
categories: articles
excerpt:
tags: ['api-client', 'RESTful api client', 'Best practices for api client']
image:
  feature:
comments: true
image:
  feature: "api-client.png"
  credit: "robert-drummond"
  creditlink: "http://www.robert-drummond.com/2013/05/08/how-to-build-a-restful-web-api-on-a-raspberry-pi-in-javascript-2/?ak_action=reject_mobile"
share: true
author: sahilsk
date: 2015-06-09T00:55:29+05:30
---

With the advent of internet of things, especially rise of mobile devices has made modern web application to be served as SaaS: Software as a servic or simple service.

Now browser only don't account for traffic surge but mobile applications as well. Few companies takes this to a new level by providing their service as a platform for other developer to write their own application above it. Thereby, rise in traffic is inevitable.

With start of `internet of things` API layer has become norm. This post is about writing clients that will consume API ie. API Clients.

If you're writing API client first thing you need to decide if you want to show http response code to users or not?
If not, then how will intimate user about errors? use custom error code. This is also good.

However, there is a better way: best of both world: Put both of them. Put status to align your response code with [http status codes](http://en.wikipedia.org/wiki/Http_error_codes)
I personally feel attaching HTTP status code with every response is a nice way of enriching api-client response. 

``` json
{
	data: "(optional) My data goes here",
	"error": "(optional) My error,if any, goes here"
}
```

Looking at above response consumer will only learn partial truth. It doesn't say anything about api server response. This response block might work well with small number of api endpoints where errors could be made familiar after a very short usage, but doesn't scale well beyond it. 

Let's enrich our response block with little few more fields.

### Starting with errors first

Let's start with an example :

[Twitter api error response](https://dev.twitter.com/overview/api/response-codes) look like this:

``` json
{	
	"errors":[
		{
			"message":"Sorry, that page does not exist",
			"code":34
		}
	]
}
```

Excerpt from Twitter Api documentation:


>> If you see an error response which is not listed in the [twitter error code table](https://dev.twitter.com/overview/api/response-codes), then fall back to the HTTP status code in order to determine the best way to address the error.

This says we need http response code as well if error code is missing in api resopnse. So, in api client response there should be ** http status code **.


A simple informative api client response could be :

``` json
{
"status" : 401,
"message": "Authentication",
"code" : 2334,
"more_info" : "http://myApp.com/docs/errors/2334"
}
```

This response block shows where to go head next to get more information on error.

- `status`:  is aligned with http status code. 
- `Code`:   is used to add more information to this specific error.
- `more_info`: 
 In API design we always strive to be **verbose**.  There is no harm. You can also give link to more documentation by using `more_info` field in your response block.


More on `status`:

- Don't try to handle every http status codes. 

	You can start with 3 codes first: **200**(OK), **404**(Not Found) & **500**(Internal Server Error).
	Later on you can add more like following, if need arises.

	- `201` - Created
	- `304` - Not Modified
	- `400` - Bad Request
	- `401` - Unauthorized
	- `403` - Forbidden

###  Enriching response
Having dealt with errors we can now proceed further with returning good responses.

I personally like attaching following field to api response:

- `Status` - HTTP Status code
- `Data` (optional) - Any data returned
- `Error Code` (optional) - HTTP code associated with the error
- `Error Message` (optional) - Message associated with the error
- `Error Explanation`(optional) - Additional error info


Verbose? but verbosity is good otherwise developer has no way of knowing what are they doing wrong? 

You don't want them go bald by leaving them pulling their hair with no information but error code in response. Right? 


### Final response block

Combining all good parts together final response should include following fields:

- `status` : HTTP Status code
- `data`(optional) : Data retreived as a result of request. It could be an array or single object
- `errors`(optional):  Array of errors with `message`, `code` and `more_info` fields


``` json

{
	"status": <HTTP Status code >,
	"data": < Any data returned>,
	"errors" : [
		{
			"message": <short error description>,
			"code" : <error code >,
			"more_info" : < url to more info page"
		}
		...
	]
}
```

Isn't it __cleaner__, __informative__ and that __without loosing__ any peice of information.


References
------------


- http://stackoverflow.com/questions/942951/rest-api-error-return-good-practices
- https://blog.apigee.com/detail/restful_api_design_tips_for_handling_exceptional_behavior
- https://blog.apigee.com/detail/restful_api_design_what_about_errors
- https://dev.twitter.com/overview/api/response-codes