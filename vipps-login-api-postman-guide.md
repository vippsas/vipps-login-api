<!-- START_METADATA
---
title: Login API extended Postman guide
sidebar_label: Postman guide
sidebar_position: 190
description: Extensive examples of the Login API using Postman.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Login API extended Postman guide

This is a postman guide for the using the Login API with Postman.
Use the Login API to confirm your identity through the log-in process.

This is an extension of the [Login API quick start guide](vipps-login-api-quick-start.md) which shows the
recommended flow in both Postman and curl.

## Prerequisites

You must have already signed up as an organization with Vipps MobilePay and have
your test credentials from the merchant portal, as described in the
[Getting started guide](https://developer.vippsmobilepay.com/docs/getting-started).


Your test sales unit must be set up for using login.
See [How to set up login on your sales unit](https://developer.vippsmobilepay.com/docs/developer-resources/portal#how-to-setup-login-on-your-sales-unit).

## Set up

Import the following files into Postman:

* [Login API Postman collection](/tools/vipps-login-api.postman_collection.json)
* [Global Postman environment](https://raw.githubusercontent.com/vippsas/vipps-developers/master/tools/vipps-api-global-postman-environment.json)

In Postman, tweak the environment with your own values (see
[API keys](https://developer.vippsmobilepay.com/docs/common-topics/api-keys/)):

* `client_id` - Merchant key required for getting the access token.
* `client_secret` - Merchant key required for getting the access token.
* `Ocp-Apim-Subscription-Key` - Merchant subscription key.
* `merchantSerialNumber` - Merchant ID.
* `mobileNumber` - The phone number for the test app profile you have received or registered.
* `well-known_uri` - URL to well-known endpoint for used environment.
   See [API Guide: well known](api-guide/integration.md#openid-connect-discovery-endpoint) for more details.
* `redirect_uri` - The URL where the user is sent after finishing a login.
   The URL must be exactly the same as the one specified in your sale unit.
   See [How to set up login on your sales unit](/docs/developer-resources/portal#how-to-setup-login-on-your-sales-unit).

For help using Postman, see
[Quick start guides](https://developer.vippsmobilepay.com/docs/quick-start-guides).

## Standard Login API calls

### Login in Browser

1. Send request `Get OIDC well-known`.
1. In your active Postman environment, copy the value of key `start_login_uri`, and use this URL in any browser.
1. Finish login. This request includes a scope parameter that requests access to user information.
   This means that if you have not yet consented to sharing your user information, a distinct screen asking for your consent will appear the first time.
1. When you finish your login, the website will update.
If you used <http://localhost>, it will appear to be broken, but that doesn't matter.
Copy the query parameter `code` from the URL in the browser. Paste this code into the key `code` in the active Postman environment.
1. From the `Vipps Login in Browser` folder, send request `Get token`. This provides the access token and ID token.
1. Send request `Get user info` to get the user info of the logged-in user.

### Login from phone number (CIBA without redirect)

1. Send request `Get OIDC well-known`. This will populate the environment variables `init_ciba_endpoint`, `token_endpoint` and `userinfo_endpoint` used in subsequent requests.
1. From the `CIBA Without Redirect` folder, send request to `Init CIBA no-redirect`. This will trigger a push message to the Vipps app registered with the `mobileNumber` in your environment. This request will populate the environment variable `auth_req_id` used for the subsequent token request.
1. Authenticate in the Vipps app and approve the login request.
1. Send request to `Token no-redirect` endpoint. This will populate the environment variable `access_token` used for the subsequent request.
1. Send request to `Get user info`. This will use the token from (4) to obtain the userinfo of the logged-in user.

### Login from phone number (CIBA with redirect)

1. Send request `Get OIDC well-known`. This will populate the environment variables `init_ciba_endpoint`, `token_endpoint` and `userinfo_endpoint` used in subsequent requests
1. From the `CIBA With Redirect` folder, submit the `Init CIBA redirect`. This will trigger a push message to the Vipps app registered with the `mobileNumber` in your environment.
1. Authenticate in the Vipps app and approve the login request. Your mobile will now open the URL specified by `redirect_uri` (default <http://localhost>) with a `code` query parameter after successful authentication. It doesn't matter if the page appears to be broken, you only need the `code` value from the address.
1. Copy/take note of the `code` parameter in the query string that you were returned to in the above step. This code has 300s time-to-live in test environment and 30s time-to-live in production environment.
1. Set the value of the `code` parameter in the body of the `Token redirect` request to the code you obtained in the previous step.
1. Submit the  `Token redirect` request. This will populate the environment variable `access_token` used for the subsequent request.
1. Send request to `Get user info`. This will use the token from the previous step to obtain the user info of the logged-in user.

## Partner API calls

### As a partner: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
1. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
1. Ensure that you have the correct values for the following fields:
   * `client_id` - Client ID for the partner.
   * `client_secret` - Client Secret for the partner.
   * `well-known_uri` - URL to well-known endpoint for used environment.
     In the test environment, you can use <https://apitest.vipps.no/access-management-1.0/access/.well-known/openid-configuration>.
     See [API Guide: well known](api-guide/integration.md#openid-connect-discovery-endpoint) for more details.
   * `redirect_uri` - The URL where the user is sent after finishing a login.
   * `mobileNumber` - Your 8 digit mobile number for your test version of Vipps. This is only used for the CIBA flows.
   * `targetMerchantSerialNumber` - The target merchant you want to log in towards.

### As a partner: Login from phone number (CIBA without redirect)

1. Send request `Get OIDC well-known`. This will populate the environment variables `init_ciba_endpoint`, `token_endpoint` and `userinfo_endpoint` used in subsequent requests.
1. Using requests in the folder `Partner login/CIBA without redirect`
1. Send request `Get partner token`
1. Send request `Start login`. This will trigger a push message to the Vipps app registered with the `mobileNumber` in your environment. This request will populate the environment variable `auth_req_id` used for the subsequent token request.
1. Authenticate in the Vipps app and approve the login request.
1. Send request `Get login token`. This will populate the environment variable `access_token` used for the subsequent request.
1. Send request `Get user info` stored in the root folder. This will use the token from (6) to obtain the userinfo of the logged-in user.

### As a partner: Login from phone number (CIBA with redirect)

1. Send request `Get OIDC well-known`. This will populate the environment variables `init_ciba_endpoint`, `token_endpoint` and `userinfo_endpoint` used in subsequent requests.
1. Using requests in the folder `Partner login/CIBA with redirect`
1. Send request `Get partner token`
1. Send request `Start login`. This will trigger a push message to the Vipps app registered with the `mobileNumber` in your environment. This request will populate the environment variable `auth_req_id` used for the subsequent token request.
1. Authenticate in the Vipps app and approve the login request. Your mobile will now open the URL specified by `redirect_uri` (default <http://localhost>) with a `code` query parameter after successful authentication. It doesn't matter if the page appears to be broken, you only need the `code` value from the address.
1. Copy/take note of the `code` parameter in the query string that you were returned to in the above step. This code has 300s time-to-live in test environment and 30s time-to-live in production environment.
1. Set the value of the `code` parameter in the body of the `Get login token` request to the code you obtained in the previous step.
1. Send request `Get login token`. This will populate the environment variable `access_token` used for the subsequent request.
1. Send request `Get user info` stored in the root folder. This will use the token from (8) to obtain the userinfo of the logged-in user.

See the
[API reference](https://developer.vippsmobilepay.com/api/login)
for details about the calls.
