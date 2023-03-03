<!-- START_METADATA
---
title: API guide
sidebar_position: 30
---
END_METADATA -->

# Integrating with Vipps Login

API version: 2.0

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

Vipps Login mostly adheres to the OAuth2 and OpenID Connect standards. The easiest - and **strongly recommended - way
to integrate with the service is therefore to use a well renowned
OAuth2.0/OpenID Connect Library for your programming language**.
Vipps does not recommend a specific library, but the list of
[OIDC Relying Party libraries](https://openid.net/developers/certified/)
certified by the OpenID Foundation is a good starting point.

## manual integration

This section contains information necessary to perform a manual integration with
Vipps Login. This **should not be attempted without a solid grasp of the OAuth2
and Open ID Connect standards**. All endpoints needed for integration can be found in our openid connect discovery endpoint.
These endpoints should be fetched dynamically by your application, since they are prone for change.

### Openid connect discovery endpoint

| Environment | Base URL                                                                                 |
|-------------|------------------------------------------------------------------------------------------|
| Test        | <https://apitest.vipps.no/access-management-1.0/access/.well-known/openid-configuration> |
| Production  | <https://api.vipps.no/access-management-1.0/access/.well-known/openid-configuration>     |

The OpenID connect discovery endpoint can be used to retrieve configuration information for openid connect clients. We recommend to fetch these dynamically, however the response from this endpoint rarely changes. Therefore it can and should be cached so it's not fetched over the network on every login. The endpoint responds with a `Cache-Control: max-age=3600` header.

You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-discovery-1_0.html).

**Request**

[`GET:/.well-known/openid-configuration`][login-discoveropenid-endpoint]

**Response**

Overview

| HTTP status        | Description                                               |
|--------------------|-----------------------------------------------------------|
| `200 OK`           | Request successful.                                       |
| `500 Server Error` | An internal Vipps problem.                                |

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

### Openid connect discovery URLs

| Operation                                           | Description                                                                         | Endpoints                                                |
|-----------------------------------------------------|-------------------------------------------------------------------------------------|----------------------------------------------------------|
| [OAuth 2.0 Authorize](#oauth-20-authorize)          | Start an OAuth 2.0 authorization.                                                   | [`GET:/oauth2/auth`][access-auth-endpoint]               |
| [OAuth 2.0 Token](#oauth-20-token)                  | Get an OAuth 2.0 access token.                                                      | [`POST:/oauth2/token`][access-token-endpoint]            |
| [Userinfo](#userinfo)                               | Returns information that the user has consented to share.                           | [`GET:/vipps-userinfo-api/userinfo`][userinfo-endpoint]  |
| [JSON Web Keys Discovery](#json-web-keys-discovery) | Get JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens. | [`GET:/.well-known/jwks.json`][login-wellknown-endpoint] |

### OAuth 2.0 Authorize

The authorize endpoint is a standard OIDC endpoint used for starting an
authorization. The client creates an request URI and directs the resource owner
to the constructed URI.

**Request**

The client constructs the request URI by adding the following parameters to the
query component of the authorization endpoint URI using the
`application/x-www-form-urlencoded` format. The client directs the resource
owner to the constructed URI using an HTTP redirection response, or by other
means available to it via the user-agent.

| Query                 | Description                                                                                                                                                                                                                                                                                                                                                                                 |
|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| response_type         | Value MUST be set to "code".                                                                                                                                                                                                                                                                                                                                                                |
| client_id             | The client identifier, issued by Vipps.                                                                                                                                                                                                                                                                                                                                                     |
| redirect_uri          | Redirect URL which the user agent is redirected to after finishing a login. If the URL is using a custom URL scheme, such as `myapp://`, a path is required: `myapp://path-to-something`. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-from-the-merchant)                                                                                                |
| scope                 | Scope of the access request, space-separated list.                                                                                                                                                                                                                                                                                                                                          |
| state                 | An opaque value (e.g. a GUID) used by the client to maintain state between the request and callback. The authorization server includes this value when redirecting the user-agent back to the client. It must be at least 8 characters long to ensure sufficient entropy. In case of a too short state parameter the end-user will be redirected back to the merchant's site with an error. |
| requested_flow        | Optional. Request a specific flow for the user. See [App integration](#app-integration), [Automatic return from Vipps app](#automatic-return-from-vipps-app) and [No dialog flow](#no-dialog-flow)                                                                                                                                                                                          |
| app_callback_uri      | Optional. The target uri for automatic switch back to merchant app. Requires `requested_flow=app_to_app`. Example `merchant-app://callback`                                                                                                                                                                                                                                                 |
| final_redirect_is_app | Optional. Either `true` or `false`. If this is `true` we will enable some compatibility features to make sure the user is returned to the app.                                                                                                                                                                                                                                              |
| code_challenge_method | Optional. Used for [PKCE](https://datatracker.ietf.org/doc/html/rfc7636), either `S256` or `plain`. Default value is `plain`                                                                                                                                                                                                                                                                |
| code_challenge        | Optional. Used for [PKCE](https://datatracker.ietf.org/doc/html/rfc7636). The value must be calculated based on the `code_verifier` later used towards the [token endpoint](#oauth-20-token)                                                                                                                                                                                                |

For example, the client directs the user-agent to make the following HTTP request:

[`GET:/oauth2/auth?client_id={client_id}&response_type=code&scope={scopes}&state={state}&redirect_uri={redirect_uri}`][access-auth-endpoint]

You can test this by entering the url into any browser. This will initiate the log in sequence.

For more information about testing this with the Postman collection, see the step-by-step instructions in the [quick start guide](../vipps-login-api-quick-start.md).

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

| Query | Description                                                                                                                                                                                                                                                             |
|-------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code  | The authorization code generated by the authorization server. This code should be sent with the subsequent `/token` request.  The client MUST NOT use the authorization code more than once. The authorization code is bound to the client identifier and redirect URI. |
| state | The exact value received from the client during the authorization request.                                                                                                                                                                                              |
| scope | The scopes that the end user has consented to. This list will always be the same as in the request query as end users are not allowed to remove individual scopes when they give consent.                                                                               |

For example, the authorization server redirects the user-agent by sending the
following HTTP response:

```http
HTTP/1.1 302 Found
Location: https://example.com/callback?code={code}&state={state}&scope={scopes}
```

If the resource owner declines the access request, or an error occurs, the authorization server adds the following parameters to the query component of the
redirection URI using the `application/x-www-form-urlencoded` format. More details in [Error handling](#error-handling)

For more general information see the standard specifications
[OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#AuthResponse) and [RFC-6749 section 4.1.1-4.1.2](https://tools.ietf.org/html/rfc6749#section-4.1.1).

### OAuth 2.0 Token

The token endpoint is a standard OIDC endpoint used for requesting Access and
ID Tokens. The client constructs the request by adding the parameters described
below to the HTTP body by using the `application/x-www-form-urlencoded` format.

**Request**

_Headers_

| Header         | Description                         |
|----------------|-------------------------------------|
| Content-Type   | "application/x-www-form-urlencoded" |
| Authorization  | "Basic {Client Credentials}"        |

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

| Key          | Description                                                                                                                                                                                                                                                                                                                                                            |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| grant_type   | Value MUST be authorization_code.                                                                                                                                                                                                                                                                                                                                      |
| code         | The authorization code received as a query param on the redirect_uri from the authorization server.                                                                                                                                                                                                                                                                    |
| redirect_uri | Redirect URL which the user agent is redirected to after finishing a login. If the URL is using a custom URL scheme, such as `myapp://`, a path is required: `myapp://path-to-something`. See [API endpoints required by Vipps from the merchant](#api-endpoints-required-from-the-merchant) . This field is required for OIDC flows, i.e. regular Vipps Login logins. |

[`POST:/oauth2/token`][access-token-endpoint]

**Response**

| HTTP status         | Description                 |
|---------------------|-----------------------------|
| `200 OK`            | Request successful.         |
| `401 Unauthorized`  | Invalid credentials.        |
| `500 Server Error`  | An internal Vipps problem.  |

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

### Userinfo

This endpoint returns the payload with the information that the user has consented to share, which is provided in
the OAuth 2.0 access token.
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo).

**Request**

_Headers_

| Header         | Description              |
|----------------|--------------------------|
| Authorization  | "Bearer {Access Token}"  |

The access token is received on a successful request to the [token endpoint](#oauth-20-token).

[`GET:/vipps-userinfo-api/userinfo`][userinfo-endpoint]

**Response**

Overview

| HTTP status         | Description                 |
|---------------------|-----------------------------|
| `200 OK`            | Request successful.         |
| `401 Unauthorized`  | Invalid credentials.        |
| `500 Server Error`  | An internal Vipps problem.  |

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
    ]
}
```

### JSON Web Keys Discovery

This endpoint returns JSON Web Keys to be used as public keys for verifying OpenID Connect ID Tokens.

**Request**

[`GET:/.well-known/jwks.json`][login-wellknown-endpoint]

**Response**

Overview

| HTTP status         | Description                |
|---------------------|----------------------------|
| `200 OK`            | Request successful.        |
| `500 Server Error`  | An internal Vipps problem. |

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

### Receive authentication result

After a successful authentication, the user agent is redirected to this endpoint
with the following parameters added to the query component. This URI needs to
be pre-registered with Vipps and supplied as a query parameter on calls to the
[OAuth2 authorize endpoint](#oauth-20-authorize).

| Param   | Description                                                                                                                                                                                                |
|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code    | The authorization code generated by the authorization server. The client MUST NOT use the authorization code more than once. The authorization code is bound to the client identifier and redirection URI. |
| state   | The exact value received from the client during the authorization request.                                                                                                                                 |

Example:

```http
HTTP/1.1 302 Found
Location: https://example.com/callback?code={code}&state={state}
```

## Error handling

If the user cancels the login or an error occurs, the user agent is redirected
to the authentication result endpoint with the following parameters
added to the query component.

| Param              | Description                                                                                                                                                    |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| error              | Standard [OAuth2](https://tools.ietf.org/html/rfc6749#section-4.1.2.1) or [OIDC](https://openid.net/specs/openid-connect-core-1_0.html#AuthError) error code.  |
| error_description  | A short text providing additional information on the error that occurred.                                                                                      |
| state              | The exact value received from the client during the authorization request.                                                                                     |

### Custom error codes

In addition to the standard errors defined in [OAuth2](https://tools.ietf.org/html/rfc6749#section-4.1.2.1) and [OIDC](https://openid.net/specs/openid-connect-core-1_0.html#AuthError), Vipps Login also has some custom error codes that you might see on the redirect:

| error_code                         | Description                                                      |
|------------------------------------|------------------------------------------------------------------|
| `access_denied`                    | User cancelled the login                                         |
| `server_error`                     | Something went wrong, please try again                           |
| `login_required`                   | User must login with interaction                                 |
| `invalid_app_callback_uri`         | The app callback uri is not on a valid format                    |
| `app_callback_uri_not_registered`  | The app callback uri is not registered as a redirect uri         |
| `outdated_app_version`             | The user's Vipps app version is too old and needs to be updated  |
| `wrong_challenge`                  | The user selected the wrong challenge                            |
| `unknown_reject_reason`            | Something went wrong. Reject reason is unknown.                  |

There may be other errors, so integrators must be able to handle undocumented errors gracefully.

Example:

```http
HTTP/1.1 302 Found
Location: https://example.com/callback?error=access_denied&error_description=user%20cancelled%20the%20login?state={state}
```

If a fatal error occurs where the user can not be redirected back to the merchant, a generic Vipps styled error page will be shown containing a brief error description.

## Call by call

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

[login-wellknown-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/wellKnown
[login-discoveropenid-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/discoverOpenIDConfiguration
[userinfo-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Userinfo-API/operation/userinfoAuthorizationCode
[access-auth-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/oauthAuth
[access-token-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/login#tag/Vipps-Login-API/operation/oauth2Token
