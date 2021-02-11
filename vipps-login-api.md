# Vipps Login API

API version: 2.0

Please note that **all merchants on Vipps Login need to include scope `api_version_2`.**
Version 1 (the version you will get if `api_version_2` is omitted) will be discontinued February 28th 2021.

Document version 4.0.2.

See the
[Vipps Login API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md)
for all the details.

## Table of contents
* [Introduction](#introduction)
  - [Activation](#activation)
  - [Versions](#versions)
  - [Migration](#migration)
  - [Flows](#flows)
    * [Remembered flow](#remembered-flow)
    * [Desktop flow - phone number based push flow](#desktop-flow---phone-number-based-push-flow)
    * [Mobile flow - deeplink based flow](#Mobile-flow---app-switch-based-flow)
        * [App to app flow](#app-to-app-flow)
        * [Automatic return from Vipps app](#automatic-return-from-vipps-app-requires-the-merchant-to-handle-user-session-cross-browsers)
    * [No dialog flow - log the user in directly when possible](#no-dialog-flow---log-the-user-in-directly-when-possible)
  - [Design guidelines and buttons](#Design-guidelines-and-buttons)
* [Core concepts](#core-concepts)
  - [OAuth 2.0](#oauth-20)
  - [OpenID Connect](#open-id-connect)
  - [Supported OpenID Connect Flows](#supported-openid-connect-flows)
    * [Authorization Code Grant](#authorization-code-grant)
  - [Tokens](#tokens)
    * [ID Token](#id-token)
    * [Access Token](#access-token)
    * [Refresh Token](#refresh-token)
    * [Token endpoint authentication method](#token-endpoint-authentication-method)
  - [Scopes](#scopes)
* [Recommendations on linking to user account](#Recommendations-on-linking-to-user-account)
* [Integrating with Vipps Login](#integrating-with-vipps-login)
    * [Manual integration](#manual-integration)
        * [Openid connect discovery endpoint](#openid-connect-discovery-endpoint)
        * [Openid connect discovery URLs](#openid-connect-discovery-urls)
            * [OAuth 2.0 Authorize](#oauth-20-authorize)
            * [OAuth 2.0 Token](#oauth-20-token)
            * [Userinfo](#userinfo)
            * [JSON Web Keys Discovery](#json-web-keys-discovery)
  * [API endpoints required from the merchant](#api-endpoints-required-from-the-merchant)
    * [Receive authentication result](#receive-authentication-result)
* [Using the special flows](#using-the-special-flows)
    * [App integration](#app-integration)
    * [Automatic return from Vipps app](#automatic-return-from-vipps-app)
    * [No dialog flow](#No-dialog-flow)
* [Error handling](#error-handling)
* [Questions and answers](#questions-and-answers)
* [Questions](#questions)

## Introduction

The Vipps Login API offers functionality for authenticating end users and authorizing clients founded on the OAuth2 and
OpenID Connect specifications. Login with Vipps is the easiest way to sign in and create an account. Users don’t need to worry about forgetting usernames and passwords. All they need to remember is their phone number. For an even smoother sign in experience, the user can choose to be remembered in the browser, enabling automatic sign-ins for later visits.

Users can create a new account through sharing high-quality data from the user’s Vipps profile. Available information includes name, email, addresses, phone number, and birth date. Norwegian national identity number is also available to [some merchants](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nin-and-how).  The identity of all Vipps users is verified using BankID, Norway’s leading electronic ID, so rest assured that these are real people with correct name and information.

There are **Vipps login plugins** available for several platforms. You find the complete list on our [plugins page](https://github.com/vippsas/vipps-plugins).
If a plugin is not available, the easiest - and **strongly recommended - way to integrate with the service is to use a well renowned OAuth2.0/OpenID Connect Library for your programming language**. Vipps does not recommend a specific library, but the list of [OIDC Relying Party libraries](https://openid.net/developers/certified/) certified by the OpenID Foundation is a good starting point.

The Vipps login API authenticates the user in the web browser. The Vipps login API should only be run in the browser window using redirects (iFrame is not supported and new window is not recommended).

See our [developer section](https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md#vipps-test-apps) for information about our test environment, test apps and test users.

### Versions
[Api Version 1.0](https://github.com/vippsas/vipps-login-api/blob/master/versions/1.0/vipps-login-api.md)

### Migration
[Api 1.0 to 2.0](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-migrate-api-1.0-to-2.0.md)

### Activation

See the FAQ:
[How can I activate and set up Vipps login?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login)

### Flows

#### Remembered flow
If a user has chosen to be remembered in browser then the authentication can be completed in the browser. The user will then either be asked to provide consent to share profile information or be logged in directly.  This applies to both desktop and mobile.

If the user is not remembered the user needs to confirm the login in the Vipps-app. The flow associated with this will differ depending on whether the user is on desktop or mobile:

#### Desktop flow - phone number based push flow
If the user is on desktop, and not remembered in browser, then the user will follow this flow.  If the user is remembered in browser then only the consent flow at the bottom will be completed. If the user already has provided consent then this step will be skipped also, allowing a direct login experience.

The user initiates the login by inputing the phone number and selecting whether to  be remembered in browser:
![Number input in desktop](images/Number_input_flow_desktop1.png)

The user goes to the Vipps app and confirms the login:
![Confirmation in app in phone number flow](images/Number_input_flow_app.png)

The user is then authenticated in browser and can provide consent if required. Then the user is redirected back to the redirect URI provided by merchant:
![Consent in desktop](images/Number_input_flow_desktop2.png	)


#### Mobile flow - deeplink based flow
If the user is on a mobile device, the Vipps landing page in the browser will automatically trigger a deeplink to the Vipps app if the user is not remembered in the browser. The user will not be prompted to enter the phone number. In the Vipps app the user confirms the login and can choose whether to be remembered in the browser for later logins. After confirming in the app the user needs to switch back to the Vipps page in the browser/app. On the Vipps landing page the user will finalise the autentication and provide consents if required. The user is then redirected back to the redirect URI provided by merchant (could be webpage or an app).

![Mobile flow with app-switch](images/Mobile_flow_with_partial-app_switch.png)

We recommend that apps initiate Vipps login in a webview. SafariViewController and Chrome Custom Tabs are preferred implementations as these are able to access cookies stored in the user's browser and they are known to support deeplinking to the Vipps app.

There are two specialised flows that merchants can use to automatically switch the user back from the Vipps-app to the originating browser/app upon login confirmation. From the illustration above this means that the page "Gå tilbake til butikken" will be skipped and that the "manual app switch"  will be replaced by an automatic app-switch (eg. deeplink). These flows give a better user experience than the standard flow, but they also require the merchant to handle some more complexity in the integration.

Which of the flows to use is controlled with the initiation of the individual login session. The merchant can thus use all available login flows on the same client_id and adapt to the different use-cases and login scenarios.

The two flows are:

##### App to app flow
This flow is designed to be used with apps. It requires that the app initiate Vipps login in a webview (SafariViewController and Chrome Custom Tabs are preferred). In this flow the merchant need to specify the app URI where the user will be returned after completing the confirmation in the Vipps app.

See [how to implement](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#app-integration).

##### Automatic return from Vipps app (requires the merchant to handle user session cross browsers)
This flow is designed for web-pages that would like to have the user automatically returned to a browser after completing the confirmation in the Vipps app. Note that there are security implications by using this flow. **It is not suited for every scenario. Merchants must make their own considerations to ensure that it is only used where suitable**.

Due to how the different mobile operating systems handle app-switch to browser, the user can be returned to a different browser than the one he/she started in. On iOS the user can e.g. start the login in Chrome and be returned to Safari after confirming in the Vipps app. This means that the merchant site cannot rely on cookies beeing present in the browser the user is returned to.

By using this flow Vipps login will be able to complete the login process even if the user ends up in a different browser. However, the merchant **must ensure that logins can complete, even without session information like cookies.**

See [how to implement](#automatic-return-from-vipps-app), including more informattion on the security considerations.

#### No dialog flow - log the user in directly when possible
This flow can be used to log the user in directly if the required prerequisites are in place. If the prerequisites are not in place, then the Vipps login process will be stopped and no interaction will be asked from the user in this flow. When using this flow a spinner will be shown while Vipps login try to log the user in. Once the process is completed the user will be returned to the merchant as in the ordinary Vipps login flow. As with the other Vipps login flow it is recommended to run Vipps login in a redirect mode and iFrame is not supported.

The user will be logged in with this flow if:
* they are remembered in the browser and no consent is required
* they are remembered in the browser and consent has previously been given


Illustration of how the flow can look when the user clicks "Logg inn" on the front page:

![No dialog flow](images/No_dialog_flow.png)



Possible use cases includes:
* When the user is going to a section of your site/service that requires the user to be logged in, e.g. my page or a personalized chatbot. This might be when the user is already on the webpage, or if the user is being linked directly to her My page from an email/newsletter.
* When a user clicks on your login option it is possible to try to log the user in with Vipps first.

In such scenarios the users that can be logged in directly will get an even better login experience and quickly come to the information and services that are relevant for them

If the user cannot be logged in directly you can e.g show them your ordinary login screen. On the login screen Vipps will be an option and users that is not remembered in browser or has not yet given consent can actively login with Vipps from here.

See [how to implement](#no-dialog-flow).

### Design guidelines and buttons
Buttons to use for Vipps login can be found as part of our
[design guidelines](https://github.com/vippsas/vipps-design-guidelines/tree/master/vipps-buttons).

## Core concepts
Vipps Login adheres to the OAuth2 and OpenID Connect standards. The core concepts releated to these are presented below. The next chapter goes into the actual implementation.

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
[userinfo endpoint](#userinfo).
Access tokens _must_ be kept confidential in transit and storage.

Example:
```
"hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus"
```
For more information see [RFC-6749 section 4.1.3-4.1.4](https://tools.ietf.org/html/rfc6749#section-4.1.3).

### Refresh token

Vipps Login does not currently support refresh tokens.


### Token endpoint authentication method
The token endpoint is a standard OIDC endpoint used for requesting Access and ID Tokens.
The default token endpoint authentication method is `client_secret_basic`.
It is possible to change the authentication method to `client_secret_post` in the Vipps portal. This setting will then apply to all login transactions on this sales unit.
Go to [How can I use client_secret_post for authentication?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication) for more information on how to change the authentication method.

For more information on the token endpoint see [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#TokenEndpoint) and [RFC-6749 section 3.2](https://tools.ietf.org/html/rfc6749#section-3.2)

## Scopes

Scopes are space-separated lists of identifiers used to specify what access privileges are being requested.
Vipps Login currently supports the following scopes:

| Scopes      | Description                                    | User consent required  |
| ------------| -----------------------------------------------|-------- |
| openid      | Scope used to request an Id-token. It provides the claim “sub” which is a unique id for the end user at that particular merchant. Note: Different merchants will get different subs for the same end user.              |   no    |
| address     | User can have up to three addresses in Vipps: home, work and other. Users' addresses are given as claims 'address' and 'other_addresses'. The claim 'address' returns the address set as 'default' for the Vipps user. And the claim 'other_addresses' returns all other addresses of the end user, if any.  We recommend that merchants fetch all addresses on a user and allow the user to choose which address to use in the relevant context. Some users will not have any registered address, in these situations the claim 'address' will be delivered, but the sub claims in address will be empty strings, e.i. "address" : {"country" : "", "street_address" : "", "address_type" : "", "formatted" : "", "postal_code" : "", "region" : "" } |   yes   |
| birthDate   | User birth date (BankID verified)                               |   yes   |
| email       | User email (verified), the flag "email_verified : true" in the response can be used by merchant to confirm for each request that the email actually is verified                                   |   yes   |
| name        | User first, middle and given name (verified with National Population Register)              |   yes   |
| phoneNumber | Verified phone number (verfied - the number used with Vipps)                          |   yes   |
| nin        | Norwegian national identity number (verified with BankID). NB: merchants need to apply for access to NIN. Go to [Who can get access to NIN and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) For more information |   yes      |
| accountNumbers | User bank account numbers. NB: merchants need to apply for access to accountNumbers. Go to [Who can get access to account numbers and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-accountnumbers-and-how) For more information |   yes      |
| api_version_2 | Needs to be added to use the latest version of Vipps Login. All merchants need to add this. Go to [Vipps Login API 2.0 migration](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-migrate-api-1.0-to-2.0.md) For more information about migrating from version 1 |   no      |

When requesting scopes that require user consent, a view listing these scopes
will be displayed to the user with the option to allow or deny the consent
request. This view is skipped if no scopes requiring consent are requested.
The user can not make changes to the list of requested scopes, and can
therefore not accept for example name and deny address.

We recommend asking for the minimal number of scopes needed for your use case to
minimize the number of users that deny the consent request.

**All merchants on Vipps Login need to include scope 'api_version_2'.**
The scope decides the information delivered as part of the ID token as well as the API response from userinfo.
Version 1 (the version you will get if 'api_version_2'is omitted) will be discontinued.

## Recommendations on linking to user account

To ensure the best user experience, we recommend performimng the following checks
related to login/registration:

First check if you already have the unique user identifier for Vipps ("ID" from
now on, and called `sub` in the response from our API) stored on one of your
accounts. If you have it, this means that the user has used Vipps on your site
earlier and have an explicit link to the account. In this case use the ID to log
the user into her account.

If you have not already stored the ID: check if the user already has an account
based on phone number and e-mail address. If this gives a match on one (and
only one) account, then you can use this to log the user into that account since
both phone number and e-mail address are verified in Vipps. Before linking an account based on e-mail, ensure that the flag "email_verified : true" in the response. If this for some reason is "false" the matching should be aborted, or the user should be prompted to login to the original account or confirm the account linking by having a confirmation link sent to the email address.

Before completing the linking, it is an advantage to do a "sanity check" on the name
of the Vipps user to the name in the existing account to make sure that the
account is not an old account where the user has abandoned the phone number or
e-mail address and this has been picked up by someone else at a later time.

If you get a match on multiple accounts, you can provide information on this and
offer the user the possibility to log in to her existing account (using the old
login method) and then link the account to Vipps.

It is also recommended on "my page" or similar in the website to provide the
option for logged in users that has not yet linked their profile to Vipps to do
so, for an easier login the next time. This just means to provide the "login
with Vipps"-button and linking the ID from Vipps with this account.

## Integrating with Vipps Login

Vipps Login adheres to the OAuth2 and OpenID Connect standards. The easiest - and **strongly recommended - way
to integrate with the service is therefore to use a well renowned
OAuth2.0/OpenID Connect Library for your programming language**.
Vipps does not recommend a specific library, but the list of
[OIDC Relying Party libraries](https://openid.net/developers/certified/)
certified by the OpenID Foundation is a good starting point.

### Manual integration

This section contains information necessary to perform a manual integration with
Vipps Login. This **should not be attempted without a solid grasp of the OAuth2
and Open ID Connect standards**. All endpoints needed for integration can be found in our openid connect discovery endpoint.
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
    "nin",
    "birthDate",
    "accountNumbers",
    "api_version_2"
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
| requested_flow            | Optional. Request a specific flow for the user. See [App integration](#app-integration), [Automatic return from Vipps app](#automatic-return-from-vipps-app) and [No dialog flow](#no-dialog-flow)                                            |
| app_callback_uri  | Optional. The target uri for automatic switch back to merchant app. Requires `requested_flow=app_to_app`. Example `merchant-app://callback`                                                             |

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

If the resource owner declines the access request, or an error occurs, the authorization server adds the following parameters to the query component of the
redirection URI using the `application/x-www-form-urlencoded` format. More details in [Error handling](#error-handling)

For more general information see the standard specifications
[OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#AuthResponse) and [RFC-6749 section 4.1.1-4.1.2](https://tools.ietf.org/html/rfc6749#section-4.1.1).

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
    "birthdate": "1815-12-10",
    "email": "user@example.com",
    "email_verified": true,
    "nin": "10121550047",
    "name": "Ada Lovelace",
    "given_name": "Ada",
    "family_name": "Lovelace",
    "sid": "7d78a726-af92-499e-b857-de263ef9a969",
    "phone_number": "4712345678",
    "address": {
        "street_address": "Suburbia 23",
        "postal_code": "2101",
        "region": "OSLO",
        "country": "NO",
        "formatted": "Suburbia 23\\n2101 OSLO\\nNO",
        "address_type": "home"
    },
    "other_addresses": [
        {
            "street_address": "Fancy Office Street 2",
            "postal_code": "0218",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Fancy Office Street 2\\n0218 OSLO\\nNO",
            "address_type": "work"
        },
        {
            "street_address": "Summer House Lane 14",
            "postal_code": "1452",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Summer House Lane 14\\n1452 OSLO\\nNO",
            "address_type": "other"
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

## Using the special flows

### App integration
_This feature is new and might need modifications to support all merchant app needs._

It is possible to enable automatic switch of users back to the merchant app, from the Vipps app. This flow is described [here](#app-to-app-flow).

Expected flow:
```
Merchant app -> Merchant app controlled browser -> Vipps app -> Merchant app controlled browser -> Merchant app
```

This flow can be enabled per login request by adding the `requested_flow=app_to_app` and `app_callback_uri` parameters to the [Authorize](#oauth-20-authorize) request.

This flow requires both the `app_callback_uri` and `redirect_uri` parameters.

The `app_callback_uri` should be a URI that makes the device switch back to the merchant's app again after the Vipps app portion of the flow is done (example: "merchant-app://callback"). 

The `redirect_uri` is opened in the webview once the vipps login flow is completed there. This url can either redirect the user to a page in the webview, or be handled/intercepted by the merchant app. In either case it is important to avoid using static client secrets in the app for completing the login. (For more information see https://github.com/openid/AppAuth-Android#utilizing-client-secrets-dangerous and https://tools.ietf.org/html/rfc8252#section-8.5).

<b>Both</b> URIs must be added in the [merchant portal](https://portal.vipps.no/), you find more information on how to do this [here](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login).

The Vipps app will return some data with the return to the `app_callback_uri`. It contains two query parameters `state` and `resume_uri`. `State` is the OIDC `state` parameter passed at the start of a login which can be used to identify the specific login if needed. The `resume_uri`parameter is an URI that is optional and can be used to resume the login after returning from the Vipps app.

Example `app_callback_uri` request
``` 
no.vipps.apptestclient://callback/?state=RFiQdrl_lvJUpVmTRSKmsZRGLM0G1N1qh0WebZ1gDNk&resume_uri=https%3A%2F%2Fapi.vipps.no%2Fvipps-login-idp%2Findex.html%3FtabId%3D7607f7f0-7ae2-49b7-9cb5-102143dac4ea
```

A typical flow/implementation might look like this:
1. An OpenID authentication flow authorization URI is generated on the merchant backend. The URI is communicated to the merchant app and is used to initiate Vipps login in a browser. SafariViewController and Chrome Custom Tabs are preferred browsers.
2. If required, Vipps login will deeplink the user over to the Vipps app (if the user is remembered in browser and you use a webview with access to these cookies the user will be authenticated directly in your webview - the user will then be on step 5 below)
3. When the user has approved the login in the Vipps app the `app_callback_uri` is triggered.
4. The `app_callback_uri` will return the user to the merchant app. The app now has 2 separate choices.
   * Display the same browser instance that was created in 1.
   * Open a browser again using the `resume_uri`query parameter that is returned with the request to the `app_callback_uri`.
5. Vipps login will finalise the authentication of the user and acquire consent to share information if needed. When this is finished the user will be redirected to the `redirect_uri`. The Vipps login process has now finished and the merchant controls the remaining process.
6. The merchant app sends the `code` and `state`parameters received in the callback to the merchant backend
7. The merchant backend fetches the access tokens and user information

Examples:
```
...&requested_flow=app_to_app&app_callback_uri=merchant-app://trigger&redirect_uri=merchant-app://callback...
...&requested_flow=app_to_app&app_callback_uri=merchant-app://trigger&redirect_uri=https://merchant.com/callback...
```

Parameters `state` and possibly `error` will be passed as query parameters to the `app_callback_uri`. The `state` parameter has the same value as the `state` parameter passed to the [Authorize](#oauth-20-authorize) request.

Example success callback:
```
merchant-app://callback?state=218gz18yveu1ybajwh2g1h3g
```

Example error callback:
```
merchant-app://callback?state=218gz18yveu1ybajwh2g1h3g?error=unknown_error
```


### Automatic return from Vipps app

When enabled this flow will automatically take the user back to a browser when they accept the login in the Vipps app. It is not suitable for every scenario please see [the detailed description](#automatic-return-from-vipps-app-requires-the-merchant-to-handle-user-session-cross-browsers) and be aware of the security implications mentioned there.

This flow can be enabled per login by adding the parameter `requested_flow=automatic_return_from_vipps_app` to the [Authorize](#oauth-20-authorize) request.

#### Implementation suggestions

**It is not possible to give a single description that ensures secure use of this flow for all scenarios. The suggestions given here may not apply to every scenario and must be considered in relation to the specifics of the implementation.**

##### Session information

The [state parameter](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#whats-the-purpose-of-the-state-parameter)  passed in the [OAuth2 authorize endpoint](#OAuth-2.0-Authorize) request can carry some information from the start of a login until the callback.
The state parameter cannot be thought of as a direct replacement of a user agent bound session.

Some relevant considerations:
* Always use [PKCE](https://oauth.net/2/pkce/)
* Avoid logging the callback URI
* [Session fixation](https://owasp.org/www-community/attacks/Session_fixation). Be aware of what is possible to set up before a login is started.
* [Login CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#login-csrf). Be aware if it's possible to input sensitive information after a login.
* [The state parameter and CSRF](https://tools.ietf.org/html/rfc6749#section-10.12). Be aware of the recommendations of the OIDC/OAuth standards.

##### Verification
It is important merchants verify that users returning to a different browser than where the login started is handled as expected. It is also recommended testing starting the login in private/incognito mode as this will have similar effects as being returned to a different browser.


### No dialog flow
This flow is described [here](#no-dialog-flow---log-the-user-in-directly-when-possible). To attempt a no dialog login add the query parameter `requested_flow=no_dialog` to the [Authorization request](#OAuth-2.0-Authorize) uri.


If the user completes the login they will be returned to the `redirect_uri` with a code that can be used to complete the login. Just like in a regular [Authorization code](#authorization-code-grant) login.

`https://client.example.com/callback?code=12adsa938721987321asd9873&state=123987218dh`

If the user is not logged in they will be returned with an error. Some possible errors are `interaction_required`, `login_required` or `server_error`.

Not logged in return uri example: `https://client.example.com/callback?error=interaction_required&error_description=User+interaction+is+required&state=1312312321983212a3b`.

In all cases a new login can be started by removing the parameter `requested_flow=no_dialog` and initiating a new login for the user.

## Questions and answers

### Compliance

**Q**: How can our system dynamically "know/find out" if the user has revoked their concent for us to have access to his personal data in our system?
**A**: We have a system for notifying merchants when an end-users revoke their consents. You find information in this webhook [here](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-webhooks.md)

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
