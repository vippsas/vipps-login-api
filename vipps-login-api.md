# Vipps Login API
API version: 0.9  
Document version 0.9

API details can be found at [Swagger UI](https://vippsas.github.io/vipps-login-api/#/) or [ReDoc](https://vippsas.github.io/vipps-login-api/redoc.html)

# Table of contents
* [Introduction](#introduction)
* [Core concepts](#core-concepts)
    * [OAuth 2.0](#oauth-20)
    * [OpenID Connect](#open-id-connect)
        * [Supported OpenID Connect Flows](#supported-openid-connect-flows)
            * [Authorization Code Grant](#authorization-code-grant)
    * [Tokens](#tokens)
        * [ID Token](#id-token)
        * [Access Token](#access-token)
        * [Refresh Token](#refresh-token)
    * [Scopes](#scopes)
* [Integrating with Vipps Login](#integrating-with-vipps-login)
    * [Manual integration](#manual-integration)
        * [Base URLs](#base-urls)
        * [API endpoints](#api-endpoints)
            * [OAuth 2.0 Authorize](#oauth-20-authorize)
            * [OAuth 2.0 Token](#oauth-20-token)
            * [OpenID Connect Userinfo](#openid-connect-userinfo)
            * [OpenID Connect Discovery](#openid-connect-discovery)
            * [JSON Web Keys Discovery](#json-web-keys-discovery)
        * [Automatic Recognition](#automatic-recognition)
    * [API endpoints required from the merchant](#api-endpoints-required-from-the-merchant)
      * [Receive authentication result](#receive-authentication-result)
      * [Error handling](#error-handling)
    
# Introduction
The Vipps Login API offers functionality for authenticating end users and authorizing clients founded on the OAuth2 and 
OpenID Connect specifications. It supports using web browsers on websites and in native apps for iOS and Android using app switching. 

**NB!**  
This service is currently in a pre-release version. This documentation will be expanded and is likely to change as we move closer to a public release.   


# Core concepts
## OAuth 2.0
[OAuth 2.0](https://tools.ietf.org/html/rfc6749) is the industry-standard protocol for authorization. 
Giving a proper introduction to the standard is out of the scope of this documentation, but there are many excellent resources 
on the web. If you are new to the subject we recommend this [talk](https://www.youtube.com/watch?v=996OiexHze0 ) by Nate Barbettini at Okta.
We also recommend reading https://aaronparecki.com/oauth-2-simplified and having a look at the documentation on https://oauth.net/2/

## Open ID Connect
[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) is a simple identity layer on top of the OAuth 2.0 protocol.
It enables Clients to verify the identity of the End-User based on the authentication performed by an Authorization Server, 
as well as to obtain basic profile information about the End-User in a REST-like manner.
Some good sources to get you started are:   
https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1  
https://connect2id.com/learn/openid-connect 

### Supported OpenID Connect Flows

#### Authorization Code Grant
The authorization code grant type is used to obtain both access tokens and refresh tokens and is optimized for 
confidential clients. Since this is a redirection-based flow, the client must be capable of interacting with the 
resource owner's user-agent (typically a web browser) and capable of receiving incoming requests (via redirection) 
from the authorization server.

For more information see [RFC-6749 section 4.1](https://tools.ietf.org/html/rfc6749#section-4.1)

## Tokens

### ID Token
The ID token is a signed information object representing the authenticated identity of the user.   
As part of the OpenID Connect standard, the ID token is encoded as a JWT and signed using the JWS standard.
The ID Token can be decoded for debugging purposes by tools such as [jwt.io](https://jwt.io/)

Example

Header
```json
{
  "alg": "RS256",
  "kid": "public:80f3c34a-9779-4e1e-b645-117f3b771af8",
  "typ": "JWT"
}
```
Body
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
See the Oidc-standard on [Id-token validation](https://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation) for the specifics. 
We recommend that you use a library for this. A good place to start is finding a library for your language at https://jwt.io/#libraries-io

### Access token
Access tokens are randoms strings that represents the authorization of a specific application to access specific parts of a user’s data.
The token itself does not provide any information, but it can be used to fetch the data that the end-user has consented to share from the [userinfo endpoint](#openid-connect-userinfo).  
Access tokens MUST be kept confidential in transit and storage. 

Example:
```
"hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus"
```
For more information see [RFC-6749 section 4.1.3-4.1.4](https://tools.ietf.org/html/rfc6749#section-4.1.3)

### Refresh token
Vipps Login does not currently support refresh tokens.

## Scopes
Scopes are space-separated lists of identifiers used to specify what access privileges are being requested.  
Vipps Login currently supports the following scopes:

| Scopes      | Description                                    | User consent required  |
| ------------| -----------------------------------------------|-------- |
| openid      | Scope used to request an Id-token              |   no    |
| address     | List containing the users addresses. Will always contain home, but can also include work and other.    |   yes   |
| birthDate   | User birth date                                |   yes   |
| email       | User email                                     |   yes   |
| name        | User first, middle and given name              |   yes   |
| phoneNumber | Verified phone number                          |   yes   |
| nnin        | Norwegian national identity number. NB, access to this scope needs to be pre-approved by Vipps |   yes      |


When requesting scopes that require user consent, a view listing these scopes will be displayed to the user with the option to
allow or deny the consent request. This view is skipped if no scopes requiring consent are requested.
The user can not make changes to the list of requested scopes, and can therefore not accept for example name and deny address.

We recommend asking for the minimal number of scopes needed for your usecase to minimize the number of users that deny the consent request.

# Integrating with Vipps Login
Vipps Login adheres to the OAuth2 and OpenIDConnect standards. The easiest way to integrate with the service is therefore to 
use a well renowned OAuth2.0/OpenID Connect Library for your programming language.
Vipps does not recommend a specific library, but the list of [OIDC Relying Party libraries](https://openid.net/developers/certified/) certified by the OpenID Foundation is a good starting point.

## Manual integration
This section contains information necessary to perform a manual integration with Vipps Login.  
This should not be attempted without a solid grasp of the OAuth2 and Open ID Connect standards.

### Base URLs

| Environment | Base URL |
|-------------|----------|
| Test        |https://apitest.vipps.no/access-management-1.0/access |
| Production  |https://api.vipps.no/access-management-1.0/access     |
    
### API endpoints

| Operation                 | Description         | Endpoint          |
| ------------------------- | ------------------- | ----------------- |
| [OAuth 2.0 Authorize](#oauth-20-authorize)           | Start an OAuth 2.0 authorization. | [`GET:/oauth2/auth`](https://vippsas.github.io/vipps-login-api/#/public/oauthAuth) |
| [OAuth 2.0 Token](#oauth-20-token)                   | Get an OAuth 2.0 access token. | [`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/public/oauth2Token) |
| [OpenID Connect Userinfo](#openid-connect-userinfo)   | Returns information that the user has consented to share. | [`GET:/userinfo`](https://vippsas.github.io/vipps-login-api/#/public/userinfo) |
| [OpenID Connect Discovery](#openid-connect-discovery) | Retrieve information for OpenID Connect clients. | [`GET:/.well-known/openid-configuration`](https://vippsas.github.io/vipps-login-api/#/public/discoverOpenIDConfiguration) |
| [JSON Web Keys Discovery](#json-web-keys-discovery)   | Get JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens. | [`GET:/.well-known/jwks.json`](https://vippsas.github.io/vipps-login-api/#/public/wellKnown) |

#### OAuth 2.0 Authorize

The authorize endpoint is a standard OIDC endpoint used for starting an authorization. The client creates an request 
URI and directs the resource owner to the constructed URI.

**Request**

The client constructs the request URI by adding the following parameters to the query component of the authorization 
    endpoint URI using the "application/x-www-form-urlencoded" format. The client directs the resource owner to the 
    constructed URI using an HTTP redirection response, or by other means available to it via the user-agent.
   
| Query             | Description                                                                                                                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| response_type     | Value MUST be set to "code".                                                                                                                                                              |
| client_id         | The client identifier, issued by Vipps.                                                                                                                                                   |
| redirect_uri      | Redirect URL which the user agent is redirected to after finishing a login. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-from-the-merchant)          |
| scope             | Scope of the access request, space-separated list.                                                                                                                                        |
| state             | An opaque value used by the client to maintain state between the request and callback. The authorization server includes this value when redirecting the user-agent back to the client.   |

For example, the client directs the user-agent to make the following HTTP request:

[`GET:/oauth2/auth?client_id={client_id}&response_type=code&scope={scopes}&state={state}&redirect_uri={redirect_uri}`](https://vippsas.github.io/vipps-login-api/#/public/oauthAuth)

**Response**

If the resource owner grants the access request, the authorization server issues an authorization code and delivers it 
    to the client by adding the following parameters to the query component of the redirection URI using the 
    "application/x-www-form-urlencoded" format.
    
| Query             | Description                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| response_type     | The authorization code generated by the authorization server. The client MUST NOT use the authorization code more than once. The authorization code is bound to the client identifier and redirection URI.    |
| state             | The exact value received from the client during the authorization request.                                                                                                                                    |

For example, the authorization server redirects the user-agent by
   sending the following HTTP response:

     HTTP/1.1 302 Found
     Location: https://client.example.com/callback?code={code}&state={state}
     
If the resource owner declines the access request or an error occurs, the authorization server following parameters to the query component of the redirection URI using the 
                                                                         "application/x-www-form-urlencoded" format.

See error handling for more information 
For more information see [RFC-6749 section 4.1.1-4.1.2](https://tools.ietf.org/html/rfc6749#section-4.1.1)

#### OAuth 2.0 Token
The token endpoint is a standard OIDC endpoint used for requesting Access and ID Tokens.
The client constructs the request by adding the parameters described below to the HTTP body by using the "application/x-www-form-urlencoded" format.

**Request**

*Headers*

| Header            | Description                            |
| ----------------- | -------------------------------------  |
| Content-Type      | "application/x-www-form-urlencoded"    |                                                                                                                                                 
| Authorization     | "Basic {Client Credentials}"           |                                                                                                                                   

The Client Credentials is a base 64 encoded string consisting of the Client id and secret issued by Vipps joined by ":"

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
| redirect_uri      | Redirect URL the user agent is redirected to after finishing a login. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-from-the-merchant). |                                                                                                                                    |

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


#### OpenID Connect Userinfo
This endpoint returns the payload of the ID Token, including the idTokenExtra values, of the provided OAuth 2.0 access token. 
You can learn more at the [OIDC Standard](http://openid.net/specs/openid-connect-core-1_0.html#UserInfo)

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

Example respose:
```json


{
  "sub" : "c06c4afe-d9e1-4c5d-939a-177d752a0944",
 "address" : [ {
     "country" : "NO",
     "street_address" : "Testgata 23",
     "address_type" : "home",
     "formatted" : "Testgata 23\n01234\nOslo\nNO",
     "postal_code" : "01234",
     "Region" : "Oslo"
   }, {
     "country" : "NO",
     "street_address" : "Robert Levins Gate 5",
     "address_type" : "work",
     "formatted" : "Robert Levins Gate 5\n0154\nOslo\nNO",
     "postal_code" : "0154",
     "region" : "Oslo"
   }, {
     "country" : "NO",
     "street_address" : "Fjellhyttevegen 2",
     "address_type" : "other",
     "formatted" : "Fjellhyttevegen 2 2\n5700\nVoss\nNO",
     "postal_code" : "5700",
     "Region" : "Voss"
   }],
  "name" : "Ada Lovelace",
  "given_name" : "Ada",
  "family_name" : "lovelace",
  "email" : "ada@exampleEmail.com",
  "sid" : "ec41669d-be3f-4a43-8556-c779c5676931"
}
```


#### OpenID Connect Discovery
The OIDC connect discovery endpoint can be used to retrieve configuration information for OpenID Connect clients.  
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
  "userinfo_endpoint": "https://apitest.vipps.no/access-management-1.0/access/userinfo",
  "scopes_supported": [
    "offline",
    "openid",
    "address",
    "name",
    "email",
    "phoneNumber",
    "nnin",
    "birthDate"
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
#### JSON Web Keys Discovery
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

## Automatic Recognition
Vipps Login will offer the option to authenticate end-users automatically when they return to a merchant's site. 
This gives merchants the opportunity to provide custom content to the user directly without the need for manual authentication. 
A precondition is that the user has consented to use Vipps login at the merchant's site, and has opted into being remembered in the browser being used.  
This feature is still under development. 

## API endpoints required from the merchant
The following endpoints are to be implemented by merchants, in order for Vipps to redirect the resource owner to them.

### Receive authentication result
After a successful authentication, the user agent is redirected to this endpoint with the following parameters added to the query component.  
This URI needs to be pre-registered with Vipps and supplied as a query parameter on calls to the [OAuth2 authorize endpoint](#OAuth-2.0-Authorize)  

| Param             | Description                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| code              | The authorization code generated by the authorization server. The client MUST NOT use the authorization code more than once. The authorization code is bound to the client identifier and redirection URI.    |
| state             | The exact value received from the client during the authorization request.                                                                                                                                    |

Example: 

     HTTP/1.1 302 Found
     Location: https://client.example.com/callback?code={code}&state={state}

### Error handling 
If the user cancels the login or an error occurs, the user agent is redirected to the receive authentication result endpoint with 
the following parameters added to the query component.

| Param             | Description                                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| error             | Standard [OAuth2](https://tools.ietf.org/html/rfc6749#section-4.1.2.1) or [OIDC](https://openid.net/specs/openid-connect-core-1_0.html#AuthError) error code. |
| error_description | A short text providing additional information on the error that occurred.                                                                                     |
| state             | The exact value received from the client during the authorization request.                                                                                    | 

Example: 

     HTTP/1.1 302 Found
     Location: https://client.example.com/callback?error=access_denied&error_description=user%20cancelled%20the%20login?state={state}

If a fatal error occurs where the user can not be redirected back to the merchant, a generic Vipps styled error page will be shown containing a brief error description.
