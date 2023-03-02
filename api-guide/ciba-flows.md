<!-- START_METADATA
---
title: API guide
sidebar_position: 30
---
END_METADATA -->

# Integrating with Vipps Login from phone number

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

Vipps Login from phone number is available for all Vipps Login enabled clients.

Vipps Login from phone number (CIBA flows) has been developed to support use cases where authentication/registration does not start in a browser or an app. These flows are described [here](overview.md#vipps-login-from-phone-number). They are based on the CIBA OIDC standard <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html>.

A sales unit can use both Vipps Login in browser and Vipps Login from phone number. It is recommended to use the same sales unit for all use cases to ensure that you, as a merchant, get the same user id ('sub') on the user across different scenarios. To ensure a consistent user experience on webpages and in apps, it is not allowed to use Vipps Login from phone number for such use cases.

## Complete login in the Vipps app

### Overview

Client-Initiated Backchannel Authentication (CIBA) enables a Client to initiate the authentication of an end-user through out-of-band mechanisms.

1) The Client shall make an "HTTP POST" request to the Backchannel Authentication Endpoint to ask for end-user authentication.
2) Vipps Login will respond immediately with a unique identifier that identifies that authentication while it tries to authenticate the user in the background.
3) The Client will receive the ID Token and Access Token by polling the token endpoint to get a response with the tokens.

### Call by call

0. Before all this, the merchant has fetched the openid configuration from the well-known endpoint and cached it.
   See [.well-known](integration.md#openid-connect-discovery-endpoint)

1. The merchant initiates a login by calling the `backchannel_authentication_endpoint` listed in the openid configuration fetched in step 0.

   For details see [Authentication Request](#authentication-request).

   Example request:

    ```http
    POST https://api.vipps.no/vipps-login-ciba/api/backchannel/authentication
    Authorization: Basic asdkjhasdjhsad=
    Content-Type: application/x-www-form-urlencoded

    scope=name address openid&login_hint=urn:mobilenumber:{mobileNumber}&state=13821s837213bng26e2n61gege26&nonce=21hebdhwqdb7261bd1b23
    ```

   Example response:

    ```http
    200 application/json
    {
      "auth_req_id": "VYGaaAMRkI6SyAm_uIywhxsN2K0",
      "expires_in": 600,
      "interval": 5
    }
    ```

2. The merchant starts polling the `token` endpoint listed in the openid configuration fetched in step 0.

   Polling in this context means doing repeated http requests with a delay between them.

   [Information about polling](#polling). Note that the polling interval should adhere to the `interval` response parameter (in seconds) returned in step 1.

   For other details about the request, see [Token request](#token-request).

   Example request:

    ```http
    POST https://api.vipps.no/access-management-1.0/access/oauth2/token
    Authorization: Basic asdkjhasdjhsad=
    Content-Type: application/x-www-form-urlencoded

    grant_type=urn%3Aopenid%3Aparams%3Agrant-type%3Aciba&auth_req_id=VYGaaAMRkI6SyAm_uIywhxsN2K0
    ```

   Example pending response (Other possible error responses can be found in the [CIBA standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.11)):

    ```http
    HTTP/1.1 400 Bad Request
    Content-Type: application/json;charset=UTF-8
    {
      "error": "authorization_pending",
      "error_description": "The authorization request is still pending"
    }
    ```

   The merchant should keep polling when it receives `authorization_pending` in the error response. After the user completes the login in the app, the `token` endpoint will give a successful response similar to the following example:

    ```http
    HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8

    {
      "access_token": "ciba.W_IfBcSr-askdjhsakjhd",
      "token_type": "Bearer",
      "expires_in": 300,
      "id_token": "eyaksjdhksajhdjkashdjksadjnn91283hedhn.eyasdkjhaskjdhskajhdkjhasdkjhaskjhdwqiuh"
    }
    ```

3. The merchant must do a GET to the `userinfo` endpoint with the header: Authorization: Bearer {access_token}, using the access_token retrieved in step 2.

   For details see [Userinfo request](integration.md#userinfo).

   Example request:

    ```http
    GET https://api.vipps.no/vipps-userinfo-api/userinfo
    Authorization: Bearer ciba.W_IfBcSr-askdjhsakjhd
    ```

   Example response:

    ```http
    HTTP/1.1 200 OK

    {
      "address": {
        "address_type": "home",
        "country": "NO",
        "formatted": "jghj khhjhhkjh\n0603\nOSLO\nNO",
        "postal_code": "0603",
        "region": "OSLO",
        "street_address": "jghj khhjhhkjh"
      },
      "family_name": "Heyerdahl",
      "given_name": "Tor Fos",
      "name": "Tor Fos Heyerdahl",
      "other_addresses": [],
      "sid": "qwieuhwqiuhdiuwqh",
      "sub": "f350ef33-22e2-47d0-9f47-12345667"
    }
    ```

### Authentication Request

Standard definition: <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request>

The Backchannel Authentication Endpoint is listed as `backchannel_authentication_endpoint` in the configuration <https://api.vipps.no/access-management-1.0/access/.well-known/openid-configuration>.

#### Authentication

The following authentication methods are currently supported:

* client_secret_basic
* client_secret_post

The default token endpoint authentication method is `client_secret_basic`. It is possible to change the authentication method to `client_secret_post` in the Vipps portal. [More information in the FAQ](../vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication).

#### The `login_hint` parameter (required)

Supported login hints:

* Norwegian mobile phone numbers can be targeted by passing login hint's on the format `urn:mobilenumber:{8 digit norwegian mobile number}`.

Example: `...&login_hint=urn:mobilenumber:12345678&...`.

#### The `scope` parameter (required)

* We support the scopes listed at [Scopes](core-concepts.md#scopes)
* The legacy `nnin` scope is not supported, use `nin` instead.

Example: `...&scope=name address birthDate nin&...`

#### The `binding_message` parameter (optional)

A human-readable identifier or message intended to be displayed on both the consumption device and the authentication device to interlock them together for the transaction by way of a visual cue for the end-user. It should not be used for attempting to conveying other information.

#### Format

The format possible for this field is limited to capital characters 'A-Z', numbers '0-9' and the character '-'. It must also be between 5 and 8 characters long. Regex: ```^[A-Z0-9\\-]{5,8}$```.

Read more about it in the standard <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request>.

Example: `....&binding_message=4MZ-CQ3&...`

#### Successful responses

Standard definition: <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#successful_authentication_request_acknowdlegment>

Responses according to the standard. Note we do return an `interval` parameter which indicates the minimum amount of time in seconds that the Client MUST wait between polling requests to the token endpoint.

### Token request

Standard definition: <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.10.1>

The responses from this endpoint is according to the standard.

* Note the required `grant_type`: `urn:openid:params:grant-type:ciba`.
* The access token can be used towards the standard [oidc userinfo endpoint](integration.md#userinfo).

#### Polling

* Long polling as described in [the CIBA standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#token_request) is currently not supported
* Remember not to poll more often than indicated by the `interval` parameter, returned from the [authentication request](#authentication-request).

### Error responses

In addition to the responses defined by the [standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.11) these responses might be returned:

* `429` status responses: Too many login requests started towards the same user at the same time. Please respect the `Retry-After` header returned.
* `error_code=old_app`: The user's Vipps app is outdated and does not support this login flow.
* `error_code=invalid_user`: No account exists, the user's account is not active or the user is in some way not eligible to use this login flow currently e.g. U15 users.

## Redirect to browser

### Overview

This CIBA-related flow enables a Client to initiate the authentication of an end-user through out-of-band mechanisms and additionally facilitates the end user
to be taken to the client's web page to finalize the flow.

1) The Client shall make an "HTTP POST" request to the Backchannel Authentication Endpoint to ask for end-user authentication.
2) The user, via their browser, will be redirected to the Client's `redirect_uri` which enables the login to be completed.


``` mermaid
sequenceDiagram
    participant Merchant
    participant VippsLogin as Vipps Login
    participant VippsApp as Vipps app
    participant Browser
    actor User

    Merchant ->> VippsLogin: POST /authentication-request (scope, login_hint, redirect_uri...)
    User ->> VippsApp: Confirm login
    VippsApp ->> Browser: Opens browser
    Browser ->> Merchant: `redirect_uri`
    Merchant ->> VippsLogin: POST /oauth2/token
    Merchant ->> VippsLogin: GET /userinfo
```


### Call by call

0. Before all this, the merchant has fetched the openid configuration from the well-known endpoint and cached it.
   See [.well-known](integration.md#openid-connect-discovery-endpoint).

1. The merchant initiates a login by calling the `backchannel_authentication_endpoint` listed in the openid configuration fetched in step 0.

   For details see [Authentication Request With Redirect](#authentication-request-with-redirect).

   Example request (the real payload will likely look different because of encoding):

    ```http
    POST https://api.vipps.no/vipps-login-ciba/api/backchannel/authentication
    Authorization: Basic asdkjhasdjhsad=
    Content-Type: application/x-www-form-urlencoded

    requested_flow=login_to_webpage&scope=openid name address&login_hint=urn:mobilenumber:{mobileNumber}&redirect_uri=https://merchantwebpage.com/callback
    ```

   Example response, the `auth_req_id` should be used to connect this login to a token response since the ID token should contain the same `auth_req_id` value.

    ```http
    200 application/json
    {
      "auth_req_id": "VYGaaAMRkI6SyAm_uIywhxsN2K0",
      "expires_in": 600,
      "interval": 5
    }
    ```

2. The user confirms the login and is then redirected to the `redirect_uri` passed in the initial request 1. The redirect will contain a `code`: `{redirect_uri}?code={code}`.

3. The merchant uses the code-parameter to obtain the login token. Perform a POST request towards the `{token_endpoint}` with `code={code}`, `grant_type=urn:vipps:params:grant-type:ciba-redirect` in the `application/x-www-form-urlencoded-body`.
   This returns an ID token and an access token that can be used to fetch userinfo.
   The ID token is a JWS that must be validated, see [ID Token](core-concepts.md#id-token). The merchant **must validate** that it contains the `auth_req_id` they have previously received from step 2.

   Example request (the real payload will likely look different because of encoding):

    ```http
    POST https://api.vipps.no/access-management-1.0/access/oauth2/token
    Authorization: Basic sadlksadkjasjdaksd
    Content-Type: application/x-www-form-urlencoded

    code=some-valid-code&grant_type=urn:vipps:params:grant-type:ciba-redirect

    ```

   Example response:

    ```json
    {
      "access_token": "hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus",
      "id_token": "eyJraWQiOiJwdWJsaWM6ZWUzNmQzZjUtMzkzNC00MDI5LTkyNmYtNzdmYTY1YmYwYjRiIiwiYWxnIjoiRVMyNTYifQ.eyJhdWQiOiJlZGRkYjMyZi01MDI4LTQzOTctYjBhYi1lOGVjZjIxOGZkYzIiLCJzdWIiOiI1MTY4ZWUwNi04NzFlLTQ2ZTYtOTQxZS0wMTAzYjk1NzA0OGUiLCJhdXRoUmVxSWQiOiI3QnpBWS1TYlZRSjM4Vi1VMEM3WjZrMjNfQ1kiLCJpc3MiOiJodHRwczpcL1wvZWNlNDZlYzQtNmY5Yy00ODliLThmZTUtMTQ2YTg5ZTExNjM1LnRlY2gtMDIubmV0XC9hY2Nlc3MtbWFuYWdlbWVudC0xLjBcL2FjY2Vzc1wvIiwiZXhwIjoxNjQzMTc5ODM3LCJpYXQiOjE2NDMxNzkyMzd9.iFvmdtRQVliAe91dBu_CZDfBD5I7WCbDTiDQxu4sOTApXFPb5EsSuEBEVfK_-14E7xjcfQLSMa6ZO06YvhRHAA",
      "expires_in": 3599,
      "scope": "openid",
      "token_type": "bearer"
    }
    ```

   Decoded ID token JWS example:

   `Header`
    ```json
    {
      "kid": "public:ee36d3f5-3934-4029-926f-77fa65bf0b4b",
      "alg": "ES256"
    }
    ```
   `Payload`
    ```json
    {
      "aud": "edddb32f-5028-4397-b0ab-e8ecf218fdc2",
      "sub": "5168ee06-871e-46e6-941e-0103b957048e",
      "auth_req_id": "7BzAY-SbVQJ38V-U0C7Z6k23_CY",
      "iss": "https://ece46ec4-6f9c-489b-8fe5-146a89e11635.tech-02.net/access-management-1.0/access/",
      "exp": 1643179837,
      "iat": 1643179237
    }
    ```

4. The merchant must do a GET  to the `userinfo` endpoint with the header: Authorization: Bearer {access_token}, using the access_token retrieved in step 3.

   For details see [Userinfo request](integration.md#userinfo).

   Example request:

    ```http
    GET https://api.vipps.no/vipps-userinfo-api/userinfo
    Authorization: Bearer W_IfBcSr-askdjhsakjhdasdfgg
    ```

   Example response:

    ```http
    HTTP/1.1 200 OK

    {
      "address": {
        "address_type": "home",
        "country": "NO",
        "formatted": "jghj khhjhhkjh\n0603\nOSLO\nNO",
        "postal_code": "0603",
        "region": "OSLO",
        "street_address": "jghj khhjhhkjh"
      },
      "family_name": "Heyerdahl",
      "given_name": "Tor Fos",
      "name": "Tor Fos Heyerdahl",
      "other_addresses": [],
      "sid": "qwieuhwqiuhdiuwqh",
      "sub": "f350ef33-22e2-47d0-9f47-12345667"
    }
    ```

### Authentication Request With Redirect

The Backchannel Authentication Endpoint is listed as `backchannel_authentication_endpoint` in the configuration <https://api.vipps.no/access-management-1.0/access/.well-known/openid-configuration>.

#### Authentication

The following authentication methods are currently supported:

* client_secret_basic
* client_secret_post

The default token endpoint authentication method is `client_secret_basic`. It is possible to change the authentication method to `client_secret_post` in the Vipps portal. [More information in the FAQ](../vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication).

Required parameters: `requested_flow`, `login_hint`, `scope`, `redirect_uri`

#### The `login_hint` parameter (required)

Supported login hints:

* Norwegian mobile phone numbers can be targeted by passing login hint's on the format `urn:mobilenumber:{8 digit norwegian mobile number}`.

Example: `...&login_hint=urn:mobilenumber:12345678&...`.

#### The `scope` parameter (required)

* We support the scopes listed at [Scopes](core-concepts.md#scopes).
* The legacy `nnin` scope is not supported, use `nin` instead.
* The `openid` scope is required

Example: `...&scope=openid name address birthDate nin&...`

#### The `binding_message` parameter (optional)

A human-readable identifier or message intended to be displayed on both the consumption device and the authentication device to interlock them together for the transaction by way of a visual cue for the end-user.

Read more about it in the standard <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request>

Note: "the binding_message value SHOULD be relatively short and use a limited set of plain text characters"

Example: `....&binding_message=4MZ-CQ3&...`

#### The `redirect_uri` parameter (required)

Redirect URL which the user agent is redirected to after finishing a login. Must be `https` in the production environment.

Example: `...&redirect_uri=https://merchant.com/callback&...`

#### Error responses

In addition to the responses defined by [the standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.13) these responses might also be returned:

* `429` status responses: Too many login requests started towards the same user at the same time. Please respect the `Retry-After` header returned.
* Most of the [general error codes](integration.md#error-handling)


## Merchants delegated consents

### Activation

See [How can I get started with delegatedConsents?](../vipps-login-api-faq.md#how-can-i-get-started-with-delegatedconsents).

### Overview

For Client-Initiated Backchannel Authentication (CIBA) it is possible to initiate the authentication of an end-user through out-of-band mechanisms and collect consents on behalf of the merchant.

1) The Client shall make an "HTTP POST" request to the Backchannel Authentication Endpoint with scope `delegatedConsents` to ask for end-user authentication and on behalf of the merchant consents.
2) Vipps Login will respond immediately with a unique identifier that identifies that authentication while it tries to authenticate the user in the background, along with collecting consents.
3) The Client will receive the ID Token and Access Token by polling the token endpoint to get a response with the tokens.
4) Token can be used by the Client to retrieve information about the user through the `userinfo` endpoint, this response will also contain the consents that are approved/declined by the user.

Requesting on behalf consents for merchant is also available for [Vipps login from phone number with redirect to browser](#redirect-to-browser), by adding scope delegatedConsents to the initial request (1).

### Call by call

0. Before all this, the merchant has fetched the openid configuration from the well-known endpoint and cached it.
   See [.well-known](#openid-connect-discovery-endpoint)

1. The merchant initiates a login by calling the `backchannel_authentication_endpoint` listed in the openid configuration fetched in step 0 with scope `delegatedConsents`.

   For details see [Authentication Request](#authentication-request).

   Example request:

    ```http
    POST https://api.vipps.no/vipps-login-ciba/api/backchannel/authentication
    Authorization: Basic asdkjhasdjhsad=
    Content-Type: application/x-www-form-urlencoded

    scope=name delegatedConsents openid&login_hint=urn:mobilenumber:{mobileNumber}&state=13821s837213bng26e2n61gege26&nonce=21hebdhwqdb7261bd1b23
    ```

   Example response:

    ```http
    200 application/json
    {
      "auth_req_id": "VYGaaAMRkI6SyAm_uIywhxsN2K0",
      "expires_in": 600,
      "interval": 5
    }
    ```

2. The merchant starts polling the `token` endpoint listed in the openid configuration fetched in step 0.

   Polling in this context means doing repeated http requests with a delay between them.

   [Information about polling](#polling). Note that the polling interval should adhere to the `interval` response parameter (in seconds) returned in step 1.

   For other details about the request, see [Token request](#token-request).

   Example request:

    ```http
    POST https://api.vipps.no/access-management-1.0/access/oauth2/token
    Authorization: Basic asdkjhasdjhsad=
    Content-Type: application/x-www-form-urlencoded

    grant_type=urn%3Aopenid%3Aparams%3Agrant-type%3Aciba&auth_req_id=VYGaaAMRkI6SyAm_uIywhxsN2K0
    ```

   Example pending response (Other possible error responses can be found in the [CIBA standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.11)):

    ```http
    HTTP/1.1 400 Bad Request
    Content-Type: application/json;charset=UTF-8
    {
      "error": "authorization_pending",
      "error_description": "The authorization request is still pending"
    }
    ```

   The merchant should keep polling when it receives `authorization_pending` in the error response. After the user completes the login in the app, the `token` endpoint will give a successful response similar to the following example:

    ```http
    HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8

    {
      "access_token": "ciba.W_IfBcSr-askdjhsakjhd",
      "token_type": "Bearer",
      "expires_in": 300,
      "id_token": "eyaksjdhksajhdjkashdjksadjnn91283hedhn.eyasdkjhaskjdhskajhdkjhasdkjhaskjhdwqiuh"
    }
    ```

3. The merchant must do a GET to the `userinfo` endpoint with the header: Authorization: Bearer {access_token}, using the access_token retrieved in step 2.

   For details see [Userinfo request](#userinfo).

   Example request:

    ```http
    GET https://api.vipps.no/vipps-userinfo-api/userinfo
    Authorization: Bearer ciba.W_IfBcSr-askdjhsakjhd
    ```

   Example response:

    ```http
    HTTP/1.1 200 OK

    {
      "family_name": "Heyerdahl",
      "given_name": "Tor Fos",
      "name": "Tor Fos Heyerdahl",
      "other_addresses": [],
      "sid": "qwieuhwqiuhdiuwqh",
      "sub": "f350ef33-22e2-47d0-9f47-12345667"
      "delegatedConsents" : {
        "language" : "EN",
        "heading" : "Give consent",
        "text" : "oidc-testclient wants to send you relevant offers through their digital channels.",
        " termsDescription" : "By confirming, you accept oidc-testclient's terms of membership. They are responsible for processing the consents. You can withdraw them at any time on their website.",
        "confirmConsentButtonText" : "Confirm",
        "links" : {
          "termsLinkText" : "Read the terms of use.",
          "termsLinkUrl" : "https://www.vipps.no/vilkar/vilkar-privat/",
          "privacyStatementLinkText" : "Read the privacy policy.",
          "privacyStatementLinkUrl" : "https://www.vipps.no/vilkar/cookie-og-personvern/"
        },
        "timeOfConsent" : "2022-11-23T11:40:13Z",
        "consents" : [ {
          "id" : "email",
          "accepted" : true,
          "required" : true,
          "textDisplayedToUser" : "Receive offers via email"
        }, {
          "id" : "sms",
          "accepted" : true,
          "required" : false,
          "textDisplayedToUser" : "Receive offers via SMS"
        }, {
          "id" : "digital",
          "accepted" : true,
          "required" : false,
          "textDisplayedToUser" : "I would like to receive digital marketing"
        }, {
          "id" : "personal",
          "accepted" : true,
          "required" : false,
          "textDisplayedToUser" : "Get customized offers"
        } ]
      }
    }
    ```

### Authentication request
Required parameters are listed in [Authentication request login from phone number](#authentication-request) and [Authentication request login from phone number with redirect](#authentication-request-with-redirect)

#### Error responses
Error responses are listed in [Error responses login from phone number](#error-responses) and [Error responses login from phone number with redirect](#error-responses-1)

#### Successful responses
Success responses for Vipps login from phone number are listed in [Successful responses login from phone number](#successful-responses)

##### Token request
Token request for Vipps login from phone number is described in [Token request login from phone number](#token-request)

##### Polling
Polling Vipps login from phone number is described in [Polling login from phone number](#polling)

