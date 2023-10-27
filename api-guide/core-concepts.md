<!-- START_METADATA
---
title: Core concepts
sidebar_label: Core concepts
sidebar_position: 30
description: Core concepts related to OAuth2 and OpenID Connect standards as used with Vipps login.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Core concepts

API version: 2.0

Vipps Login is to a large degree based on the OAuth2 and OpenID Connect standards. Some core concepts related to these are presented below.

## OAuth 2.0

[OAuth 2.0](https://tools.ietf.org/html/rfc6749) is the industry-standard
protocol for authorization. Giving a proper introduction to the standard is
out of the scope of this documentation, but there are many excellent resources
on the web. If you are new to the subject we recommend this
[talk](https://www.youtube.com/watch?v=996OiexHze0 ) by Nate Barbettini at Okta.
We also recommend reading
[OAuth 2 Simplified](https://aaronparecki.com/oauth-2-simplified) and having a
look at [the documentation](https://oauth.net/2/).

## Open ID Connect

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) is a
simple identity layer on top of the OAuth 2.0 protocol. It enables Clients to
verify the identity of the End-User based on the authentication performed by an
Authorization Server, as well as to obtain basic profile information about the
End-User in a REST-like manner.

Some good sources to get you started are:
[Identity, Claims, & Tokens – An OpenID Connect Primer](https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1)
and
[OpenID Connect explained](https://connect2id.com/learn/openid-connect).

## Supported OpenID Connect Flows

### Authorization Code Grant

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
  "msn": "123456",
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

These access tokens are provided by the
[access-management-1.0/access/oauth2/token][access-token-endpoint]
endpoint.
The token itself does not provide any information, but it can be used to
fetch the data that the end-user has consented to share from the
[userinfo endpoint](browser-flow-integration.md#userinfo).
Access tokens *must* be kept confidential in transit and storage.

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

It is possible to change the authentication method to `client_secret_post` in the merchant portal. This setting will then apply to all login transactions on this sales unit.
Go to [How can I use client_secret_post for authentication?](../vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication) for more information on how to change the authentication method.

For more information on the token endpoint see [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#TokenEndpoint) and [RFC-6749 section 3.2](https://tools.ietf.org/html/rfc6749#section-3.2).

## Scopes

Scopes are space-separated lists of identifiers used to specify what access privileges are being requested.
Vipps Login currently supports the following scopes:

| Scopes | Description | User consent required |
|--------|-------------|-----------------------|
| openid | This is the scope used to request an Id-token. It provides the claim `sub` which is a unique ID for the end user at that particular merchant. Note: Different merchants will get different `sub`s for the same end user. | no |
| address | The user can have up to three addresses in their Vipps or MobilePay app: home, work and other. User addresses are given as claims `address` and `other_addresses`. The claim `address` returns the address set as default for the Vipps or MobilePay user. The claim `other_addresses` returns any other addresses of the end user.  We recommend that merchants fetch all addresses on a user and allow the user to choose which address to use in the relevant context. Some users will not have any registered address, in these situations the claim `address` will be delivered, but the `sub` claims in address will be empty strings (e.g., `"address" : {"country" : "", "street_address" : "", "address_type" : "", "formatted" : "", "postal_code" : "", "region" : "" }`). If a user has information in the `Unit, floor or other details` field, this will be included in the `street_address` response. The `Street address` will then be presented first, before "\n". Then, the contents from `Unit, floor or other details` (e.g., `Suburbia 23"\nUnit B5`) | yes |
| birthDate | User birthdate (verified with National Population Register) | yes |
| email | User email (verified). The flag `email_verified : true` in the response can be used by merchants to confirm that the email actually is verified for each request. | yes |
| name | User first, middle, and given name (verified with National Population Register) | yes |
| phoneNumber | Verified phone number (verified - the number used with Vipps MobilePay) | yes |
| nin | Norwegian national identity number (NIN) (verified with National Population Register). Note, merchants need to apply for access to NIN. See [Who can get access to NIN and how?](../vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) for more information. | yes |
| delegatedConsents | Enabled consents to be collected on behalf of a merchant. This scope is only supported in *Vipps login from phone number*, see [Merchant delegated consents](flows/phone-number-ciba-flows.md#merchants-delegated-consents). | yes |

When requesting scopes that require user consent, a view listing these scopes
will be displayed to the user with the option to allow or deny the consent
request. This view is skipped if no scopes requiring consent are requested.
The user can not make changes to the list of requested scopes, and can
therefore not accept for example name and deny address.

We recommend asking for the minimal number of scopes needed for your use case to
minimize the number of users that deny the consent request.

[access-token-endpoint]: https://developer.vippsmobilepay.com/api/login/#tag/Login-API/operation/oauth2Token
