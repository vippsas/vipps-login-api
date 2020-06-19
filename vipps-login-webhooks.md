# Webhooks

## Table of contents
* [Consent](#consent-webhooks)
    * [Revoke](#revoke)
 
## Consent Webhooks

### Revoke
Users can revoke their consent to share data with merchants. This is done in the profile section -> personal information in the app. If a user removes the consent to share data with a merchant, the practical consequences is that the user needs to give a consent the next time he/she would like to login with Vipps or share data as part of other Vipps services with the merchant.
 
When a user revokes their consent, Vipps will send an `CONSENT_REVOKED` event containing the unique identifier (sub) for the given user to the webhook provided by the merchant.

This service is optional for the merchant and can be used to trigger events on the merchant's side. I.e. the merchant can inform the user that they still have an account, can provide information on alternative login solutions or where the user should go if they would like to delete the data they have stored with the merchant. 
  
  
**Content**  
The webhook is sent as a `POST` with a `text/plain` body containing an unsigned Jason Web Token ([JWT](https://jwt.io/)).  
The JWT format was choosen to allow for the possibility to add signing on a later state, but as of now the JWT  is delivered unsigned with the algorithm set to none and is therefore no more secure than a regular json and should be handled as such.

**Example request:**
```text
eyJhbGciOiJub25lIiwidHlwIjogIkpXVCJ9Cg.eyJleHAiOjE1OTI1NzE3ODgsImlhdCI6MTU5MjU3MTQ4OCwibmJmIjoxNTkyNTcxNDg4LCJzdWIiOiJjOWQxMDQ0NC1kOTkyLTQ4NTAtYWM2MC05ZDM1MDIwOTUwMDgiLCJldmVudCI6IkNPTlNFTlRfUkVWT0tFRCJ9
```

*Decoded JWT*
```
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

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
