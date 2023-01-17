<!-- START_METADATA
---
title: Quick start
sidebar_position: 20
---
END_METADATA -->

# Quick start

Use the Login API to confirm your identity through the log-in process.

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

## Table of Contents

* [Postman](#postman)
  * [Prerequisites](#prerequisites)
  * [Step 1: Get the Vipps Postman collection and environment](#step-1-get-the-vipps-postman-collection-and-environment)
  * [Step 2: Import the Vipps Postman files](#step-2-import-the-vipps-postman-files)
  * [Step 3: Set up Postman environment](#step-3-set-up-postman-environment)
* [Make API calls](#make-api-calls)
  * [Vipps Login in Browser](#vipps-login-in-browser)
  * [Vipps Login from phone number (CIBA without redirect)](#vipps-login-from-phone-number-ciba-without-redirect)
  * [Vipps Login from phone number (CIBA with redirect)](#vipps-login-from-phone-number-ciba-with-redirect)
  
<!-- END_COMMENT -->

## Postman

### Prerequisites

Review
[Vipps quick start guides](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/quick-start-guides)
for information about getting your test environment set up.

### Step 1: Get the Vipps Postman collection and environment

Save the following files to your computer:

* [Vipps Login API Postman collection](tools/vipps-login-api.postman_collection.json)
* [Vipps API Global Postman environment](https://raw.githubusercontent.com/vippsas/vipps-developers/master/tools/vipps-api-global-postman-environment.json)

### Step 2: Import the Vipps Postman files

1. In Postman, click *Import* in the upper-left corner.
1. In the dialog that opens, with *File* selected, click *Upload Files*.
1. Select the two files you have just downloaded and click *Import*.

### Step 3: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
1. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
1. Ensure that you have the correct values for the following fields:
   * `client_id` - Client ID for the merchant (the "username").
     See [Common topics: API Keys](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/common-topics/api-keys#getting-the-api-keys) for details.
   * `client_secret` - Client Secret for the merchant (the "password").
   * `well-known_uri` - URL to well-known endpoint for used environment.
      In the test environment, you can use <https://apitest.vipps.no/access-management-1.0/access/.well-known/openid-configuration>.
      See [API Guide: well known](vipps-login-api#openid-connect-discovery-endpoint) for more details.
   * `redirect_uri` - The URL where the user is sent after finishing a login.
      The URL must be exactly the same as the one specified on [portal.vipps.no](https://portal.vipps.no) in the _Setup login_ section for your sale unit.
      For testing, you could, for example, use <http://localhost> in both places.
   * `mobileNumber` - Your 8 digit mobile number for your test version of Vipps. This is only used for the CIBA flows.

## Make API calls

### Vipps Login in Browser

1. Send request `Get OIDC well-known`.
1. In your active Postman environment, copy the value of key `start_login_uri`, and use this URL in any browser.
1. Finish Vipps login. This request includes a scope parameter that requests access to user information.
   This means that if you have not yet consented to sharing your user information, a distinct screen asking for your consent will appear the first time.
1. When you finish your login, the website will update.
If you used <http://localhost>, it will appear to be broken, but that doesn't matter.
Copy the query param `code` from the URL in the browser. Paste this code into the key `code` in the active Postman environment.
1. From the `Vipps Login in Browser` folder, send request `Get token`. This provides the access token and id token.
1. Send request `Get user info` to get the user info of the logged-in user.

### Vipps Login from phone number (CIBA without redirect)

1. Send request `Get OIDC well-known`. This will populate the environment variables `init_ciba_endpoint`, `token_endpoint` and `userinfo_endpoint` used in subsequent requests.
2. From the `CIBA Without Redirect` folder, send request to `Init CIBA no-redirect`. This will trigger a push message to the Vipps app registered with the `mobileNumber` in your environment. This request will populate the environment variable `auth_req_id` used for the subsequent token request.
3. Authenticate in the Vipps app and approve the login request.
4. Send request to `Token no-redirect` endpoint. This will populate the environment variable `access_token` used for the subsequent request.
5. Send request to `Get user info`. This will use the token from (4) to obtain the userinfo of the logged-in user.

### Vipps Login from phone number (CIBA with redirect)

1. Send request `Get OIDC well-known`. This will populate the environment variables `init_ciba_endpoint`, `token_endpoint` and `userinfo_endpoint` used in subsequent requests
1. From the `CIBA With Redirect` folder, submit the `Init CIBA redirect`. This will trigger a push message to the Vipps app registered with the `mobileNumber` in your environment.
1. Authenticate in the Vipps app and approve the login request. Your mobile will now open the URL specified by `redirect_uri` (default <http://localhost>) with a `code` query parameter after successful authentication. It doesn't matter if the page appears to be broken, you only need the `code` value from the address.
1. Copy/take note of the `code` parameter in the query string that you were returned to in the above step. This code has 300s time-to-live in test environment and 30s time-to-live in production environment.
1. Set the value of the `code` parameter in the body of the `Token redirect` request to the code you obtained in the previous step.
1. Submit the  `Token redirect` request. This will populate the environment variable `access_token` used for the subsequent request.
1. Send request to `Get user info`. This will use the token from the previous step to obtain the user info of the logged-in user.

See the
[API reference](https://vippsas.github.io/vipps-developer-docs/api/login)
for details about the calls.
