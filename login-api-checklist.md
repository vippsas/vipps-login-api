<!-- START_METADATA
---
title: Login API checklist
sidebar_label: Checklist
sidebar_position: 40
description: Checklist for full integration with the Login API.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Checklist

API version: 2.0.

## Endpoints to integrate

Integrate the [API endpoints](https://developer.vippsmobilepay.com/api/login/#tag/Login-API). For examples of requests and responses, see the [Postman collection](/tools/vipps-login-api.postman_collection.json) and [environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json).

| Endpoint | Comment |
|----------|---------|
| OpenID Connect (Get OIDC well-known endpoint) | [`GET:/access-management-1.0/access/.well-known/openid-configuration`](https://developer.vippsmobilepay.com/api/login/#tag/Login-API/operation/discoverOpenIDConfiguration) |
| The OAuth 2.0 authorize endpoint | [`GET:/access-management-1.0/access/oauth2/auth`](https://developer.vippsmobilepay.com/api/login/#tag/Login-API/operation/oauthAuth) |
| The OAuth 2.0 token endpoint | [`POST:/access-management-1.0/access/oauth2/token`](https://developer.vippsmobilepay.com/api/login/#tag/Login-API/operation/oauth2Token) |
| JSON Web Keys Discovery | [`GET:/access-management-1.0/access/.well-known/jwks.json`](https://developer.vippsmobilepay.com/api/login/#tag/Login-API/operation/wellKnown) |

## Integration considerations

| Action | Comment                                                                                                                                                                                                                                                                                                                                                                                                                       |
|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Use correct flow   | Native app integrations use the [app-to-app flow](api-guide/overview.md#app-to-app-flow). [Merchant initiated login](api-guide/overview.md#merchant-initiated-login) must *not* be used for web-based login.                                                                                                                                                                                                                  |
| Do not use embedded iFrames | Integration with the Login API is redirect-based (i.e., do not use an embedded iFrame).                                                                                                                                                                                                                                                                                                                                       |
|  Set company name and logo  | The sales unit name appears on the Vipps MobilePay landing page. Both the name and logo appear in the Vipps or MobilePay app under *Personal information* > *Companies with access*. See [how to change name and logo](login-api-faq.md#how-can-i-change-my-name-and-logo).                                                                                                                                             |
|   Whitelist redirect URIs   | Ensure that all `redirect_uri`s for your integration have been added to the merchant portal white list. See the [FAQ](login-api-faq.md#how-can-i-activate-and-set-up-vipps-login) for how to do this. Ensure that all URLs use `HTTPS` or native URL schemes (i.e., `vipps://`), not `HTTP`. ALL URLs must match the exact URI sent on `/auth` request. No query parameters or additional trailing `/` can be included. |
|  Use only required scopes | Request only the scopes that you require, no "nice to have" scopes. If using national identity number, you must be granted access to request these scopes. See [FAQ](login-api-faq.md#who-can-get-access-to-nin-and-how) for details.                                                                                                                                                                                   |
|  Comply with our terms and conditions   | If you, as a merchant, will act on behalf of others (share data you have gotten from us with other merchants), ensure that you comply with our terms and conditions to do this. This is shown in section 8.4 in our [terms and conditions for merchants](api-guide/overview.md#vipps-login-from-phone-number).                                                                                                                |
|    Present terms and conditions | Terms and conditions are presented to the user, and the necessary consents are collected from the user (i.e., consent to marketing purposes, etc.).                                                                                                                                                                                                                                                                           |

## Security

| Action | Comment |
|--------|---------|
| Generate a `state` parameter    | Ensure that a unique [`state` parameter](login-api-faq.md#whats-the-purpose-of-the-state-parameter) is generated for each `/auth` request. |
| Do not share `client_secret`   | Ensure that the `client_secret` is kept secret and is never be shared to browsers or native apps. |

## Quality assurance

| Action | Comment |
|--------|---------|
| Ensure functionality | Ensure that your solution is verified to work if the user start in a "non-default" browser on mobile, e.g. start Vipps Login from Chrome browser on iOS. |
| Update user registry | Implement proper linking of the Vipps or MobilePay user to your own user registry. This login must be based on either phone number or e-mail address. See [recommendations on linking to user account](api-guide/important-information.md#Recommendations-on-linking-to-user-account). |
| Handle errors | Make sure to log and handle all errors. For example, handle cancelled logins and error situations while redirecting the user back to `redirect_uri` (i.e., redirect with an `error` query parameter). In addition, display errors in a way that the users (customers and merchant employees/administrators) can see and understand them. |
| Include standard HTTP headers | Send the [HTTP headers](https://developer.vippsmobilepay.com/docs/knowledge-base/http-headers) in all API requests for better tracking and troubleshooting (mandatory for partners and platforms, who must send these headers as part of the checklist approval). |

## Avoid integration pitfalls

| Action | Comment   |
|--------|-----------|
| Follow design guidelines| The branding must be according to the [Design guidelines](https://developer.vippsmobilepay.com/docs/design-guidelines).|
| Educate customer support| Make sure your customer service, etc. has all the tools and information they need available in *your* system, through the APIs listed in the first item in this checklist, and that they do not need to visit [portal.vipps.no](https://portal.vipps.no) for normal work.|
