<!-- START_METADATA
---
title: API Guide
sidebar_position: 30
---
END_METADATA -->

# Vipps Login API

API version: 2.0

Document version 4.0.10.
<!-- START_TOC -->

## Table of contents

* [Introduction](#introduction)
  * [Activation](#activation)
* [Flows](#flows)
  * [Vipps Login in browser](#vipps-login-in-browser)
    * [Remembered flow](#remembered-flow)
    * [Desktop flow - phone number based push flow](#desktop-flow---phone-number-based-push-flow)
    * [Mobile flow - deeplink based flow](#mobile-flow---deeplink-based-flow)
      * [App to app flow](#app-to-app-flow)
      * [Automatic return from Vipps app (requires the merchant to handle user session cross browsers)](#automatic-return-from-vipps-app-requires-the-merchant-to-handle-user-session-cross-browsers)
    * [No dialog flow - log the user in directly when possible](#no-dialog-flow---log-the-user-in-directly-when-possible)
  * [Vipps Login from phone number](#vipps-login-from-phone-number)
  * [Vipps Login from QR-code](#vipps-login-from-qr-code)
  * [Vipps Login + Vipps Recurring](#vipps-login--vipps-recurring)
* [Design guidelines and buttons](#design-guidelines-and-buttons)
* [Core concepts](#core-concepts)
  * [OAuth 2.0](#oauth-20)
  * [Open ID Connect](#open-id-connect)
  * [Supported OpenID Connect Flows](#supported-openid-connect-flows)
    * [Authorization Code Grant](#authorization-code-grant)
* [Tokens](#tokens)
  * [ID Token](#id-token)
  * [Access token](#access-token)
  * [Refresh token](#refresh-token)
  * [Token endpoint authentication method](#token-endpoint-authentication-method)
* [Scopes](#scopes)
* [Recommendations on linking to user account](#recommendations-on-linking-to-user-account)
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
  * [Error handling](#error-handling)
    * [Custom error codes](#custom-error-codes)
  * [Call by call](#call-by-call)
  * [Using Vipps Login in native applications](#using-vipps-login-in-native-applications)
* [Using the special flows](#using-the-special-flows)
  * [App integration](#app-integration)
    * [A typical flow/implementation might look like this:](#a-typical-flowimplementation-might-look-like-this)
  * [Automatic return from Vipps app](#automatic-return-from-vipps-app)
    * [Implementation suggestions](#implementation-suggestions)
      * [Session information](#session-information)
      * [Verification](#verification)
  * [No dialog flow](#no-dialog-flow)
* [Integrating with Vipps Login from phone number](#integrating-with-vipps-login-from-phone-number)
  * [Activating Vipps Login from phone number](#activating-vipps-login-from-phone-number)
  * [Complete login in the Vipps app](#complete-login-in-the-vipps-app)
    * [Activation](#activation)
    * [Overview](#overview)
    * [Call by call](#call-by-call)
    * [Authentication Request (https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request)](#authentication-request-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-10htmlauthrequest)
      * [Authentication](#authentication)
      * [The `login_hint` parameter (required)](#the-loginhint-parameter-required)
      * [The `scope` parameter (required)](#the-scope-parameter-required)
      * [The `binding_message` parameter (optional)](#the-bindingmessage-parameter-optional)
      * [Format](#format)
      * [Error responses](#error-responses)
      * [Successful responses (https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#successful_authentication_request_acknowdlegment)](#successful-responses-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-10htmlsuccessfulauthenticationrequestacknowdlegment)
    * [Token request (https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.10.1)](#token-request-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-10htmlrfcsection101)
      * [Polling](#polling)
    * [Error responses](#error-responses)
  * [Redirect to browser](#redirect-to-browser)
    * [Activation](#activation)
    * [Overview](#overview)
    * [Call by call](#call-by-call)
    * [Authentication Request With Redirect (https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request)](#authentication-request-with-redirect-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-10htmlauthrequest)
      * [Authentication](#authentication-1)
      * [The `login_hint` parameter (required)](#the-loginhint-parameter-required-1)
      * [The `scope` parameter (required)](#the-scope-parameter-required)
      * [The `binding_message` parameter (optional)](#the-bindingmessage-parameter-optional)
      * [The `redirect_uri` parameter (required)](#the-redirecturi-parameter-required)
      * [Error responses](#error-responses)
* [Integrating with Vipps Login from QR code](#integrating-with-vipps-login-from-qr-code)
  * [Activating Vipps Login from QR code](#activating-vipps-login-from-qr-code)
  * [Initiate login from QR code](#initiate-login-from-qr-code)
    * [Call by call](#call-by-call)
  * [Initiate login from QR code with redirect to browser](#initiate-login-from-qr-code-with-redirect-to-browser)
    * [Overview](#overview)
    * [Call by call](#call-by-call)
* [Consent](#consent-webhooks)
  * [Revoke](#revoke)
* [Questions?](#questions)

<!-- END_TOC -->

## Introduction

Vipps Login is the easiest way to sign in and create an account in both digital and physical settings. No typing required! Users don’t need to worry about forgetting or loosing their usernames and passwords. All they need to remember is their phone number. Vipps Login is easier for the user while you as a merchant get better conversion and data quality.

Vipps Login is available for several use cases:

* On web pages and in apps - Vipps Login is used in the browser
* Directly from merchant systems (e.g. point of sales systems or call-center solutions) based on phone number
* Directly from machines and vending machines based on phone number
* Directly from QR-codes on screens, posters, etc.

Users can create a new account through sharing high-quality data from the user’s Vipps profile. Available information includes name, email, addresses, phone number, and birth date. [Norwegian national identity number](vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) and [account numbers](vipps-login-api-faq.md#who-can-get-access-to-account-numbers-and-how) are also available to some merchants.  The identity of all Vipps users is verified as part of Vipps' know your customer process, so rest assured that these are real people with correct name and information.

When Vipps Login is used in the browser, the user can choose to be remembered in the browser, enabling automatic sign-ins for later visits. The possibility for the user to be remembered in browser is a key feature of the service and cannot be controlled (turned off) by individual merchants. This means that it is not possible to set up Vipps Login is a 2 factor authentication (2FA).

Vipps Login is _not_ an electronic identification or certified eID.

For **Vipps Login in browser** there are **plugins** available for several platforms. You find the complete list on our [plugins page](https://github.com/vippsas/vipps-plugins).
If a plugin is not available, the easiest - and **strongly recommended - way to integrate with the service is to use a well renowned OAuth2.0/OpenID Connect Library for your programming language**. Vipps does not recommend a specific library, but the list of [OIDC Relying Party libraries](https://openid.net/developers/certified/) certified by the OpenID Foundation is a good starting point.

Vipps Login in browser should only be run in the browser window using redirects (iFrame is not supported and new window is not recommended).

[**Vipps Login from phone number**](vipps-login-api.md#vipps-login-from-phone-number) is based on the Client initiated backchannel authentication (CIBA) standard and have been developed to support use cases where it is the merchant/merchant's systems that trigger the authentication/registration and thus login cannot be done in the user's browser.

[**Vipps Login from QR-code**](#vipps-login-from-qr-code) is a new flow that we will gradually roll out support for. Our [How It Works page](vipps-login-from-QR-api-howitworks.md) shows the flow.

See our [developer section](https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md#vipps-test-apps) for information about our test environment, test apps and test users.

### Activation

See the FAQ:
[How can I activate and set up Vipps Login?](vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login)

## Flows

### Vipps Login in browser

The generic steps in the Vipps Login in browser flow are shown in our [How it works guide](vipps-login-api-howitworks.md). This also explains how the login in browser flow adapts to different preconditions in regards to whether the user has chosen to be remembered in the browser they are currently using and whether the user has already consented to share information with the specific merchant (sale unit). Below is a more detailed description on alternative flows and the choices that can be done to impact these flows when implementing Vipps Login.

#### Remembered flow

If a user has chosen to be remembered in browser, then the authentication can be completed in the browser. The user will then either be asked to provide consent to share profile information or be logged in directly. This applies to both desktop and mobile.

If the user is not remembered the user needs to confirm the login in the Vipps-app. The flow associated with this will differ depending on whether the user is on desktop or mobile:

#### Desktop flow - phone number based push flow

If the user is on desktop and not remembered in browser, then they will follow this flow. If the user is remembered in browser, then only the consent flow at the bottom will be completed. If the user already has provided consent, then this step will be skipped also, allowing a direct login experience.

The user initiates the login by entering the phone number and selecting whether to be remembered in the browser:
![Number input in desktop](images/Number_input_flow_desktop1.png)

The user goes to the Vipps app and confirms the login:
![Confirmation in app in phone number flow](images/Number_input_flow_app.png)

The user is then authenticated in browser and can provide consent if required. Then the user is redirected back to the redirect URI provided by merchant:
![Consent in desktop](images/Number_input_flow_desktop2.png )

#### Mobile flow - deeplink based flow

If the user is on a mobile device, the Vipps landing page in the browser will automatically trigger a deeplink to the Vipps app if the user is not remembered in the browser. The user will not be prompted to enter the phone number.

In the Vipps app, the user confirms the login and can choose whether to be remembered in the browser for later logins. After confirming in the app, the user needs to switch back to the Vipps page in the browser/app. On the Vipps landing page, the user will finalise the authentication and provide consents if required. The user is then redirected back to the redirect URI provided by merchant (could be webpage or an app).

![Mobile flow with app-switch](images/Mobile_flow_with_partial-app_switch.png)

Apps should follow the [recommendations](#using-vipps-login-in-native-applications) to use correct browser types for their platform.

There are two specialised flows that merchants can use to automatically switch the user back from the Vipps-app to the originating browser/app upon login confirmation. From the illustration above, this means that the page "Gå tilbake til butikken" will be skipped and that the "manual app switch" will be replaced by an automatic app-switch (eg. deeplink). These flows give a better user experience than the standard flow, but they also require the merchant to handle some more complexity in the integration.

Which of the flows to use is controlled with the initiation of the individual login session. The merchant can thus use all available login flows on the same client_id and adapt to the different use cases and login scenarios.

The flows are described below.

##### App to app flow

This flow is designed to be used with apps. It requires that the app initiate Vipps Login in an external browser that is opened within the app, see [specification](#using-vipps-login-in-native-applications). In this flow the merchant need to specify the app URI where the user will be returned after completing the confirmation in the Vipps app.

See [how to implement](vipps-login-api.md#app-integration).

##### Automatic return from Vipps app (requires the merchant to handle user session cross browsers)

This flow is designed for web-pages that would like to have the user automatically returned to a browser after completing the confirmation in the Vipps app. Note that there are security implications by using this flow. **It is not suited for every scenario. Merchants must make their own considerations to ensure that it is only used where suitable**.

Due to how the different mobile operating systems handle app-switch to browser, the user can be returned to a different browser than the one he/she started in. On iOS the user can e.g. start the login in Chrome and be returned to Safari after confirming in the Vipps app. This means that the merchant site cannot rely on cookies being present in the browser the user is returned to.

By using this flow Vipps Login will be able to complete the login process even if the user ends up in a different browser. However, the merchant **must ensure that logins can complete, even without session information like cookies.**

See [how to implement](#automatic-return-from-vipps-app), including more information on the security considerations.

#### No dialog flow - log the user in directly when possible

This flow can be used to log in the user directly, if the required prerequisites are in place. If the prerequisites are not in place, then the Vipps Login process will be stopped and no interaction will be requested from the user in this flow. When using this flow, a spinner will be shown while Vipps Login tries to log the user in. Once the process is completed, the user will be returned to the merchant as in the ordinary Vipps Login flow. As with the other Vipps Login flow, it is recommended to run Vipps Login in a redirect mode and iFrame is not supported.

The user will be logged in with this flow if:

* they are remembered in the browser and no consent is required
* they are remembered in the browser and consent has previously been given

Illustration of how the flow can look when the user clicks "Logg inn" on the front page:

![No dialog flow](images/No_dialog_flow.png)

Possible use cases includes:

* When the user is going to a section of your site/service that requires them to be logged in, e.g. my page or a personalized chatbot. This might be when the user is already on the webpage, or if the user is being linked directly to the page from an email/newsletter.
* When a user clicks on your login option, it is possible to try to log the user in with Vipps first.

In such scenarios, a user that can be logged in directly will get an even better login experience and quickly come to the information and services that are relevant for them.

If the user cannot be logged in directly, you can e.g. show them your ordinary login screen. On the login screen, Vipps will be an option. A user that is not remembered in browser or has not yet given consent can actively login with Vipps from here.

See [how to implement](#no-dialog-flow).

### Vipps Login from phone number

Vipps Login from phone number is based on the Client Initiated Backchannel Authentication (CIBA) standard and built to support special cases where login does not start in browser or app. This means that it is the merchant/merchant's systems that trigger the authentication/registration and thus login cannot be done in the user's browser. This could typically be physical contexts e.g point of sales solutions, on the phone e.g call-center solutions or devices/terminals e.g TV-boxes. To ensure a consistent user experience on webpages and in apps, it is not allowed to use Vipps Login from phone number for such use cases.

Since Vipps Login from phone number is reserved for such special cases, it needs to be specially enabled by Vipps for eligible sale units. [Description on how to order Vipps Login from phone number can be found here](vipps-login-api-faq.md#who-can-get-access-to-vipps-login-from-phone-number-and-how). A sale unit can be set up with both Vipps Login in browser and from phone number (and QR-code) enabled. It is required to use the same sale unit for all use cases to ensure you, as a merchant, get the same user id ('sub') on the user across different scenarios.

Vipps Login from phone number is initiated using the user's mobile number. This triggers a push message from the user's Vipps app. By clicking the push message, the user is taken to Vipps to confirm the authentication/registration. If the user has not already consented to share information with a merchant, such consent will be required. If the user has not enabled push from Vipps, they need to manually open the Vipps app and possibly pull the home screen down for a refresh to receive the authentication request.

The merchant controls whether the user should get the confirmation of completion in the Vipps app or if they should be taken to the merchant's web page to finalise the flow. The merchant can e.g. take the user to their web page to enable input of more information, accept terms and condition, log the user in at their web page, show relevant information/offers or to continue to set up an agreement or completing a purchase. This is illustrated in [How It Works](vipps-login-from-phone-number-api-howitworks.md).

Illustration of how the flow will look like when the user end the flow and get the confirmation of completion in the Vipps app:

![Confirm completion in Vipps app](images/CIBA_flow_in_app.png)

Illustration of how the flow will look like if the user is taken to the merchant's web page:

![Redirect to browser](images/CIBA_flow_take_to_merchant.png)

The merchant has the option to show a confirmation code (`binding_message`) to the user in the app for added security:

![Optional confirmation code (`binding_message`)](images/CIBA_Confirmation_code.png)

See [how to implement](#integrating-with-vipps-login-from-phone-number).

### Vipps Login from QR-code

With Vipps Login from QR code you can retrieve user data and log users in through a QR code.

For an illustration of the flows that will be supported see [How It Works](vipps-login-from-QR-api-howitworks.md).

Integration details and developer documentation can be [found here](#integrating-with-vipps-login-from-qr-code).

### Vipps Login + Vipps Recurring

* [Vipps Login + Vipps Recurring](vipps-login-recurring-howitworks.md): Learn how you can use Login and Recurring together.

## Design guidelines and buttons

Buttons to use for Vipps Login can be found as part of our
[design guidelines](https://github.com/vippsas/vipps-design-guidelines/tree/master/vipps-buttons).

## Core concepts

Vipps Login adheres to the OAuth2 and OpenID Connect standards. The core concepts related to these are presented below. The next chapter goes into the actual implementation.

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

You can read more at the [OIDC standard](https://openid.net/specs/openid-connect-core-1_0.html#IDToken).

It is important to validate the Id-token before using any data contained in it.
See the OIDC standard on
[Id-token validation](https://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation)
for the specifics. We recommend that you use a library for this. A good place
to start is finding a library for your language at [jwt.io](https://jwt.io/#libraries-io).

### Access token

Access tokens are random strings that represent the authorization of a
specific application to access specific parts of a user’s data.
The token itself does not provide any information, but it can be used to
fetch the data that the end-user has consented to share from the
[userinfo endpoint](#userinfo).
Access tokens _must_ be kept confidential in transit and storage.

Example:

```text
"hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus"
```

For more information see [RFC-6749 section 4.1.3-4.1.4](https://tools.ietf.org/html/rfc6749#section-4.1.3).

### Refresh token

Vipps Login does not currently support refresh tokens.

### Token endpoint authentication method

The token endpoint is a standard OIDC endpoint used for requesting Access and ID Tokens.
The default token endpoint authentication method is `client_secret_basic`.

It is possible to change the authentication method to `client_secret_post` in the Vipps portal. This setting will then apply to all login transactions on this sales unit.
Go to [How can I use client_secret_post for authentication?](vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication) for more information on how to change the authentication method.

For more information on the token endpoint see [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#TokenEndpoint) and [RFC-6749 section 3.2](https://tools.ietf.org/html/rfc6749#section-3.2).

## Scopes

Scopes are space-separated lists of identifiers used to specify what access privileges are being requested.
Vipps Login currently supports the following scopes:

| Scopes      | Description                                    | User consent required  |
| ------------| -----------------------------------------------|-------- |
| openid      | Scope used to request an Id-token. It provides the claim “sub” which is a unique id for the end user at that particular merchant. Note: Different merchants will get different subs for the same end user.              |   no    |
| address     | User can have up to three addresses in Vipps: home, work and other. Users' addresses are given as claims 'address' and 'other_addresses'. The claim 'address' returns the address set as 'default' for the Vipps user. And the claim 'other_addresses' returns all other addresses of the end user, if any.  We recommend that merchants fetch all addresses on a user and allow the user to choose which address to use in the relevant context. Some users will not have any registered address, in these situations the claim 'address' will be delivered, but the sub claims in address will be empty strings, e.i. "address" : {"country" : "", "street_address" : "", "address_type" : "", "formatted" : "", "postal_code" : "", "region" : "" } . If a user has information in the «Unit, floor or other details» field this will be included in the "street_address" respons. The «Street address» will then be presented first before "\n" and then the contents from «Unit, floor or other details», e.g: "Suburbia 23"\nUnit B5"|   yes   |
| birthDate   | User birth date (verified with National Population Register)                               |   yes   |
| email       | User email (verified), the flag "email_verified : true" in the response can be used by merchant to confirm for each request that the email actually is verified                                   |   yes   |
| name        | User first, middle and given name (verified with National Population Register)              |   yes   |
| phoneNumber | Verified phone number (verified - the number used with Vipps)                          |   yes   |
| nin        | Norwegian national identity number (verified with National Population Register). NB: merchants need to apply for access to NIN. Go to [Who can get access to NIN and how?](vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) For more information |   yes      |
| accountNumbers | User bank account numbers. NB: merchants need to apply for access to accountNumbers. Go to [Who can get access to account numbers and how?](vipps-login-api-faq.md#who-can-get-access-to-accountnumbers-and-how) For more information |   yes      |

When requesting scopes that require user consent, a view listing these scopes
will be displayed to the user with the option to allow or deny the consent
request. This view is skipped if no scopes requiring consent are requested.
The user can not make changes to the list of requested scopes, and can
therefore not accept for example name and deny address.

We recommend asking for the minimal number of scopes needed for your use case to
minimize the number of users that deny the consent request.

## Recommendations on linking to user account

To ensure the best user experience, we recommend performing the following checks
related to login/registration:

First check if you already have the unique user identifier for Vipps ("ID" from
now on, and called `sub` in the response from our API) stored on one of your
accounts. If you have it, this means that the user has used Vipps on your site
earlier and have an explicit link to the account. In this case use the ID to log
the user into her account.

The `sub` is based on the user's national identity number ("fødselsnummer"
in Norway), and does not change (except in very special cases).

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

If the user does not have an existing account, but this is required for some reason, e.g. because you have a separate sign-up process or the users need to be an existing member/customer, you should provide the user with a message explaining the situation after Vipps Login has finished. If relevant you should provide links or directions on how to create the required account.

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
| Test        |<https://apitest.vipps.no/access-management-1.0/access/.well-known/openid-configuration> |
| Production  |<https://api.vipps.no/access-management-1.0/access/.well-known/openid-configuration>     |

The OpenID connect discovery endpoint can be used to retrieve configuration information for openid connect clients. We recommend to fetch these dynamically, however the response from this endpoint rarely changes. Therefore it can and should be cached so it's not fetched over the network on every login. The endpoint responds with a `Cache-Control: max-age=3600` header.

You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-discovery-1_0.html).

**Request**

[`GET:/.well-known/openid-configuration`][login-discoveropenid-endpoint]

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
  "backchannel_logout_supported": false,
  "backchannel_logout_session_supported": false,
  "frontchannel_logout_supported": false,
  "frontchannel_logout_session_supported": false,
  "end_session_endpoint": "https://apitest.vipps.no/access-management-1.0/access/oauth2/sessions/logout"
}
```

#### Openid connect discovery URLs

| Operation                 | Description         | Endpoints |
| ------------------------- | ------------------- | ------------------ |
| [OAuth 2.0 Authorize](#oauth-20-authorize)            | Start an OAuth 2.0 authorization. | [`GET:/oauth2/auth`][access-auth-endpoint] |
| [OAuth 2.0 Token](#oauth-20-token)                    | Get an OAuth 2.0 access token. | [`POST:/oauth2/token`][access-token-endpoint] |
| [Userinfo](#userinfo)                                 | Returns information that the user has consented to share. | [`GET:/vipps-userinfo-api/userinfo`][userinfo-endpoint] |
| [JSON Web Keys Discovery](#json-web-keys-discovery)   | Get JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens. | [`GET:/.well-known/jwks.json`][login-wellknown-endpoint] |

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
| app_callback_uri  | Optional. The target uri for automatic switch back to merchant app. Requires `requested_flow=app_to_app`. Example `merchant-app://callback`
| final_redirect_is_app | Optional. Either `true` or `false`. If this is `true` we will enable some compatibility features to make sure the user is returned to the app.                                                             |
| code_challenge_method | Optional. Used for [PKCE](https://datatracker.ietf.org/doc/html/rfc7636), either `S256` or `plain`. Default value is `plain` |
| code_challenge | Optional. Used for [PKCE](https://datatracker.ietf.org/doc/html/rfc7636). The value must be calculated based on the `code_verifier` later used towards the [token endpoint](#oauth-20-token)

For example, the client directs the user-agent to make the following HTTP request:

[`GET:/oauth2/auth?client_id={client_id}&response_type=code&scope={scopes}&state={state}&redirect_uri={redirect_uri}`][access-auth-endpoint]

You can test this by entering the url into any browser. This will initiate the log in sequence.
For more information about testing this with the Postman collection, see the step-by-step instructions in the [quick start guide](vipps-login-api-quick-start.md).

**Please note:** URIs specified on [portal.vipps.no](https://portal.vipps.no/)
must be _exactly_ the same as used in the API calls. Be extra careful with
trailing `/` and URL-encoded entities. If the URIs are not identical you will get
this error:

>The provided authorization grant (e.g., authorization code, resource owner credentials) or refresh token is invalid, expired, revoked, does not match the redirection URI used in the authorization request, or was issued to another client

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

```http
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

_Headers_

| Header            | Description                            |
| ----------------- | -------------------------------------  |
| Content-Type      | "application/x-www-form-urlencoded"    |
| Authorization     | "Basic {Client Credentials}"           |

The Client Credentials is a base 64 encoded string consisting of the Client id
and secret issued by Vipps joined by ":"

Example in JavaScript:

```javascript
var client_id = 123456-test-4a3d-a47c-412136fd0871
var client_secret = testdzlJbUZaM1lqODlnUUtrUHI=

var wordArrayAzp = CryptoJS.enc.Utf8.parse(client_id + ":" + client_secret);
var client_authorization = CryptoJS.enc.Base64.stringify(wordArrayAzp);
```

_Form content_

| Key               | Description                                                                                                                                                              |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| grant_type        | Value MUST be authorization_code.                                                                                                                                        |
| code              | The authorization code received from the authorization server.                                                                                                           |
| redirect_uri      | Redirect URL which the user agent is redirected to after finishing a login. If the URL is using a custom URL scheme, such as `myapp://`, a path is required: `myapp://path-to-something`. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-from-the-merchant) . This field is required for OIDC flows, i.e. regular Vipps Login logins. |                                                                                                                                    |

[`POST:/oauth2/token`][access-token-endpoint]

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

##### Userinfo

This endpoint returns the payload with the information that the user has consented to share, which is provided in
the OAuth 2.0 access token.
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo).

**Request**

_Headers_

| Header            | Description                            |
| ----------------- | -------------------------------------  |
| Authorization     | "Bearer {Access Token}"                |

The access token is received on a successful request to the [token endpoint](#oauth-20-token).

[`GET:/vipps-userinfo-api/userinfo`][userinfo-endpoint]

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

##### JSON Web Keys Discovery

This endpoint returns JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens and if enabled,
OAuth 2.0 JWT Access Tokens.

**Request**

[`GET:/.well-known/jwks.json`][login-wellknown-endpoint]

**Response**

Overview

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `500 Server Error`      | An internal Vipps problem.                              |

Examples:
_200 response_

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

This operation does not require authentication.

### API endpoints required from the merchant

The following endpoints are to be implemented by merchants, in order for Vipps
to redirect the resource owner to them.

#### Receive authentication result

After a successful authentication, the user agent is redirected to this endpoint
with the following parameters added to the query component. This URI needs to
be pre-registered with Vipps and supplied as a query parameter on calls to the
[OAuth2 authorize endpoint](#oauth-20-authorize).

| Param             | Description|
| ----------------- | ---------- |
| code              | The authorization code generated by the authorization server. The client MUST NOT use the authorization code more than once. The authorization code is bound to the client identifier and redirection URI.    |
| state             | The exact value received from the client during the authorization request.|

Example:

```http
HTTP/1.1 302 Found
Location: https://client.example.com/callback?code={code}&state={state}
```

### Error handling

If the user cancels the login or an error occurs, the user agent is redirected
to the receive authentication result endpoint with the following parameters
added to the query component.

| Param             | Description                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| error             | Standard [OAuth2](https://tools.ietf.org/html/rfc6749#section-4.1.2.1) or [OIDC](https://openid.net/specs/openid-connect-core-1_0.html#AuthError) error code. |
| error_description | A short text providing additional information on the error that occurred.                                                                                     |
| state             | The exact value received from the client during the authorization request.                                                                                    |

#### Custom error codes

In addition to the standard errors defined in [OAuth2](https://tools.ietf.org/html/rfc6749#section-4.1.2.1) and [OIDC](https://openid.net/specs/openid-connect-core-1_0.html#AuthError), Vipps Login also has some custom error codes that you might see on the redirect:

| error_code                           | Description                                                       |
| ------------------------------------ | ----------------------------------------------------------------- |
| `access_denied`                      | User cancelled the login                                          |
| `server_error`                       | Something went wrong, please try again                            |
| `login_required`                     | User must login with interaction                                  |
| `invalid_app_callback_uri`           | The app callback uri is not on a valid format                     |
| `app_callback_uri_not_registered`    | The app callback uri is not registered as a redirect uri          |
| `outdated_app_version`               | The user's Vipps app version is too old and needs to be updated   |
| `wrong_challenge`                    | The user selected the wrong challenge                             |
| `unknown_reject_reason`              | Something went wrong. Reject reason is unknown.                   |

There may be other errors, so integrators must be able to handle undocumented errors gracefully.

Example:

```http
HTTP/1.1 302 Found
Location: https://client.example.com/callback?error=access_denied&error_description=user%20cancelled%20the%20login?state={state}
```

If a fatal error occurs where the user can not be redirected back to the merchant, a generic Vipps styled error page will be shown containing a brief error description.

### Call by call

A very basic case, from a user clicks "Log in with Vipps" until the merchant receives login token
and the users information, is shown below:

0. Before all this, the merchant has fetched the openid configuration from the well-known endpoint
   and cached it.

   See [.well-known](#openid-connect-discovery-endpoint).

1. The merchant initiates a login by calling the `authorization_endpoint` from .well-known.
   `GET {authorization_endpoint}?client_id={client_id}&response_type=code&scope={scopes}&state={state}&redirect_uri={redirect_uri}`

   See [Authorization endpoint](#oauth-20-authorize).

2. This will bring the user to the Vipps Login-screen, where they will consent to sharing
   information with the merchant. After consenting, the browser is redirected to the
   `redirect_uri` supplied by the merchant.
   `{redirect_uri}?code={code}&state={state}&scope={scopes}`

3. The merchant uses the `code`-parameter to obtain the login token.
    `POST {token_endpoint}` with `code={code}`, `grant_type=authorization_code`, and
   `redirect_uri={redirect_uri}` in the `application/x-www-form-urlencoded`-body.
   This returns (amongst others) an `access_token` that can be used to fetch userinfo.

    See [Token endpoint](#oauth-20-token).

4. To obtain userinfo, the merchant must do a GET to the `userinfo_endpoint` with the header:
    `Authorization: Bearer {access_token}`. This returns the information the user consented to sharing.

    See [Userinfo](#userinfo).

### Using Vipps Login in native applications

Web-views should not be used when using Vipps Login in a native application. Instead, the user should be redirected using an external browser openend by/within the app.

Android: Use [Custom Tabs](https://developer.chrome.com/multidevice/android/customtabs) or fallback to open external browser on user's device.

iOS: Use [SFAuthenticationSession](https://developer.apple.com/documentation/safariservices/sfauthenticationsession) (for iOS 11 and 12) or [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession) (for iOS 13 and above).

## Using the special flows

### App integration

It is possible to enable automatic switch of users back to the merchant app,
from the Vipps app. This flow is described [here](#app-to-app-flow).

Expected flow:

```text
Merchant app -> Merchant app controlled browser -> Vipps app -> Merchant app controlled browser -> Merchant app
```

Note that you should use an
[external browser that is opened within the app, and not web-views](#using-vipps-login-in-native-applications).

This flow can be enabled per login request by adding the `requested_flow=app_to_app`
and `app_callback_uri` parameters to the [Authorize](#oauth-20-authorize) request.

This flow requires both the `app_callback_uri` and `redirect_uri` parameters.

The `app_callback_uri` should be a URI that makes the device switch back to the merchant's
app again after the Vipps app portion of the flow is done (example: "merchant-app://callback").

The `redirect_uri` is opened in the browser once the Vipps login flow is completed there.
This URL can either redirect the user to a page in the browser, or be handled/intercepted by the merchant app.
In either case it is important to avoid using static client secrets in the app for completing the login.
(For more information see <https://github.com/openid/AppAuth-Android#utilizing-client-secrets-dangerous> and <https://tools.ietf.org/html/rfc8252#section-8.5>).

_Both_ URIs must be added in the [portal.vipps.no](https://portal.vipps.no/),
you find more information on how to do this [here](vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login).

**Please note:** URIs specified on [portal.vipps.no](https://portal.vipps.no/)
must be _exactly_ the same as used in the API calls. Be extra careful with
trailing `/` and URL-encoded entities. If the URIs are not identical you will get
this error:

>The provided authorization grant (e.g., authorization code, resource owner credentials) or refresh token is invalid, expired, revoked, does not match the redirection URI used in the authorization request, or was issued to another client

The Vipps app will return some data with the return to the `app_callback_uri`.
It contains two query parameters `state` and `resume_uri`.
`State` is the OIDC `state` parameter passed at the start of a login which can be used to identify the specific login if needed.

The `resume_uri` parameter is generated by Vipps
and can optionally be used to resume the login when the user returns from the Vipps app.
If the merchant app manages to keep the initial browser window open it is not required to use the `resume_uri`.
To use the `resume_uri` it must be opened in the same browser used in the initial phase
(such as ASWebAuthenticationSession or Chrome Custom Tabs). It is required that the browser contains the cookies
from the interaction that happens before the user is sent to the Vipps app.

Example `app_callback_uri` request

```http
merchant-app://callback/?state=RFiQdrl_lvJUpVmTRSKmsZRGLM0G1N1qh0WebZ1gDNk&resume_uri=https%3A%2F%2Fapi.vipps.no%2Fvipps-login-idp%2Findex.html%3FtabId%3D7607f7f0-7ae2-49b7-9cb5-102143dac4ea
```

#### A typical flow/implementation might look like this

![A typical flow](images/app2appsuggestion.png)

The dotted lines in this diagram are handled by Vipps (or the user),
while the filled lines need to be implemented by the merchant.

1. Merchant backend generates an [OpenID authentication flow authorization URI](#oauth-20-authorize).
   The URI is communicated to the merchant app.
2. Merchant app uses the URI to initiate Vipps Login in an external browser that is opened within the app,
   see the [specification](#using-vipps-login-in-native-applications) for details.
3. Vipps Login will open the Vipps app, if required.
   (If the user is remembered in the browser, they will be authenticated directly
    and they will then be on step 6 below).
4. Vipps app opens the deep link `app_callback_uri` parameter after the user has approved the login.
5. Merchant app handles the link in one of two ways:
   * Display the same browser instance that was created in 1.
   * Open a browser again using the `resume_uri`-query parameter that is returned with the request to the `app_callback_uri`.
6. Vipps Login finalizes the authentication of the user and obtains consent to share information if needed.
   When this is finished the user will be redirected to the `redirect_uri`.
   The Vipps Login process has now finished and the merchant controls the remaining process.
7. Merchant app sends the `code` and `state`parameters received in the callback to the merchant backend.
8. Merchant backend fetches the access tokens and user information.

Example authorize request URL:

```http
.../oauth2/auth?app_callback_uri=https://example.com/app-callback&requested_flow=app_to_app&scope=<scopes>&response_type=code&redirect_uri=merchantapp://callback&code_challenge_method=S256&state=<state>&nonce=<nonce>&client_id=<clientid>&code_challenge=<challenge>
```

Parameters `state` and possibly `error` will be passed as query parameters to the `app_callback_uri`. The `state` parameter has the same value as the `state` parameter passed to the [Authorize](#oauth-20-authorize) request.

Example success callback from app (step 4):

```http
https://example.com/app-callback?state=218gz18yveu1ybajwh2g1h3g&resume_uri=https://.../vipps-login-idp/index.html?tabid=fjeiwfje43t
merchantapp://app-callback?state=218gz18yveu1ybajwh2g1h3g&resume_uri=https://.../vipps-login-idp/index.html?tabid=fjeiwfje43t
```

Example error callback from app (step 4):

```http
https://example.com/app-callback?state=218gz18yveu1ybajwh2g1h3g?error=unknown_error
merchantapp://app-callback?state=218gz18yveu1ybajwh2g1h3g?error=unknown_error
```

Example success callback from browser (step 6):

```http
https://example.com/redirect-uri?state=218gz18yveu1ybajwh2g1h3g&code=<code>&scope=openid
merchantapp://redirect-uri?state=218gz18yveu1ybajwh2g1h3g&code=<code>&scope=openid
```

### Automatic return from Vipps app

When enabled, this flow will automatically take the user back to a browser when they accept the login in the Vipps app. It is not suitable for every scenario. Please see [the detailed description](#automatic-return-from-vipps-app-requires-the-merchant-to-handle-user-session-cross-browsers) and be aware of the security implications mentioned there.

This flow can be enabled per login by adding the parameter `requested_flow=automatic_return_from_vipps_app` to the [Authorize](#oauth-20-authorize) request.

#### Implementation suggestions

**It is not possible to give a single description that ensures secure use of this flow for all scenarios. The suggestions given here may not apply to every scenario and must be considered in relation to the specifics of the implementation.**

##### Session information

The [state parameter](vipps-login-api-faq.md#whats-the-purpose-of-the-state-parameter)  passed in the [OAuth2 authorize endpoint](#oauth-20-authorize)) request can carry some information from the start of a login until the callback.
The state parameter cannot be thought of as a direct replacement of a user agent bound session.

Some relevant considerations:

* Always use [PKCE](https://oauth.net/2/pkce/).
* Avoid logging the callback URI.
* [Session fixation](https://owasp.org/www-community/attacks/Session_fixation). Be aware of what is possible to set up before a login is started.
* [Login CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#login-csrf). Be aware if it's possible to input sensitive information after a login.
* [The state parameter and CSRF](https://tools.ietf.org/html/rfc6749#section-10.12). Be aware of the recommendations of the OIDC/OAuth standards.

##### Verification

It is important that merchants verify that users returning to a different browser than where the login started are handled as expected. It is also recommended to test starting the login in private/incognito mode, as this will have similar effects as being returned to a different browser.

### No dialog flow

This flow is described [here](#no-dialog-flow---log-the-user-in-directly-when-possible). To attempt a no dialog login, add the query parameter `requested_flow=no_dialog` to the [Authorization request](#oauth-20-authorize) uri.

If the user completes the login, they will be returned to the `redirect_uri` with a code that can be used to complete the login, just like in a regular [Authorization code](#authorization-code-grant) login.

`https://client.example.com/callback?code=12adsa938721987321asd9873&state=123987218dh`

If the user is not logged in, they will be returned with an error. Some possible errors are `interaction_required`, `login_required` or `server_error`.

Not logged in return uri example: `https://client.example.com/callback?error=interaction_required&error_description=User+interaction+is+required&state=1312312321983212a3b`.

In all cases, a new login can be started by removing the parameter `requested_flow=no_dialog` and initiating a new login for the user.

## Integrating with Vipps Login from phone number

### Activating Vipps Login from phone number

Vipps Login from phone number (CIBA flows) has been developed to support use cases where authentication/registration does not start in a browser or an app. These flows are described [here](#vipps-login-from-phone-number). They are based on the CIBA OIDC standard <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html>.

Vipps Login from phone number is reserved for special cases and needs to be specially enabled by Vipps for eligible sale units. Instructions can be found at [Who can get access to Vipps Login from phone number and how](vipps-login-api-faq.md#who-can-get-access-to-vipps-login-from-phone-number-and-how). 

A sale unit can be set up with both Vipps Login in browser and phone number (and QR-code) enabled, and it is required to use the same sale unit for all use cases to ensure that you, as a merchant, get the same user id ('sub') on the user across different scenarios. To ensure a consistent user experience on webpages and in apps, it is not allowed to use Vipps Login from phone number for such use cases.

### Complete login in the Vipps app

#### Activation

See [Activating Vipps Login from phone number](#activating-vipps-login-from-phone-number)

#### Overview

Client-Initiated Backchannel Authentication (CIBA) enables a Client to initiate the authentication of an end-user through out-of-band mechanisms.

1) The Client shall make an "HTTP POST" request to the Backchannel Authentication Endpoint to ask for end-user authentication.
2) Vipps Login will respond immediately with a unique identifier that identifies that authentication while it tries to authenticate the user in the background.
3) The Client will receive the ID Token and Access Token by polling the token endpoint to get a response with the tokens.

#### Call by call

0. Before all this, the merchant has fetched the openid configuration from the well-known endpoint and cached it.
   See [.well-known](#openid-connect-discovery-endpoint)

1. The merchant initiates a login by calling the `backchannel_authentication_endpoint` listed in the openid configuration fetched in step 0.

    For details see [Authentication Request](#authentication-request-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-1_0htmlauth_request).

    Example request:

    ```http
    POST https://api.vipps.no/vipps-login-ciba/api/backchannel/authentication
    Authorization: Basic asdkjhasdjhsad=
    Content-Type: application/x-www-form-urlencoded

    scope=name address openid&login_hint=urn:mobilenumber:{mobileNumber}&state=13821s837213bng26e2n61gege26&nonce=21hebdhwqdb7261bd1b23
    ```

    Example response:

    ```json
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

    For other details about the request see [Token request](#token-request-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-1_0htmlrfcsection101).

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

#### Authentication Request (<https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request>)

The Backchannel Authentication Endpoint is listed as `backchannel_authentication_endpoint` in the configuration <https://api.vipps.no/access-management-1.0/access/.well-known/openid-configuration>.

##### Authentication

The following authentication methods are currently supported:

* client_secret_basic
* client_secret_post

The default token endpoint authentication method is `client_secret_basic`. It is possible to change the authentication method to `client_secret_post` in the Vipps portal. [More information in the FAQ](vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication).

##### The `login_hint` parameter (required)

Supported login hints:

* Norwegian mobile phone numbers can be targeted by passing login hint's on the format `urn:mobilenumber:{8 digit norwegian mobile number}`.

Example: `...&login_hint=urn:mobilenumber:12345678&...`.

##### The `scope` parameter (required)

* We support the scopes listed at [Scopes](#scopes)
* The legacy `nnin` scope is not supported, use `nin` instead.

Example: `...&scope=name address birthDate nin&...`

##### The `binding_message` parameter (optional)

A human-readable identifier or message intended to be displayed on both the consumption device and the authentication device to interlock them together for the transaction by way of a visual cue for the end-user. It should not be used for attempting to conveying other information.

##### Format

The format possible for this field is limited to capital characters 'A-Z', numbers '0-9' and the character '-'. It must also be bewteen 5 and 8 characters long. Regex: ```^[A-Z0-9\\-]{5,8}$```.

Read more about it in the standard <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request>.

Example: `....&binding_message=4MZ-CQ3&...`

##### Error responses

In addition to the responses defined by the standard these responses might be returned:

* `429` status responses: Too many login requests started towards the same user at the same time. Please respect the `Retry-After` header returned.

##### Successful responses (<https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#successful_authentication_request_acknowdlegment>)

Responses according to the standard. Note we do return an `interval` parameter which indicates the minimum amount of time in seconds that the Client MUST wait between polling requests to the token endpoint.

#### Token request (<https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.10.1>)

The responses from this endpoint is according to the standard.

* Note the required `grant_type`: `urn:openid:params:grant-type:ciba`.
* The access token can be used towards the standard [oidc userinfo endpoint](#userinfo).

##### Polling

* Long polling as described in [the CIBA standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#token_request) is currently not supported
* Remember not to poll more often than indicated by the `interval` parameter returned from the [authentication request](#authentication-request-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-1_0htmlauth_request).

#### Error responses

In addition to the responses defined by the [standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.11) these responses might be returned:

* `error_code=old_app`: The user's Vipps app is outdated and does not support this login flow.
* `error_code=invalid_user`: No account exists, the user's account is not active or the user is in some way not eligible to use this login flow currently e.g. U15 users.

### Redirect to browser

#### Activation

See [Activating Vipps Login from phone number](#activating-vipps-login-from-phone-number).

#### Overview

This CIBA-related flow enables a Client to initiate the authentication of an end-user through out-of-band mechanisms and additionally facilitates the end user
to be taken to the client's web page to finalise the flow.

1) The Client shall make an "HTTP POST" request to the Backchannel Authentication Endpoint to ask for end-user authentication.
2) The user, via their browser, will be redirected to the Client's `redirect_uri` which enables the login to be completed.

![image](https://user-images.githubusercontent.com/1453728/134330475-d6910cbd-c216-4c64-bef9-e0f84cb0a7fd.png)

#### Call by call

0. Before all this, the merchant has fetched the openid configuration from the well-known endpoint and cached it.
   See [.well-known](#openid-connect-discovery-endpoint).

1. The merchant initiates a login by calling the `backchannel_authentication_endpoint` listed in the openid configuration fetched in step 0.

    For details see [Authentication Request With Redirect](vipps-login-api.md#authentication-request-with-redirect-httpsopenidnetspecsopenid-client-initiated-backchannel-authentication-core-1_0htmlauth_request).

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
The ID token is a JWS that must be validated, see [ID Token](#id-token). The merchant **must validate** that it contains the `auth_req_id` they have previously received from step 2.

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
      "token_type": "bearer",
    }
    ```

    Decoded ID token JWS example:

    ```json
    {
      "kid": "public:ee36d3f5-3934-4029-926f-77fa65bf0b4b",
      "alg": "ES256"
    }.{
      "aud": "edddb32f-5028-4397-b0ab-e8ecf218fdc2",
      "sub": "5168ee06-871e-46e6-941e-0103b957048e",
      "auth_req_id": "7BzAY-SbVQJ38V-U0C7Z6k23_CY",
      "iss": "https://ece46ec4-6f9c-489b-8fe5-146a89e11635.tech-02.net/access-management-1.0/access/",
      "exp": 1643179837,
      "iat": 1643179237
    }.[Signature]
    ```

4. The merchant must do a GET  to the `userinfo` endpoint with the header: Authorization: Bearer {access_token}, using the access_token retrieved in step 3.

    For details see [Userinfo request](#userinfo).

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

#### Authentication Request With Redirect (<https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request>)

The Backchannel Authentication Endpoint is listed as `backchannel_authentication_endpoint` in the configuration <https://api.vipps.no/access-management-1.0/access/.well-known/openid-configuration>.

##### Authentication

The following authentication methods are currently supported:

* client_secret_basic
* client_secret_post

The default token endpoint authentication method is `client_secret_basic`. It is possible to change the authentication method to `client_secret_post` in the Vipps portal. [More information in the FAQ](vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication).

Required parameters: `requested_flow`, `login_hint`, `scope`, `redirect_uri`

##### The `login_hint` parameter (required)

Supported login hints:

* Norwegian mobile phone numbers can be targeted by passing login hint's on the format `urn:mobilenumber:{8 digit norwegian mobile number}`.

Example: `...&login_hint=urn:mobilenumber:12345678&...`.

##### The `scope` parameter (required)

* We support the scopes listed at [Scopes](#scopes).
* The legacy `nnin` scope is not supported, use `nin` instead.
* The `openid` scope is required

Example: `...&scope=openid name address birthDate nin&...`

##### The `binding_message` parameter (optional)

A human-readable identifier or message intended to be displayed on both the consumption device and the authentication device to interlock them together for the transaction by way of a visual cue for the end-user.

Read more about it in the standard <https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#auth_request>

Note: "the binding_message value SHOULD be relatively short and use a limited set of plain text characters"

Example: `....&binding_message=4MZ-CQ3&...`

##### The `redirect_uri` parameter (required)

Redirect URL which the user agent is redirected to after finishing a login. Must be `https` in the production environment.

Example: `...&redirect_uri=https://merchant.com/callback&...`

##### Error responses

In addition to the responses defined by [the standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.13) these responses might also be returned:

* `429` status responses: Too many login requests started towards the same user at the same time. Please respect the `Retry-After` header returned.
* Most of the [general error codes](vipps-login-api.md#error-handling)

## Integrating with Vipps Login from QR code

[Vipps login from QR api how it works](vipps-login-from-QR-api-howitworks.md)

Note: Vipps Login from QR code is not supported in our merchant test environment at the moment. The full flow can only be completed in the production environment.

### Activating Vipps Login from QR code

To use Vipps Login from QR Code you first need to [activate Vipps Login](vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login) in the Merchant portal. Since Vipps Login From QR code is a billed service, it is not enabled by default. To use Vipps Login from QR Code you therefore need to submit a request to [accessuserinfo@vipps.no](mailto:accessuserinfo@vipps.no) containing your organisation's invoicing details, such as:

* Invoice address of company/organisation
* Name of invoice recipient/approver
* E-mail address we can send the invoice to, if necessary

We currently don't have any self service functionality on QR codes in our merchant portal (yet), so for the integration to work we will also need you to submit some technical details on how you plan to integrate the QR flow:

* Should the QR code redirect the user back to your website? In that case, you need to send us a `redirect uri`. We can redirect the users back to your website after they have authenticated and consented in the Vipps app.
* Should the QR code flow end in the Vipps app? In that case, you need to send us a `callback uri` that we can use to ping your backend service when the user has authenticated and consented in the Vipps app.

With this in place, we can issue a Vipps Login QR code.

### Initiate login from QR code

#### Call by call

Prerequisite:

* Client needs to register a QR code along with a webhook in our system. See [Activating Vipps Login from QR code](#activating-vipps-login-from-qr-code)

Steps:

0. The client has fetched the OpenID configuration from the [.well-known endpoint](#openid-connect-discovery-endpoint) and cached it.

1. The user scans the QR code and then confirms the login in the Vipps app.

2. The client will receive a JWS on the preregistered webhook.

3. The client needs to validate this JWS using the keyset found in 'jwks_uri' under [.well-known](#openid-connect-discovery-endpoint).
   Decoded JWS payload example.

   ```json
   {
       "aud": "acae94b4-7b30-4615-9806-10c3b42079a3",
       "auth_req_id": "6tw7GmfPQRcWuydzlAwrUakpEEw",
       "iss": "https://api.vipps.no/access-management-1.0/access/",
       "exp": 1643981298,
       "iat": 1643980998,
   }
   ```

4. The client exchanges the `auth_req_id` for login tokens by passing it to the `{token_endpoint}`. Perform a POST request, with `content_type=application/x-www-form-urlencoded`, and include the `auth_req_id={auth_req_id}` and `grant_type=urn:vipps:params:grant-type:qr` parameters in the body.

    This returns an ID token and an access token that can be used to fetch userinfo.
    * The access token can be used towards the standard [oidc userinfo endpoint](#userinfo).
    * The ID token is a JWS that must be validated, see ID Token.
    * The claim `qr_id` can be used by the client to identify the specific QR code that the user scanned.
    * Error responses as defined by the [CIBA standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.11).

    Example request:

    ```http
    POST https://api.vipps.no/access-management-1.0/access/oauth2/token
    Authorization: Basic asdkjhasdjhsad=
    Content-Type: application/x-www-form-urlencoded

    grant_type=urn%3Avipps%3Aparams%3Agrant-type%3Aqr&auth_req_id=6tw7GmfPQRcWuydzlAwrUakpEEw
    ```

    Example response:

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

    Decoded ID token JWS example:

    ```json
    {
      "kid": "public:ee36d3f5-3934-4029-926f-77fa65bf0b4b",
      "alg": "ES256"
      }.{
      "aud": "8de3f38f-4a79-4eb3-930f-1e595d460a57",
      "sub": "d6614352-ca55-4d89-8f82-d2facde311a4",
      "iss": "https://api.vipps.no/access-management-1.0/access/",
      "exp": 1643984388,
      "iat": 1643983788,
      "qr_id": "oKAz7q1O",
      "qr_description": "description of QR code"
      }.[Signature]
    ```

5. The client must do a GET to the `userinfo` endpoint with the header: Authorization: Bearer {access_token}, using the access token retrieved in step 4.

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

### Initiate login from QR code with redirect to browser

#### Overview

[Vipps login from QR api how it works](#vipps-login-from-qr-code)

#### Call by call

Prerequisite:

* Client needs to register a QR code along with a redirect uri in our system. See [Activating Vipps Login from QR code](#activating-vipps-login-from-qr-code).

Steps:

0. The client has fetched the OpenID configuration from the [.well-known endpoint](#openid-connect-discovery-endpoint) and cached it.

1. The user scans the QR code and then confirms the login in the Vipps app. The user is then redirected to the preregistered `redirect_uri`. The redirect will contain the query parameter `code`: `{redirect_uri}?code={code}`.

2. The client exchanges the code for login tokens by passing it to the `{token_endpoint}`. Perform a POST request, with `content_type=application/x-www-form-urlencoded`, and include the `code={code}` and `grant_type=urn:vipps:params:grant-type:qr-redirect` parameters in the body.

    This returns an ID token and an access token that can be used to fetch userinfo.
    * The access token can be used towards the standard [oidc userinfo endpoint](#userinfo)
    * The ID token is a JWS that must be validated, see ID Token.
    * The claim `qr_id` can be used by the client to identify the specific QR code that the user scanned.
    * Error responses as defined by the [CIBA standard](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.11).

    Example request (the real payload will likely look different because of encoding):

    ```http
    POST https://api.vipps.no/access-management-1.0/access/oauth2/token
    Authorization: Basic sadlksadkjasjdaksd
    Content-Type: application/x-www-form-urlencoded

    code=some-valid-code&grant_type=urn:vipps:params:grant-type:qr-redirect

    ```

    Example response:

    ```json
    {
       "access_token": "hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus",
       "id_token": "eyJraWQiOiJwdWJsaWM6ZWUzNmQzZjUtMzkzNC00MDI5LTkyNmYtNzdmYTY1YmYwYjRiIiwiYWxnIjoiRVMyNTYifQ.eyJhdWQiOiJlZGRkYjMyZi01MDI4LTQzOTctYjBhYi1lOGVjZjIxOGZkYzIiLCJzdWIiOiI1MTY4ZWUwNi04NzFlLTQ2ZTYtOTQxZS0wMTAzYjk1NzA0OGUiLCJhdXRoUmVxSWQiOiI3QnpBWS1TYlZRSjM4Vi1VMEM3WjZrMjNfQ1kiLCJpc3MiOiJodHRwczpcL1wvZWNlNDZlYzQtNmY5Yy00ODliLThmZTUtMTQ2YTg5ZTExNjM1LnRlY2gtMDIubmV0XC9hY2Nlc3MtbWFuYWdlbWVudC0xLjBcL2FjY2Vzc1wvIiwiZXhwIjoxNjQzMTc5ODM3LCJpYXQiOjE2NDMxNzkyMzd9.iFvmdtRQVliAe91dBu_CZDfBD5I7WCbDTiDQxu4sOTApXFPb5EsSuEBEVfK_-14E7xjcfQLSMa6ZO06YvhRHAA",
       "expires_in": 3599,
         "scope": "openid",
         "token_type": "bearer",
    }
    ```

    Decoded ID token JWS example:

    ```json
    {
    "kid": "public:ee36d3f5-3934-4029-926f-77fa65bf0b4b",
    "alg": "ES256"
    }.{
    "aud": "8de3f38f-4a79-4eb3-930f-1e595d460a57",
    "sub": "d6614352-ca55-4d89-8f82-d2facde311a4",
    "iss": "https://api.vipps.no/access-management-1.0/access/",
    "exp": 1643984388,
    "iat": 1643983788,
    "qr_id": "oKAz7q1O",
    "qr_description": "description of QR code"
    }.[Signature]
    ```

3. The client must do a GET to the `userinfo` endpoint with the header: Authorization: Bearer {access_token}, using the access token retrieved in step 3.

   For details see [Userinfo request](#userinfo).

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

## Consent Webhooks

### Revoke

Users can revoke their consent to share data with merchants. This is done in the profile section -> personal information in the app. If a user removes the consent to share data with a merchant, the practical consequences is that the user needs to give a consent the next time he/she would like to login with Vipps or share data as part of other Vipps services with the merchant.

When a user revokes their consent, Vipps will send an `CONSENT_REVOKED` event containing the unique identifier (sub) for the given user to the webhook provided by the merchant.

This service is optional for the merchant and can be used to trigger events on the merchant's side. I.e. the merchant can inform the user that they still have an account, can provide information on alternative login solutions or where the user should go if they would like to delete the data they have stored with the merchant.
  
**Content**  
The webhook is sent as a `POST` with a `text/plain` body containing an unsigned Json Web Token ([JWT](https://jwt.io/)).  
The JWT format was chosen to allow for the possibility to add signing on a later state, but as of now the JWT  is delivered unsigned with the algorithm set to none and is therefore no more secure than a regular json and should be handled as such.

**Example request:**

```text
eyJhbGciOiJub25lIiwidHlwIjogIkpXVCJ9Cg.eyJleHAiOjE1OTI1NzE3ODgsImlhdCI6MTU5MjU3MTQ4OCwibmJmIjoxNTkyNTcxNDg4LCJzdWIiOiJjOWQxMDQ0NC1kOTkyLTQ4NTAtYWM2MC05ZDM1MDIwOTUwMDgiLCJldmVudCI6IkNPTlNFTlRfUkVWT0tFRCJ9
```

*Decoded JWT*

```json
Header
{
  "alg": "none",
  "typ": "JWT"
}
Payload
{
  "exp": 1592571788,
  "iat": 1592571488,
  "nbf": 1592571488,
  "sub": "c9d10444-d992-4850-ac60-9d3502095008",
  "event": "CONSENT_REVOKED"
}
```

## Questions?

For more information and to order the product go to:

* [Frequently Asked Questions](vipps-login-api-faq.md)

* [Product information in Norwegian](https://www.vipps.no/produkter-og-tjenester/bedrift/innlogging-og-identifisering/logg-inn-med-vipps/)

* [Product FAQ in Norwegian](https://vipps.no/hjelp/vipps/vipps-logg-inn)

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).


[login-wellknown-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/wellKnown
[login-discoveropenid-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/discoverOpenIDConfiguration
[userinfo-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Userinfo-API/operation/userinfoAuthorizationCode
[access-auth-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/oauthAuth
[access-token-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/oauth2Token
