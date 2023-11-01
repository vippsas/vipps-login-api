<!-- START_METADATA
---
title: Partner keys
sidebar_label: Partner keys
sidebar_position: 33
description: Vipps Login offers partner keys which allow partners make API calls on behalf of their merchants.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Partner keys

In addition to the normal authentication, we offer
[Partner keys](https://developer.vippsmobilepay.com/docs/partner/partner-keys)
which allow partners make API calls on behalf of their merchants.


**Please note:** Partner API keys are only functional in the production environment, not the test environment.

Partner keys are available for the following flows:

* [Vipps Login in browser](overview.md#vipps-login-in-browser)
* [Vipps Login from phone number](./flows/merchant-initiated-login-integration.md)
  * [Regular](flows/merchant-initiated-login-integration.md#integrating-with-vipps-login-from-phone-number)
  * [Redirect to browser](flows/merchant-initiated-login-integration.md#redirect-to-browser)

If you are a Vipps MobilePay partner managing integrations on behalf of merchants, you can use your partner API credentials to authenticate, and then send the Merchant Serial Number (MSN) to identify which of your merchants you are acting on behalf of.

The main differences between partner key integration and merchant integration are:

* For merchant integration, authentication is based on the Vipps Login access token, described in the
  [Access token](core-concepts.md#access-token) section.
  But for partner key integration, a token is obtained from
  [`POST: /accesstoken/get`](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost).
  This is the same token endpoint used for eCom API payments. The token must be passed as a Bearer token
  in the header (e.g., `Authorization: Bearer <access-token>`). For partner key integration, this
  authorization method *must* be used. The `token_endpoint_authentication` methods,
  `client_secret_basic` and `client_secret_post`, do not work for partner key integrations.
* For partner key integration, the `Merchant-Serial-Number` header must be sent as part of
  backend requests to identify which merchant you're acting on behalf of.

Read more about partner verification in:

* [Partner keys](https://developer.vippsmobilepay.com/docs/partner/partner-keys)
* [Access token API guide](https://developer.vippsmobilepay.com/docs/APIs/access-token-api)

Browser and phone number flows have minor differences and are described in more detail further down.

## Partner key in the Vipps Login in browser flow

### Authorization endpoint

To initiate a login as partner, redirect the end-user's browser to a `/auth` URL containing the merchant serial number as a `msn` query parameter instead of `client_id`.

```http request
GET https://api.vipps.no/access-management-1.0/access/oauth2/auth?msn={Merchant-Serial-Number}&response_type=code&scope={scopes}&state={state}&redirect_uri={redirect_uri}
```

This request will return a redirect to a `/auth` URL where the `msn` query parameter is replaced by the merchant's `client_id`

### Token endpoint

For the `/token` endpoint, you should authenticate using an access token. Regular `client_secret_basic` or `client_secret_post` does not work for partner key integration.

1. Fetch [access_token](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost)
2. Use the access token in the `Authorization` header, `Bearer access-token`.
3. Add the `Merchant-Serial-Number` header with the target merchant serial number.

**Example request:**

```http request
POST https://api.vipps.no/access-management-1.0/access/oauth2/token
Authorization: Bearer <access-token>
Content-Type: application/x-www-form-urlencoded
Merchant-Serial-Number: 12345

grant_type=authorization_code&code={authorization_code}&redirect_uri={redirect_uri}
```

## Partner in Vipps Login from phone number flow

The `Merchant-Serial-Number` header must be sent in the header of all API requests.

1. Fetch [access_token](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost)
2. Use the access token in the `Authorization` header `Bearer access-token`
3. Add the `Merchant-Serial-Number` header with the target merchant serial number

### Example /backchannel/authentication request

```http request
POST https://api.vipps.no/vipps-login-ciba/api/backchannel/authentication
Authorization: Bearer <access-token>
Content-Type: application/x-www-form-urlencoded
Merchant-Serial-Number: 12345

scope=name address openid&login_hint=urn:msisdn:{msisdn}&state=13821s837213bng26e2n61gege26&nonce=21hebdhwqdb7261bd1b23
```

### Example /token request (without redirect)

For the subsequent token request you should authorize with the same access token and add the `Merchant-Serial-Number` header as in the `/backchannel/authentication` request.

```http request
POST https://apitest.vipps.no/access-management-1.0/access/oauth2/token
Authorization: Bearer <access-token>
Content-Type: application/x-www-form-urlencoded
Merchant-Serial-Number: 12345

auth_req_id=<auth-req-id>&grant_type=urn:openid:params:grant-type:ciba
```

## ID token validation

When acting as a partner on behalf of a merchant's sales unit, you do not have access to the merchant's `client_id`, which is used for [ID token validation](core-concepts.md#id-token).
We strongly encourage partners to use the `msn` claim in the ID token for this purpose and ensure that its value is the same as the Merchant serial number used for the login.

## Requirements

* The target client must be configured with `client_secret_basic` authentication.

## Merchant Serial Number (MSN)

This is a unique ID for the sales unit. This is a required parameter if you are a partner making API requests on behalf of a merchant. The partner must use the merchant's MSN, not the partner's MSN.

## Userinfo

For fetching userinfo, the token received during the login flow must be used.
