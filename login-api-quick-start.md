---
title: Login Quick start
sidebar_label: Quick start
sidebar_position: 20
description: Quick steps for getting started with the Login API.
toc_min_heading_level: 2
toc_max_heading_level: 5
pagination_next: null
pagination_prev: null
---

import ApiSchema from '@theme/ApiSchema';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Quick start

Use the Login API to confirm your identity through the log-in process.

## Before you begin

The provided example values in this guide must be changed with the values for your sales unit and user.
This applies for API keys, HTTP headers, reference, phone number, etc.


## Log in with browser

### Step 1 - Setup

You must have already signed up as an organization with Vipps MobilePay and have
your test credentials from the merchant portal.

In addition, your sales unit must be set up for login.
See [Knowledge base: How to set up Login for your sales unit](https://developer.vippsmobilepay.com/docs/developer-resources/portal/#how-to-set-up-login-for-your-sales-unit).

You will need the following values, as described in the
[Getting started guide](https://developer.vippsmobilepay.com/docs/getting-started):

* `client_id` - Client_id for a test sales unit.
* `client_secret` - Client_secret for a test sales unit.
* `Ocp-Apim-Subscription-Key` - Subscription key for a test sales unit.
* `merchantSerialNumber` - The unique ID for a test sales unit.
* `internationalMobileNumber` - The MSISDN for the test app profile you have received or registered. This is your test mobile number including country code.
* `redirect_uri` - The website to send the user to after they log in.
   This must be exactly the same redirect URI as the one you specified on your sales unit in the merchant portal.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

**Please note:** To prevent your sensitive data and credentials from being synced to the Postman cloud,
store them in the *Current Value* fields of your Postman environment.

In Postman, import the following files:

* [Login API Postman collection](/tools/vipps-login-api.postman_collection.json)
* [Global Postman environment](pathname:///tools/vipps-api-global-postman-environment.json)

🔥 **To reduce risk of exposure, never store production keys in Postman or any similar tools.** 🔥

Update the *Current Value* field in your Postman environment with your **Merchant Test** keys.
Use *Current Value* field for added security, as these values are not synced to the cloud.

</TabItem>
<TabItem value="curl">

No additional setup needed :)

</TabItem>
</Tabs>


### Step 2 - Get OIDC well-known endpoint

Get configuration information for OpenID Connect clients by using
[OpenID configuration endpoint](https://developer.vippsmobilepay.com/api/login/#tag/Login-API/operation/discoverOpenIDConfiguration).


<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send Get OIDC well-known
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/access-management-1.0/access/.well-known/openid-configuration \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X GET
```

</TabItem>
</Tabs>

The URL of the OpenID Provider's OAuth 2.0 Authorization Endpoint is provided in the response.


### Step 3 - Log in

Log the user in by using OpenID Connect.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

In your active Postman environment, copy the value of key `start_login_uri` and paste it into the address field of any browser.

</TabItem>
<TabItem value="curl">

Compose the URI in this format ([OAuth 2.0 Authorize](./api-guide/browser-flow-integration.md#oauth-20-authorize)):

```http
https://apitest.vipps.no/access-management-1.0/access/oauth2/auth?client_id=YOUR-CLIENT-ID&response_type=code&scope=openid%20name%20phoneNumber%20address%20birthDate&state=8652682f-ba1d-4719-b1ec-8694ba97bde7&redirect_uri=http://localhost
```

Paste the URL into the address field of any browser.

</TabItem>
</Tabs>

Finish the login.
If you have not yet consented to sharing your user information, a new screen will be presented in the app requesting your consent.

If you have already completed this process and selected *Remember me in browser* earlier, this will take you straight to the redirect URL.

### Step 4 - Get token

On the redirect URL page, copy the `code` value out from the address field in the URL.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">


Paste the code into the key `code` in the active Postman environment and then get the token.

```bash
Send request Get token
```

</TabItem>
<TabItem value="curl">

Use the `code` in the following command.
You will also need to generate client authorization.

The client credentials is a base64-encoded string consisting of the client_id and secret issued by Vipps.

Example in JavaScript:

```javascript
var client_id = 123456-test-4a3d-a47c-412136fd0871
var client_secret = testdzlJbUZaM1lqODlnUUtrUHI=

var wordArrayAzp = CryptoJS.enc.Utf8.parse(client_id + ":" + client_secret);
var client_authorization = CryptoJS.enc.Base64.stringify(wordArrayAzp);
```

```bash
curl https://apitest.vipps.no/access-management-1.0/access/oauth2/token \
-H 'Content-Type: application/x-www-form-urlencoded' \
-H 'Authorization: Basic {client credentials}' \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X POST \
--data-urlencode 'grant_type=authorization_code' \
--data-urlencode 'code=THE CODE FROM THE URL' \
--data-urlencode 'redirect_uri=http://localhost'
```

Copy the access token from the response.

</TabItem>
</Tabs>


### Step 5 - (Optional) Get userinfo

Send request `Get Userinfo`. This uses [`GET:/vipps-userinfo-api/userinfo/`][userinfo-endpoint-login].

Use the access token from the previous step.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get userinfo
```

</TabItem>
<TabItem value="curl">


```bash
curl https://apitest.vipps.no/vipps-userinfo-api/userinfo/ \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: YOUR-MSN" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X GET
```

</TabItem>
</Tabs>


## Next steps

See the [Login API guide](api-guide/README.md) to read about the concepts and details.

For more examples, see the step-by-step instructions in the
[Login API Postman guide](login-api-postman-guide.md).


[create-payment-endpoint]: https://developer.vippsmobilepay.com/api/epayment#tag/CreatePayments/operation/createPayment
[get-payment-endpoint]: https://developer.vippsmobilepay.com/api/epayment#tag/QueryPayments/operation/getPayment
[userinfo-endpoint]: https://developer.vippsmobilepay.com/api/userinfo#operation/getUserinfo
[userinfo-endpoint-login]: https://developer.vippsmobilepay.com/api/userinfo/#operation/userinfoAuthorizationCode
