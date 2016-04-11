#Overview

##Getting Started

<aside class="notice">SurveyMonkey recently released a v3 of our API! We've made some huge improvements with this version and we encourage you to check it out. If you already use our v2 API, you can find its documentation <a href="https://developer.surveymonkey.com/docs/overview/getting-started/">here</a>.</aside>

The SurveyMonkey API is REST-based, uses OAuth2 for authentication, and returns responses in JSON. To get started you will need to register an app in our developer portal.

To register an app:

1. Click on the **MY APPS** tab in the upper left corner and sign into your SurveyMonkey account. Your [SurveyMonkey plan](https://www.surveymonkey.com/pricing/?ut_source=dev_portal&amp;ut_source2=docs) determines which API features and scopes you can access.
1. Enter an existing **Mashery User Name** and **API Secret** or accept our [API Developer terms of service](https://developer.surveymonkey.com/tou/) and sign up for a new Mashery account.
1. Click **+ Add New App**. Enter your **App Name** and provide your app's **Redirect URI** and click **Create App**. Under **App Details** you will find an API key and secret you can use for OAuth and an access token for your own SurveMonkey account.
1. Click **Settings** to review and adjust scopes for your app. Scopes allow your application to access particular resources on behalf of a user. Based on your application's needs, you can choose to either require scopes or set them as optional. If certain scopes are required, users will have to have paid plans in order for Oauth to succeed. Click on scopes to toggle their requirements and click **Update Scopes** when finished.

SurveyMonkey lists code examples on [Github](https://github.com/SurveyMonkey) and monitors questions tagged as `surveymonkey` on [StackOverflow](http://stackoverflow.com/search?q=surveymonkey). If you have an SDK or example you would like added, let us know. We also offer limited email support for technical questions at [api-support@surveymonkey.com](mailto: api-support@surveymonkey.com).

###Scopes

Scopes allow your application to access particular resources on behalf of a user. For example, the Create/Modify surveys scope allows your application to create a survey in a user's account. During the OAuth process the user will approve or disapprove the scopes you have requested access to. Based on your application's needs you can choose to either require scopes, set them as optional, or not require them. All required scopes must be approved for the OAuth process to succeed.

Some scopes are only available to accounts on SurveyMonkey paid plans. If your application uses scopes tied to paid plans, any accounts authenticating with your application need that plan or higher. If your application will only access your own account, you will need to [upgrade to the appropriate paid plan](https://www.surveymonkey.com/pricing/?ut_source=dev_portal&amp;ut_source2=docs) for the scopes you intend to use.

Scopes also respect plan limits. See [plans](https://www.surveymonkey.com/pricing/?ut_source=dev_portal&amp;ut_source2=docs) for more information on plan limits. 


|Scope|Minimum SurveyMonkey plan|What the scope allows|
|-----|------------|-----------------------------|
|View Surveys|BASIC (Free)|The application can fetch a list of and the details for surveys the user has access to.|
|Create/Modify Surveys|SELECT|The application can create surveys, modify, or delete surveys on behalf of a user.|
|View Collectors|BASIC (Free)|The application can fetch collectors for surveys that the user has access to.|
|Create/Modify Collectors|BASIC (Free)|The application can create, modify, or delete survey collectors on behalf of the user.|
|View Contacts|BASIC (Free)|The application can fetch a user's contacts and contact lists.|
|Create/Modify Contacts|BASIC (Free)|The application can create, modify, or delete contacts in the user's account or contact lists.|
|View Responses|BASIC (Free)|The application can fetch survey responses.|
|View Response Details|GOLD|The application can fetch answer data for survey responses.|
|Create/Modify Responses|PLATINUM|The application can create, modify, or delete survey responses for surveys the user has access to.|
|View Webhooks|BASIC (Free)|The application can fetch a user's list of webhooks and their details.|
|Create/Modify Webhooks|BASIC (Free)|The application can create, modify, or delete webhooks on behalf of the user.|
|View Users|BASIC (Free)|The application can fetch a user's details.|
|View Groups|PLATINUM|The application can fetch a user's group details.|
|View Library Assets|BASIC (Free)|The application can fetch a user's library assets (survey themes and templates).|

##Authentication

The SurveyMonkey API supports OAuth 2.0. 

![OAuth 2](https://raw.githubusercontent.com/SurveyMonkey/public_api_docs/master/images/oauth_2.png)

If your application will only access your own SurveyMonkey account, you can use the access token, generated when you registered your app, as part of your application's configuration. Obtain this token in the **Settings** of your app in the [**MY APPS**](https://developer.surveymonkey.com/apps/) tab.

If your application will access many SurveyMonkey accounts, implement the OAuth 2.0 three-step flow outlined below to allow users to authorize your app to access their accounts. This flow generates a long-lived access token your application can use with every API call to the associated SurveyMonkey account. It's important to note that the access token only grants access when used in combination with your API credentials (API key and client ID) and only to the SurveyMonkey account which was authorized. Your application will need to obtain additional access tokens for each SurveyMonkey account you wish to access. 

If your application has required [scopes](#scopes), users may need a [paid SurveyMonkey plan](https://www.surveymonkey.com/pricing/?ut_source=dev_portal&amp;ut_source2=docs) to successfully Oauth into your application.   


####Step 1: Direct user to SurveyMonkey's OAuth authorization page

You applications should send the user whose SurveyMonkey account you wish to access to a specially crafted Oauth link at https://api.surveymonkey.net. The page presented to the user there will identify the application name you configured when you registered your application. Users will either be asked to enter their SurveyMonkey user name and password, or, if they are already logged into SurveyMonkey, just an "Authorize" button.

The OAuth link should be `https://api.surveymonkey.net/oauth/authorize` with urlencoded parameters: `redirect_uri`, `client_id`, `response_type`, and `api_key`. 

* `response_type` will always be set to the value `code`
* `client_id` your Mashery user name
* `redirect_uri` URL encoded OAuth redirect URI you registered for your application (can be found and edited [here](https://developer.surveymonkey.com/apps/))

>Example OAuth Link

```shell
https://api.surveymonkey.net/oauth/authorize?response_type=code&redirect_uri=https%3A%2F%2Fapi.surveymonkey.com%2Fapi_console%2Foauth2callback&client_id=SurveyMonkeyApiConsole&api_key=u366xz3zv6s9jje5mm3495fk
```


```python
SM_API_BASE = "https://api.surveymonkey.net"
AUTH_CODE_ENDPOINT = "/oauth/authorize"

def oauth_dialog(client_id, redirect_uri, api_key):
	url_params = urllib.urlencode({
		'redirect_uri': redirect_uri,
		'client_id': client_id,
		'response_type': 'code',
		'api_key': api_key
	})

	auth_dialog_uri = SM_API_BASE + AUTH_CODE_ENDPOINT + '?' + url_params
	print "\nThe OAuth dialog url was " + auth_dialog_uri + "\n"

	# Insert code here that redirects user to OAuth Dialog url
```

####Step 2: User authorization generates short lived code

Once the user makes their choice whether to authorize access or not, SurveyMonkey will generate a 302 redirect sending their browser to your redirect URI along with a short-lived code included as a query parameter. Your application needs to use that code to make another API request before it expires (5 minutes). In that request, you will send us the code you received along with your client secret, client ID, API key, and redirect URI. We will verify all that information. If it's good, we will return a long-lived access token in exchange.

>Generate Short Code or Deny Access

```shell
"Access Authorized" redirect: `https://api.surveymonkey.com/api_console/oauth2callback?code=SHORTLIVEDCODE`
"Access Denied" redirect: `https://api.surveymonkey.com/api_console/oauth2callback?error_description=Resource+owner+canceled+the+request&error=access_denied`
```


```python
def handle_redirect(redirect_uri):
	# Parse authorization code out of url
	query_string = urlparse.urlsplit(redirect_uri).query
	authorization_code = urlparse.parse_qs(query_string).get('code', [])

	# parse_qs returns a list for every query param, just get the first one
	if not authorization_code:
		return

	return authorization_code[0]
```

####Step 3: Exchanging for a long-lived access token

Create a form-encoded HTTP POST request to `https://api.surveymonkey.net/oauth/token?api_key=YOUR_API_KEY` with the following encoded form fields: `client_secret`, `code`, `redirect_uri` and `grant_type`. The grant type must be set to "authorization_code". The `client_secret` and `api_key` can be found [here](https://developer.surveymonkey.com/apps/).

If successful, the access token will be returned encoded as JSON in the response body of your POST request. The key will be `access_token` and the value can be passed to our API as an HTTP header in the format `Authorization: bearer YOUR_ACCESS_TOKEN`. The value of the header must be "bearer" followed by a single space and then your access token.


>Exchange for long-lived token

```shell
curl -i -X POST https://api.surveymonkey.net/oauth/token?api_key=YOUR_API_KEY -d \
	"client_secret=YOUR_CLIENT_SECRET \
	&code=AUTH_CODE \
	&redirect_uri=YOUR_REDIRECT_URI \
	&client_id=YOUR_CLIENT_ID \
	&grant_type=authorization_code"
```

```python
SM_API_BASE = "https://api.surveymonkey.net"
ACCESS_TOKEN_ENDPOINT = "/oauth/token"

def exchange_code_for_token(auth_code, api_key, client_secret, client_id, redirect_uri):
	data = {
		"client_secret": client_secret,
		"code": auth_code,
		"redirect_uri": redirect_uri,
		"client_id": client_id,
		"grant_type": "authorization_code"
	}

	access_token_uri = SM_API_BASE + ACCESS_TOKEN_ENDPOINT + '?api_key=' + api_key
	access_token_response = requests.post(access_token_uri, data=data)
	access_json = access_token_response.json()

	if 'access_token' in access_json:
		return access_json['access_token']
	else:
		print access_json
		return None
```

####Keeping the access token safe

Since the access token is long-lived, you can store it in your database and reuse it so you don't need to ask the user to authorize access each time they use your application. Take care to keep access tokens secret. Since your API key and client ID are easily obtained by inspecting the authorization link in your application, the access token would allow any user to gain access to the data in the associated SurveyMonkey account. If access tokens are compromised please contact API support at [api-support@surveymonkey.com](mailto:api-support@surveymonkey.com).

####Token expiration and revocation

Our access tokens do not currently expire but may in the future. We will warn all developers before making changes.

Access tokens can be revoked by the user. If this happens, you will get a JSON-encoded response body including a key `status`with a value of `1` and a key `errmsg` with the value of `Client revoked access grant` when making an API request. If you get this response, you will need to complete OAuth again.

####Unauthorizing an app

To unauthorize an app: 

1. Log into the linked SurveyMonkey account 
2. Select **My Account** from the username dropdown in the upper right
3. Scroll to **Linked Accounts** and click **Remove** next to the app you want to unauthorize

##Request and Response Limits 

You can [upgrade your SurveyMonkey account](https://www.surveymonkey.com/pricing/?ut_source=dev_portal&amp;ut_source2=docs) to get higher request limits. When you upgrade to a higher plan, it can take up to an hour for your new request limit to take effect.

####Request Limits for SurveyMonkey Plans

Plan | Max Requests Per Second | Max Requests Per Day
------- | ------- | -------
BASIC (free) | 2 | 1,000
SELECT | 4 | 5,000
GOLD | 6 | 7,000
PLATINUM/ENTERPRISE | 10 | 15,000

####Response Limits

We also have global response limits for database utilization fair use.

Property | Limit
------- | -------
Max Page Size | 1000 unless otherwise specified
Max Survey Size | 1000 questions, surveys over limit will return a 413

####Increasing Limits

If you think you may be in danger of exceeding your request-limit threshold, please [fill out this form](https://surveymonkey.wufoo.com/forms/apis-advanced-access-request/) with an estimate of your anticipated API activity. We may be able to increase your limit to meet your application's needs.

##Error Codes

|Error Code|HTTP Status Code|Message|
|----------|----------------|-------|
|1000|400 Bad Request|Unable to process the request with the provided input.|
|1001|400 Bad Request|The body provided was not a proper JSON string.|
|1002|400 Bad Request|Invalid schema in the body provided.
|1003|400 Bad Request|Invalid URL parameters.|
|1004|400 Bad Request|Invalid request headers.|
|1010|401 Authorization Error|The authorization token was not provided.|
|1011|401 Authorization Error|The authorization token provided was invalid.|
|1012|401 Authorization Error|The authorization token provided has expired.
|1013|401 Authorization Error|Client revoked access to the authorization token provided.|
|1014|403 Permission Error|Permission has not been granted by the user to make this request.|
|1015|403 Permission Error|The user does not have the required plan to make this request.|
|1020|404 Resource Not Found|There was an error retrieving the requested resource.|
|1025|409 Resource Conflict|Unable to complete the request due to a conflict. Check the settings for the resource.
|1026|409 Resource Conflict|The requested resource already exists|
|1030|413 Request Entity Too Large|The requested entity is too large, it can not be returned.|
|1050|500 Internal Server Error|Oh bananas! We couldn't process your request.|
|1051|503 Internal Server Error|Service unreachable. Please try again later.|
​
​