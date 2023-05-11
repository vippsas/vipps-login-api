---
title: Important information
sidebar_label: Important information
sidebar_position: 20
description: Important information needed for Login
pagination_next: null
pagination_prev: null
---

# Important information

Here is some useful information which can help you as you integrate with Vipps Login.

## Design guidelines and buttons

Buttons to use for Vipps Login can be found as part of our
[design guidelines](https://developer.vippsmobilepay.com/docs/vipps-design-guidelines/).

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
both phone number and e-mail address are verified in Vipps.
Before linking an account based on e-mail, ensure that the flag `email_verified : true` in the response.
If this for some reason is "false", the matching should be aborted,
or the user should be prompted to log in to the original account
or confirm the account linking by having a confirmation link sent to the email address.

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

If the user does not have an existing account, but this is required for some reason
(e.g. because you have a separate sign-up process or the users need to be an existing member/customer),
you should provide the user with a message explaining the situation after Vipps Login has finished.
If relevant, you should provide links or directions on how to create the required account.

## Using Vipps Login in native applications

Web views should not be used when using Vipps Login in a native application. Instead, the user should be redirected using an external browser opened by or within the app.

Android: Use [Custom Tabs](https://developer.chrome.com/multidevice/android/customtabs) or fallback to open external browser on user's device.

iOS: Use [SFAuthenticationSession](https://developer.apple.com/documentation/safariservices/sfauthenticationsession) (for iOS 11 and 12) or
[ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession) (for iOS 13 and above).

## Revoke Consent Webhook

Users can revoke their consent to share data with merchants. This is done in the profile section -> personal information in the app.
If a user removes the consent to share data with a merchant, the practical consequences is that the user needs to give a consent the next time he/she would like to login with Vipps
or share data as part of other Vipps services with the merchant.

When a user revokes their consent, Vipps will send an `CONSENT_REVOKED` event containing the unique identifier `sub` for the given user to the webhook provided by the merchant.

This service is optional for the merchant and can be used to trigger events on the merchant's side.
For example, the merchant can inform the user that they still have an account, can provide information on alternative login solutions, or
can inform the user where to go if they would like to delete the data they have stored with the merchant.

**Content**  
The webhook is sent as a `POST` with a `text/plain` body containing an unsigned Json Web Token ([JWT](https://jwt.io/)).  
The JWT format was chosen to allow for the possibility to add signing on a later state, but as of now the JWT is delivered unsigned with the algorithm set to `none`.
It is therefore no more secure than a regular json and should be handled as such.

**Example request:**

```text
eyJhbGciOiJub25lIiwidHlwIjogIkpXVCJ9Cg.eyJleHAiOjE1OTI1NzE3ODgsImlhdCI6MTU5MjU3MTQ4OCwibmJmIjoxNTkyNTcxNDg4LCJzdWIiOiJjOWQxMDQ0NC1kOTkyLTQ4NTAtYWM2MC05ZDM1MDIwOTUwMDgiLCJldmVudCI6IkNPTlNFTlRfUkVWT0tFRCJ9
```

*Decoded JWT*

`Header`

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

`Payload`

```json
{
  "exp": 1592571788,
  "iat": 1592571488,
  "nbf": 1592571488,
  "sub": "c9d10444-d992-4850-ac60-9d3502095008",
  "event": "CONSENT_REVOKED"
}
```
