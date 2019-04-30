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
    * [Exception handling](#exception-handling)
    * [Error groups](#error-groups)
    * [Error codes](#error-codes)
* [API endpoints required by Vipps from the merchant](#api-endpoints-required-by-vipps-from-the-merchant)
    * [Callback endpoints](#callback-endpoints)
    * [Callback](#callback)
    

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
The authorization code grant type is used to obtain both access tokens and refresh tokens and is optimized for 
    confidential clients. Since this is a redirection-based flow, the client must be capable of interacting with the 
    resource owner's user-agent (typically a web browser) and capable of receiving incoming requests (via redirection) 
    from the authorization server.
TODO: Add a better flow chart

     +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)

Note: The lines illustrating steps (A), (B), and (C) are broken into two parts as they pass through the user-agent.
   
    The flow illustrated in the figure includes the following steps:
    
      (A)  The client initiates the flow by directing the resource owner's
           user-agent to the authorization endpoint. The client includes
           its client identifier, requested scope, local state, and a
           redirection URI to which the authorization server will send the
           user-agent back once access is granted (or denied).
    
      (B)  The authorization server authenticates the resource owner (via
           the user-agent) and establishes whether the resource owner
           grants or denies the client's access request.
    
      (C)  Assuming the resource owner grants access, the authorization
           server redirects the user-agent back to the client using the
           redirection URI provided earlier (in the request or during
           client registration). The redirection URI includes an
           authorization code and any local state provided by the client
           earlier.
    
      (D)  The client requests an access token from the authorization
           server's token endpoint by including the authorization code
           received in the previous step. When making the request, the
           client authenticates with the authorization server.  The client
           includes the redirection URI used to obtain the authorization
           code for verification.
    
      (E)  The authorization server authenticates the client, validates the
           authorization code, and ensures that the redirection URI
           received matches the URI used to redirect the client in
           step (C).  If valid, the authorization server responds back with
           an access token and, optionally, a refresh token.

For more information see [RFC-6749 section 4.1](https://tools.ietf.org/html/rfc6749#section-4.1)

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

## OpenID Connect Discovery
The well known endpoint an be used to retrieve configuration information for OpenID Connect clients.  
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-discovery-1_0.html)

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

```json
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
| redirect_uri      | Redirect url the useragent is redirected to after finishing a login. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-by-vipps-from-the-merchant)          |
| scope             | Scope of the access request, space separated list.                                                                                                                                        |
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

For more information see [RFC-6749 section 4.1.1-4.1.2](https://tools.ietf.org/html/rfc6749#section-4.1.1)

## OAuth 2.0 Token
The token endpoint is a an OIDC standard endpoint used for requesting various combination of ID, Access and Refresh tokens.
The type of request (and corresponding response) is determined by the grant_type.

**Request**

[`POST:/oauth2/token`](https://vippsas.github.io/vipps-login-api/#/public/oauth2Token)

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `401 Unauthorized`      | Invalid credentials.                                    |
| `500 Server Error`      | An internal Vipps problem.                              |


```json
{
  "access_token": "hel39XaKjGH5tkCvIENGPNbsSHz1DLKluOat4qP-A4.WyV61hCK1E2snVs1aOvjOWZOXOayZad0K-Qfo3lLzus",
  "expires_in": 3600,
  "id_token": 0,
  "refresh_token": "string",
  "scope": "openid",
  "token_type": "bearer"
}
```
**Response**

For more information see [RFC-6749 section 4.1.3-4.1.4](https://tools.ietf.org/html/rfc6749#section-4.1.3)

## OpenID Connect Userinfo
This endpoint returns the payload of the ID Token, including the idTokenExtra values, of the provided OAuth 2.0 access token. 
You can learn more at the [OIDC Standard](http://openid.net/specs/openid-connect-core-1_0.html#UserInfo)


**Request**

[`GET:/userinfo`](https://vippsas.github.io/vipps-login-api/#/public/userinfo)

**Response**

Overview

| HTTP status             | Description                                             |
| ----------------------- | ------------------------------------------------------- |
| `200 OK`                | Request successful.                                     |
| `401 Unauthorized`      | Invalid credentials.                                    |
| `500 Server Error`      | An internal Vipps problem.                              |

Examples
```json
{
  "birthdate": "string",
  "email": "string",
  "email_verified": true,
  "family_name": "string",
  "gender": "string",
  "given_name": "string",
  "locale": "string",
  "middle_name": "string",
  "name": "string",
  "phone_number": "string",
  "profile": "string",
  "sub": "string",
  "updated_at": 0
}
```

## Exception handling

TODO: Fill out

## Error groups
## Error codes

# API endpoints required by Vipps from the merchant
## Callback endpoints
## Callback
