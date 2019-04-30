# Vipps Login API
API version: 0.9

Document version 0.9

API details: [Swagger UI](https://vippsas.github.io/vipps-login-api/#/),
[swagger.yaml](https://raw.githubusercontent.com/vippsas/vipps-login-api/master/docs/swagger.yaml),
[swagger.json](https://raw.githubusercontent.com/vippsas/vipps-login-api/master/docs/swagger.json).

# Table of contents

* [Overview](#overview)
  * [Login](#login)
  * [Getting Started](#getting-started)
  * [Supported OpenId Connect Flows](#supported-openid-connect-flows)
    * [Authorization Code Grant](#authorization-code-grant)
* [API endpoints](#api-endpoints)
    * [JSON Web Keys Discovery](#json-web-keys-discovery)
    * [OpenID Connect Discovery](#openid-connect-discovery)
    * [OAuth 2.0 Authorize](#oauth-20-authorize)
    * [OAuth 2.0 Token](#oauth-20-token)
    * [OpenID Connect Userinfo](#openid-connect-userinfo)

# Overview
The Vipps Login API offers functionality for authenticating and authorizing end users founded on the OAuth2 and OpenId connect
specifications. It supports using web browsers on websites and in native apps for iOS and Android using app switching. 

## Login
This service is currently in a pre release version. This documentation will be expanded, and is likely to undergo changes 
as we move closer to a public release.  

## Getting Started

* The first step towards using Vipps Login is creating a client.  

TODO: Hvordan skal et brukersted få opprettet en client? Skal vi skrive en mailadresse her i første omgang?

## Supported OpenId Connect Flows
### Authorization Code Grant


# Api endpoints
This section contains complete HTTP `requests` and `responses` for each API endpoint

| Operation                 | Description         | Endpoint          |
| ------------------------- | ------------------- | ----------------- |
| [JSON Web Keys Discovery](#json-web-keys-discovery)   | Get JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens. | [`GET:/.well-known/jwks.json`](https://vippsas.github.io/vipps-login-api/#/public/wellKnown) |
| [OpenID Connect Discovery](#openid-connect-discovery) | Retrieve information for OpenID Connect clients. | [`GET:/.well-known/openid-configuration`](https://vippsas.github.io/vipps-login-api/#/public/discoverOpenIDConfiguration) |
| [OAuth 2.0 Authorize](#OAuth-2.0-Authorize)           | Start an Oauth 2.0 authorization. | [`GET:/oauth2/auth`](https://vippsas.github.io/vipps-login-api/#/public/oauthAuth) |
| [OAuth 2.0 Token](#OAuth-2.0-Token)                   | Get an Oauth 2.0 access token. | [`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/public/oauth2Token) |
| [OpenID Connect Userinfo](#OpenID-Connect-Userinfo)   | Returns information the user have consented to give. | [`GET:/userinfo`](https://vippsas.github.io/vipps-login-api/#/public/userinfo) |

## JSON Web Keys Discovery
This endpoint returns JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens and, if enabled, 
OAuth 2.0 JWT Access Tokens.
**Request**

[`GET:/.well-known/jwks.json`](https://vippsas.github.io/vipps-login-api/#/public/wellKnown)

**Response**

Overview

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `500 Server Error`      | An internal Vipps problem.                              |

Examples

200 response

```
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

## OpenID Connect Discovery
TODO: Add description

**Request**

[`GET:/.well-known/openid-configuration`](https://vippsas.github.io/vipps-login-api/#/public/discoverOpenIDConfiguration)

**Response**

Overview

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `401 Unauthorized`      | Invalid credentials.                                    |
| `500 Server Error`      | An internal Vipps problem.                              |

Examples

200 response

```
{
    "issuer": "https://apitest.vipps.no/access-management-1.0/access/",
    "authorization_endpoint": "https://apitest.vipps.no/access-management-1.0/access/oauth2/auth",
    "token_endpoint": "https://apitest.vipps.no/access-management-1.0/access/oauth2/token",
    "jwks_uri": "https://apitest.vipps.no/access-management-1.0/access/.well-known/jwks.json",
    "subject_types_supported": [
        "string"
    ],
    "response_types_supported": [
        "string"
    ],
    "claims_supported": [
        "string"
    ],
    "grant_types_supported": [
        "string"
    ],
    "response_modes_supported": [
        "string"
    ],
    "userinfo_endpoint": "https://apitest.vipps.no/access-management-1.0/access/userinfo",
    "scopes_supported": [
        "string"
    ],
    "token_endpoint_auth_methods_supported": [
        "string"
    ],
    "userinfo_signing_alg_values_supported": [
        "string"
    ],
    "id_token_signing_alg_values_supported": [
        "string"
    ],
    "request_parameter_supported": true,
    "request_uri_parameter_supported": true,
    "require_request_uri_registration": true,
    "claims_parameter_supported": false
}
```

## OAuth 2.0 Authorize
TODO: Add description

**Request**

[`GET:/oauth2/auth`](https://vippsas.github.io/vipps-login-api/#/public/oauthAuth)

**Response**

## OAuth 2.0 Token
TODO: Add description

**Request**

[`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/public/oauth2Token)

**Response**

## OpenID Connect Userinfo
TODO: Add description

**Request**

[`GET:/userinfo`](https://vippsas.github.io/vipps-login-api/#/public/userinfo)

**Response**

# HTTP response codes
TODO: Kanskje flytte desse til hver enkel request over?
This API returns the following HTTP statuses in the responses:

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `302 Found`             | Request successful, client is redirected.               |
| `401 Unauthorized`      | Invalid credentials.                                    |
| `500 Server Error`      | An internal Vipps problem.                              |

## Exception handling

TODO: Fill out

## Error groups
## Error codes

# API endpoints required by Vipps from the merchant
## Callback endpoints
## Callback
