# Vipps Login API

API version: 1.0  

Document version 3.0.1.

## Table of contents
* [Introduction](#introduction)
  - [Activation](#activation)     
  - [Flows](#flows)
    * [Remembered flow](#remembered-flow)
    * [Phone number based flow - desktop](#phone-number-based-flow---desktop)
    * [Phone number based flow - mobile](#phone-number-based-flow---mobile)
    * [App-switch based flow - browser on mobile](#app-switch-based-flow---browser-on-mobile)   
* [Core concepts](#core-concepts)
  - [OAuth 2.0](#oauth-20)
  - [OpenID Connect](#open-id-connect)
  - [Supported OpenID Connect Flows](#supported-openid-connect-flows)
    - [Authorization Code Grant](#authorization-code-grant)
  - [Tokens](#tokens)
    * [ID Token](#id-token)
    * [Access Token](#access-token)
    * [Refresh Token](#refresh-token)
  - [Scopes](#scopes)
* [Recommendations](#recommendations)
* [Integrating with Vipps Login](#integrating-with-vipps-login)
    * [Manual integration](#manual-integration)
        * [Openid connect discovery endpoint](#openid-connect-discovery-endpoint)
        * [Openid connect discovery URLs](#openid-connect-discovery-urls)
            * [OAuth 2.0 Authorize](#oauth-20-authorize)
            * [OAuth 2.0 Token](#oauth-20-token)
            * [Userinfo](#userinfo)
            * [JSON Web Keys Discovery](#json-web-keys-discovery)
    * [Automatic Recognition](#automatic-recognition)
  * [API endpoints required from the merchant](#api-endpoints-required-from-the-merchant)
    * [Receive authentication result](#receive-authentication-result)
* [Error handling](#error-handling)
* [Questions and answers](#questions-and-answers)
* [Questions](#questions)

## Introduction

The Vipps Login API offers functionality for authenticating end users and authorizing clients founded on the OAuth2 and
OpenID Connect specifications. Login with Vipps is the easiest way to sign in and create an account. Users don’t need to worry about forgetting usernames and passwords. All they need to remember is their phone number. For an even smoother sign in experience, the user can choose to be remembered in the browser, enabling automatic sign-ins for later visits.

Users can create a new account through sharing high-quality data from the user’s Vipps profile. Available information includes name, email, addresses, phone number, and birth date. Norwegian national identity number is also available to [some merchants](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nnin).  The identity of all Vipps users is verified using BankID, Norway’s leading electronic ID, so rest assured that these are real people with correct name and information.

We offer free plugins for [Magento](https://github.com/vippsas/vipps-login-magento) and [WordPress/WooCommerce](https://github.com/vippsas/vipps-login-wordpress).

The Vipps login API authenticates the user in the web browser. The Vipps login API should only be run in the browser window using redirects (iFrame is not supported and new window is not recommended). For optimal performance the API should be opened in the main browser, also if you are integrating from an app. Some webviews in app will work functionally, but others (e.g Safari ViewController) will not work.

## Activation

See the FAQ:
[How can I activate and set up Vipps login?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login)

## Flows

### Remembered flow
If a user has chosen to be remembered in browser then the authentication can be completed in the browser. The user will then either be asked to provide consent to share profile information or be logged in directly.  This applies to both desktop and mobile.

If the user is not remembered the user needs to confirm the login in the Vipps-app. The flow associated with this will differ depending on the context where the user starts the login with Vipps (device, browser/app/wewbview):

### Phone number based flow - desktop
If the user is on desktop, and not remembered in browser, then the user will follow this flow.  If the user is remembered in browser then only the consent flow at the bottom will be completed. If the user already has provided consent then this step will be skipped also, allowing a direct login experience.

The user initiates the login by inputing the phone number and selecting whether to  be remembered in browser:
![Number input in desktop](../../images/Number_input_flow_desktop1.png)

The user goes to the Vipps app and confirms the login:
![Confirmation in app in phone number flow](../../images/Number_input_flow_app.png)

The user is then authenticated in browser and can provide consent if required. Then the user is redirected back to the redirect URI provided by merchant:
![Consent in desktop](../../images/Number_input_flow_desktop2.png	)

### Phone number based flow - mobile
This is the fallback flow for mobile devices if it is not possible to use the app-switch based flow. This is the same flow as for desktop above but please note that the user in this flow need to manually navigate back to the browser where they entered the phone number. If the user is remembered in browser the confirmation in app flow will be skipped.

![Phone number based flow - mobile](../../images/Mobile_flow_with_phone_number.png)

### App-switch based flow - browser on mobile
When the user starts to login with Vipps the browser automatically redirects the user to the Vipps-app, where login is confirmed and the user can choose not to be remembered in browser. After confirming in app the user is automatically redirected back to browser to finalise the autentication and provide consents if required. Then the user is redirected back to the redirect URI provided by merchant (can be webpage or app).

![Mobile flow with app-switch](../../images/Mobile_flow_with_app-switch.png)

The app-switch based flow is only supported when the Vipps login API is called in a browser (on iOS only Safari is supported). You can open the API in the browser from an app, but webview in app is not supported.  

Buttons to use for Vipps login can be found as part of our
[design guidelines](https://github.com/vippsas/vipps-design-guidelines/tree/master/vipps-buttons).

## Core concepts

### OAuth 2.0

[OAuth 2.0](https://tools.ietf.org/html/rfc6749) is the industry-standard
protocol for authorization. Giving a proper introduction to the standard is
out of the scope of this documentation, but there are many excellent resources
on the web. If you are new to the subject we recommend this
[talk](https://www.youtube.com/watch?v=996OiexHze0 ) by Nate Barbettini at Okta.
We also recommend reading
[OAuth 2 Simplified](https://aaronparecki.com/oauth-2-simplified) and having a
look at [the documentation](https://oauth.net/2/).

### Open ID Connect

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) is a
simple identity layer on top of the OAuth 2.0 protocol. It enables Clients to
verify the identity of the End-User based on the authentication performed by an
Authorization Server, as well as to obtain basic profile information about the
End-User in a REST-like manner.

Some good sources to get you started are:   
[Identity, Claims, & Tokens – An OpenID Connect Primer](https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1)
and  
[OpenID Connect explained](https://connect2id.com/learn/openid-connect).

### Supported OpenID Connect Flows

#### Authorization Code Grant

The authorization code grant type is used to obtain both access tokens and
refresh tokens and is optimized for confidential clients. Since this is a
redirection-based flow, the client must be capable of interacting with the
resource owner's user-agent (typically a web browser) and capable of receiving
incoming requests (via redirection)
from the authorization server.

For more information see
[RFC-6749 section 4.1](https://tools.ietf.org/html/rfc6749#section-4.1).

## Tokens

### ID Token

The ID token is a signed information object representing the authenticated identity of the user.   
As part of the OpenID Connect standard, the ID token is encoded as a JWT and signed using the JWS standard.
The ID Token can be decoded for debugging purposes by tools such as [jwt.io](https://jwt.io/).

Example header:

```json
{
  "alg": "RS256",
  "kid": "public:80f3c34a-9779-4e1e-b645-117f3b771af8",
  "typ": "JWT"
}
```

Example body:

```json
{
  "at_hash": "tyFnH20TOmPZkgJU8e5iKw",
  "aud": [
    "vipps-integration"
  ],
  "auth_time": 1557319296,
  "exp": 1557322938,
  "iat": 1557319338,
  "iss": "https://apitest.vipps.no/access-management-1.0/access/",
  "jti": "62a85e56-3d45-4c7e-a055-46932093257a",
  "nonce": "",
  "rat": 1557319255,
  "sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944"
}
```

You can read more at the [OIDC standard](https://openid.net/specs/openid-connect-core-1_0.html#IDToken)

It is important to validate the Id-token before using any data contained in it.
See the Oidc-standard on
[Id-token validation](https://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation)
for the specifics. We recommend that you use a library for this. A good place
to start is finding a library for your language at [jwt.io](https://jwt.io/#libraries-io).

### Access token

Access tokens are randoms strings that represents the authorization of a
specific application to access specific parts of a user’s data.
The token itself does not provide any information, but it can be used to
fetch the data that the end-user has consented to share from the
[userinfo endpoint](#openid-connect-userinfo).  
Access tokens _must_ be kept confidential in transit and storage.

Example:
```
"hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus"
```
For more information see [RFC-6749 section 4.1.3-4.1.4](https://tools.ietf.org/html/rfc6749#section-4.1.3).

### Refresh token

Vipps Login does not currently support refresh tokens.

## Scopes

Scopes are space-separated lists of identifiers used to specify what access privileges are being requested.  
Vipps Login currently supports the following scopes:

| Scopes      | Description                                    | User consent required  |
| ------------| -----------------------------------------------|-------- |
| openid      | Scope used to request an Id-token. It provides the claim “sub” which is a unique id for the end user at that particular merchant. Note: Different merchants will get different subs for the same end user.              |   no    |
| address     | List containing the users addresses. Will always contain home, but can also include work and other.    |   yes   |
| birthDate   | User birth date (BankID verified)                               |   yes   |
| email       | User email (verified), the flag "email_verified : true" in the response can be used by merchant to confirm for each request that the email actually is verified                                   |   yes   |
| name        | User first, middle and given name (verified with National Population Register)              |   yes   |
| phoneNumber | Verified phone number (verfied - the number used with Vipps)                          |   yes   |
| nnin        | Norwegian national identity number (verified with BankID). NB: merchants need to apply for access to NNIN. Go to [Who can get access to NNIN and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nnin-and-how) For more information |   yes      |
| accountNumbers | User bank account numbers. NB: merchants need to apply for access to accountNumbers. Go to [Who can get access to account numbers and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-accountnumbers-and-how) For more information |   yes      |

When requesting scopes that require user consent, a view listing these scopes
will be displayed to the user with the option to allow or deny the consent
request. This view is skipped if no scopes requiring consent are requested.
The user can not make changes to the list of requested scopes, and can
therefore not accept for example name and deny address.

We recommend asking for the minimal number of scopes needed for your use case to
minimize the number of users that deny the consent request.

## Recommendations

To ensure the best user experience, we recommend to perform the following checks
related to login/registration:

First check if you already have the unique user identifier for Vipps ("ID" from
now on, and called `sub` in the response from our API) stored on one of your
accounts. If you have it, this means that the user has used Vipps on your site
earlier and have an explicit link to the account. In this case use the ID to log
the user into her account.

If you have not already stored the ID: check if the user already have an account
based on phone number and e-mail address. If this gives a match on one (and
only one) account, then you can use this to log the user into that account since
both phone number and e-mail address is verified in Vipps.

Before completing the link it is an advantage to do a "sanity check" on the name
of the Vipps user to the name in the existing account to make sure that the
account is not an old account where the user has abandoned  the phone number or
e-mail address an this has been picked up by someone else at a later time.

If you get a match on multiple accounts you can provide information on this and
offer the user the possibility to log in to her existing account (using the old
login method) and then link the account to Vipps.

It is also recommended on "my page" or similar in the website to provide the
option for logged in users that has not yet linked their profile to Vipps to do
so, for an easier login the next time. This just means to provide the "login
with Vipps"-button and linking the ID from Vipps with this account.

## Integrating with Vipps Login

Vipps Login adheres to the OAuth2 and OpenIDConnect standards. The easiest way
to integrate with the service is therefore to use a well renowned
OAuth2.0/OpenID Connect Library for your programming language.
Vipps does not recommend a specific library, but the list of
[OIDC Relying Party libraries](https://openid.net/developers/certified/)
certified by the OpenID Foundation is a good starting point.

### Manual integration

This section contains information necessary to perform a manual integration with
Vipps Login. This should not be attempted without a solid grasp of the OAuth2
and Open ID Connect standards. All endpoints needed for integration can be found in our openid connect discovery endpoint. 
These endpoints should be fetched dynamically by your application, since they are prone for change.

#### Openid connect discovery endpoint

| Environment | Base URL |
|-------------|----------|
| Test        |https://apitest.vipps.no/access-management-1.0/access/.well-known/openid-configuration |
| Production  |https://api.vipps.no/access-management-1.0/access/.well-known/openid-configuration     |

The openid connect discovery endpoint can be used to retrieve configuration information for openid connect clients.  
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-discovery-1_0.html).

**Request**

[`GET:/.well-known/openid-configuration`](https://vippsas.github.io/vipps-login-api/#/public/discoverOpenIDConfiguration)

**Response**

Overview

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `500 Server Error`      | An internal Vipps problem.                              |

Example response from the merchant test environment:

```json
{
  "issuer": "https://apitest.vipps.no/access-management-1.0/access/",
  "authorization_endpoint": "https://apitest.vipps.no/access-management-1.0/access/oauth2/auth",
  "token_endpoint": "https://apitest.vipps.no/access-management-1.0/access/oauth2/token",
  "jwks_uri": "https://apitest.vipps.no/access-management-1.0/access/.well-known/jwks.json",
  "subject_types_supported": [
    "public",
    "pairwise"
  ],
  "response_types_supported": [
    "code",
    "code id_token",
    "id_token",
    "token id_token",
    "token",
    "token id_token code"
  ],
  "claims_supported": [
    "sub"
  ],
  "grant_types_supported": [
    "authorization_code",
    "implicit",
    "client_credentials",
    "refresh_token"
  ],
  "response_modes_supported": [
    "query",
    "fragment"
  ],
  "userinfo_endpoint": "https://apitest.vipps.no/vipps-userinfo-api/userinfo",
  "scopes_supported": [
    "offline",
    "openid",
    "address",
    "name",
    "email",
    "phoneNumber",
    "nnin",
    "birthDate",
    "accountNumbers"
  ],
  "token_endpoint_auth_methods_supported": [
    "client_secret_post",
    "client_secret_basic",
    "private_key_jwt",
    "none"
  ],
  "userinfo_signing_alg_values_supported": [
    "none",
    "RS256"
  ],
  "id_token_signing_alg_values_supported": [
    "RS256"
  ],
  "request_parameter_supported": true,
  "request_uri_parameter_supported": true,
  "require_request_uri_registration": true,
  "claims_parameter_supported": false,
  "revocation_endpoint": "https://apitest.vipps.no/access-management-1.0/access/oauth2/revoke",
  "backchannel_logout_supported": true,
  "backchannel_logout_session_supported": true,
  "frontchannel_logout_supported": true,
  "frontchannel_logout_session_supported": true,
  "end_session_endpoint": "https://apitest.vipps.no/access-management-1.0/access/oauth2/sessions/logout"
}
```

#### Openid connect discovery URLs

| Operation                 | Description         | Endpoints |
| ------------------------- | ------------------- | ------------------ |
| [OAuth 2.0 Authorize](#oauth-20-authorize)            | Start an OAuth 2.0 authorization. | [`GET:/oauth2/auth`](https://vippsas.github.io/vipps-login-api/#/public/oauthAuth) |
| [OAuth 2.0 Token](#oauth-20-token)                    | Get an OAuth 2.0 access token. | [`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/public/oauth2Token) |
| [Userinfo](#userinfo)                                 | Returns information that the user has consented to share. | [`GET:/userinfo`](https://vippsas.github.io/vipps-login-api/#/public/userinfo) |
| [JSON Web Keys Discovery](#json-web-keys-discovery)   | Get JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens. | [`GET:/.well-known/jwks.json`](https://vippsas.github.io/vipps-login-api/#/public/wellKnown) |

##### OAuth 2.0 Authorize

The authorize endpoint is a standard OIDC endpoint used for starting an
authorization. The client creates an request URI and directs the resource owner
to the constructed URI.

**Request**

The client constructs the request URI by adding the following parameters to the
query component of the authorization endpoint URI using the
`application/x-www-form-urlencoded` format. The client directs the resource
owner to the constructed URI using an HTTP redirection response, or by other
means available to it via the user-agent.

| Query             | Description                                                                                                                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| response_type     | Value MUST be set to "code".                                                                                                                                                              |
| client_id         | The client identifier, issued by Vipps.                                                                                                                                                   |
| redirect_uri      | Redirect URL which the user agent is redirected to after finishing a login. If the URL is using a custom URL scheme, such as `myapp://`, a path is required: `myapp://path-to-something`. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-from-the-merchant)          |
| scope             | Scope of the access request, space-separated list.                                                                                                                                        |
| state             | An opaque value used by the client to maintain state between the request and callback. The authorization server includes this value when redirecting the user-agent back to the client.   |

For example, the client directs the user-agent to make the following HTTP request:

[`GET:/oauth2/auth?client_id={client_id}&response_type=code&scope={scopes}&state={state}&redirect_uri={redirect_uri}`](https://vippsas.github.io/vipps-login-api/#/public/oauthAuth)

**Response**

If the resource owner grants the access request, the authorization server issues
an authorization code and delivers it to the client by adding the following
parameters to the query component of the redirection URI using the
`application/x-www-form-urlencoded` format.

| Query             | Description                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| code              | The authorization code generated by the authorization server. The client MUST NOT use the authorization code more than once. The authorization code is bound to the client identifier and redirection URI.    |
| state             | The exact value received from the client during the authorization request.                                                                                                                                    |
| scope             | The scopes that the end user has consented to. This list will always be the same as in the request query as end users are not allowed to remove individual scopes when they give consent.                                                                                                                                        |

For example, the authorization server redirects the user-agent by sending the
following HTTP response:

```
HTTP/1.1 302 Found
Location: https://client.example.com/callback?code={code}&state={state}&scope={scopes}
```

If the resource owner declines the access request or an error occurs, the
authorization server following parameters to the query component of the
redirection URI using the `application/x-www-form-urlencoded` format.

See error handling for more information.

For more information see
[RFC-6749 section 4.1.1-4.1.2](https://tools.ietf.org/html/rfc6749#section-4.1.1).

##### OAuth 2.0 Token

The token endpoint is a standard OIDC endpoint used for requesting Access and
ID Tokens. The client constructs the request by adding the parameters described
below to the HTTP body by using the `application/x-www-form-urlencoded` format.

**Request**

*Headers*

| Header            | Description                            |
| ----------------- | -------------------------------------  |
| Content-Type      | "application/x-www-form-urlencoded"    |                                                                                                                                                 
| Authorization     | "Basic {Client Credentials}"           |                                                                                                                                   

The Client Credentials is a base 64 encoded string consisting of the Client id
and secret issued by Vipps joined by ":"

Example in JavaScript:
```
var client_id = 123456-test-4a3d-a47c-412136fd0871
var client_secret = testdzlJbUZaM1lqODlnUUtrUHI=

var wordArrayAzp = CryptoJS.enc.Utf8.parse(client_id + ":" + client_secret);
var client_authorization = CryptoJS.enc.Base64.stringify(wordArrayAzp);
```

*Form content*

| Key               | Description                                                                                                                                                              |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| grant_type        | Value MUST be authorization_code.                                                                                                                                        |
| code              | The authorization code received from the authorization server.                                                                                                           |
| redirect_uri      | Redirect URL which the user agent is redirected to after finishing a login. If the URL is using a custom URL scheme, such as `myapp://`, a path is required: `myapp://path-to-something`. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-from-the-merchant) . This field is required for OIDC flows, i.e. regular Vipps Login logins. |                                                                                                                                    |

[`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/public/oauth2Token)

**Response**

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `401 Unauthorized`      | Invalid credentials.                                    |
| `500 Server Error`      | An internal Vipps problem.                              |


Example response:
```json
{
  "access_token": "hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus",
  "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6InB1YmxpYzo4MGYzYzM0YS05Nzc5LTRlMWUtYjY0NS0xMTdmM2I3NzFhZjgiLCJ0eXAiOiJKV1QifQ.eyJhdF9oYXNoIjoidHlGbkgyMFRPbVBaa2dKVThlNWlLdyIsImF1ZCI6WyJ2aXBwcy1pbnRlZ3JhdGlvbiJdLCJhdXRoX3RpbWUiOjE1NTczMTkyOTYsImV4cCI6MTU1NzMyMjkzOCwiaWF0IjoxNTU3MzE5MzM4LCJpc3MiOiJodHRwczovL2FwaXRlc3QudmlwcHMubm8vYWNjZXNzLW1hbmFnZW1lbnQtMS4wL2FjY2Vzcy8iLCJqdGkiOiI2MmE4NWU1Ni0zZDQ1LTRjN2UtYTA1NS00NjkzMjA5MzI1N2EiLCJub25jZSI6IiIsInJhdCI6MTU1NzMxOTI1NSwic3ViIjoiYzA2YzRhZmUtZDllMS00YzVkLTkzOWEtMTc3ZDc1MmEwOTQ0In0.OljG0W_TCfxkrRntj_5He3U0PH94SDZvlK-dvUJe8H5jj8QSiSnqiv65kyzxdr8Bq1MwG7a6Mtlnn4MoL8AyxKUVe6s81CNaYmwaHsWLw2Z2JmiPn5_X4lEy1nHVDX3R7lFKDQqFLSGnGNPU9bACj-Si18LBR-qv060wEj3b1ShrVeUIZCL1Yhxb6cIGl_8RivRto9dBrzggyOlVTtmoPrm9TLYF7UGWjlbmHTqpBWsCQIOeQqgs7RmSBt5k3O9nmP7guVxo5MWv_2Z0XuCqobLDDXJ29Rk_W6d79y-lPzq_TedNb_lCdVJF7u9qDYFbIPuQwXp26CeIJcR-nc-t0qEoNmLru_x-9Z8dCjjzkZbWqyNsNedQU1zt0WFbHjRkodVoHNcRZVT5W5hCe54lmZ6lUqyKwHW0_3Rpd2CI6lPdCOhC-Tze5cUDfb8jT_0OZqCI_wAuWvb6_4VeHqhvUav6Mh6d7AxNJQYG6BAJo9TzyrG7ho4mSpb2wWMr8gmRi8pTQbqa40whPqptpiz_j4AHcsrRckjYONU0USKlnNcBGc24M4sprcLZ6vxFqDYmDoZwUDRdZWRpUbqm_nCmCKb20Z6l5O7h32KvOApopJe2NIeAynli3Nl05QVGOdoT1mZDLYXbtyb0b_4qhRflySr6gaczcf2ovUKAToKNs_4",
  "expires_in": 3599,
  "scope": "openid",
  "token_type": "bearer"
}
```

#####  Userinfo

This endpoint returns the payload with the information that the user has consented to share, which is provided in 
the OAuth 2.0 access token.
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo).

**Request**

*Headers*

| Header            | Description                            |
| ----------------- | -------------------------------------  |
| Authorization     | "Bearer {Access Token}"                |    

The access token is received on a successful request to the [token endpoint](#oauth-20-token)

[`GET:/userinfo`](https://vippsas.github.io/vipps-login-api/#/public/userinfo)

**Response**

Overview

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `401 Unauthorized`      | Invalid credentials.                                    |
| `500 Server Error`      | An internal Vipps problem.                              |

Example response:
```json
{
    "sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
    "birthdate": "10.12.1815",
    "email": "user@example.com",
    "email_verified": true,
    "nnin": "10121550047",
    "name": "Ada Lovelace",
    "given_name": "Ada",
    "family_name": "Lovelace",
    "sid": "7d78a726-af92-499e-b857-de263ef9a969",
    "phone_number": "4712345678",
    "address": [
        {
            "street_address": "Suburbia 23",
            "postal_code": "2101",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Suburbia 23\\n2101 OSLO\\nNO",
            "address_type": "home",
            "default": true
        },
        {
            "street_address": "Fancy Office Street 2",
            "postal_code": "0218",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Fancy Office Street 2\\n0218 OSLO\\nNO",
            "address_type": "work",
            "default": false
        },
        {
            "street_address": "Summer House Lane 14",
            "postal_code": "1452",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Summer House Lane 14\\n1452 OSLO\\nNO",
            "address_type": "other",
            "default": false
        }
    ],
    "accounts": [
        {
            "account_name": "My savings",
            "account_number": "12064590675",
            "bank_name": "My bank"
        }
    ]
}
```

#####  JSON Web Keys Discovery

This endpoint returns JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens and if enabled,
OAuth 2.0 JWT Access Tokens.

**Request**

[`GET:/.well-known/jwks.json`](https://vippsas.github.io/vipps-login-api/#/public/wellKnown)

**Response**

Overview

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `500 Server Error`      | An internal Vipps problem.                              |

Examples:  
*200 response*

```json
{
    "keys": [
        {
            "use": "sig",
            "kty": "RSA",
            "kid": "public:80f3c34a-9779-4e1e-b645-117f3b771af8",
            "alg": "RS256",
            "n": "n4kytA_ZeI3Znl96b-zteWmrSHjRmXnvLmACQ4W_BVRTtuhumuoLpOpavUcEOhajbLKqBrANC0dd7ABtL92gHRoVRp8VMSNBG6ykuD04gevxqgm2Gz1mGineWBrjINwY-WboPMqgyZLLKw-JjZ6EqHm67TnWxrKjk3135tGILWvrTJ6ykglGpfH0jpGtEOS6VUuSWeW5VitEGeOFDwWja4mYXZbfGICtKnD6LZOM8sFKyC9diBmDXuXZvxdwdBfVh9JvWMaZ5bJ_mv4iJ0qU0FM1yginRMLXY3MOxwLSLSQjmDPV8NZecUsK-UEfpJl7lvLdEFqKCdqQxkEEyLPMq_rM50F5QIbKa0BYoa973-OR9cJ-XQkLNCtF5i003Z3V-5c7N3xBNlpGYsp4CCvv--zFpxvRZ0k-axgu1Loj1eOZRuX1DL86iTQ6YXcBGPYlRkRNJOn7mkRV8wn8kp_DMlzJ7bgE-NFEA32wFrEimPCrGUfdNdjn6gMsWKcbiHiM4NlIDrCNPiD2CVPrHGuVex8R0cpKU0Cvxc8mXqvDc-VJbrqrRNPbZJh0Zoz4wfK9LESdYGAah9PhgHg5LDFHmhT0W3FPmy1Gmfk2ino4PryJ_rDXe0WT84IEGV9YEM631gDerS63D1dKvTP01YT5sd2Ymn9eptH9grvFeOhuvz0",
            "e": "AQAB"
        }
    ]
}
```
This operation does not require authentication

### Automatic Recognition

Vipps Login will offer the option to authenticate end-users automatically when
they return to a merchant's site. This gives merchants the opportunity to
provide custom content to the user directly without the need for manual
authentication. A precondition is that the user has consented to use Vipps
login at the merchant's site, and has opted into being remembered in the browser
being used. This feature is still under development.

## API endpoints required from the merchant

The following endpoints are to be implemented by merchants, in order for Vipps
to redirect the resource owner to them.

### Receive authentication result

After a successful authentication, the user agent is redirected to this endpoint
with the following parameters added to the query component. This URI needs to
be pre-registered with Vipps and supplied as a query parameter on calls to the
[OAuth2 authorize endpoint](#OAuth-2.0-Authorize).  

| Param             | Description                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| code              | The authorization code generated by the authorization server. The client MUST NOT use the authorization code more than once. The authorization code is bound to the client identifier and redirection URI.    |
| state             | The exact value received from the client during the authorization request.                                                                                                                                    |

Example:
```
HTTP/1.1 302 Found
Location: https://client.example.com/callback?code={code}&state={state}
```

## Error handling

If the user cancels the login or an error occurs, the user agent is redirected
to the receive authentication result endpoint with the following parameters
added to the query component.

| Param             | Description                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| error             | Standard [OAuth2](https://tools.ietf.org/html/rfc6749#section-4.1.2.1) or [OIDC](https://openid.net/specs/openid-connect-core-1_0.html#AuthError) error code. |
| error_description | A short text providing additional information on the error that occurred.                                                                                     |
| state             | The exact value received from the client during the authorization request.                                                                                    |

Example:
```
HTTP/1.1 302 Found
Location: https://client.example.com/callback?error=access_denied&error_description=user%20cancelled%20the%20login?state={state}
```

If a fatal error occurs where the user can not be redirected back to the merchant, a generic Vipps styled error page will be shown containing a brief error description.

## Questions and answers

### Compliance  

**Q**: How can our system dynamically "know/find out" if the user has revoked their concent for us to have access to his personal data in our system?  
**A**: We are working on a system for notifying merchants when an end user revokes their consent. It will consist of an additional callback url that you register on the merchant portal which we will send a notification to when an end user removes their consent  

### Technical

**Q**: Can we have multiple URIs as landing pages?  
**A**: You can register as many callback urls as you want; and then you specify which one you use in the request to [/auth](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/oauthAuth)  

### UX

**Q**: Can we change the name that appears in customer's Vipps app under `Login and Access`?  
**A**: The name which is displayed in the app is the name of the Sale Unit. You can do it yourself in the merchant portal, [https://portal.vipps.no/](https://portal.vipps.no/). Press `rediger`/`edit` under `salgsstedsinfo`/`?` and change to the desired name.  

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
