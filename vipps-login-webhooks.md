# Webhooks

## Table of contents
* [Consent](#consent-webhooks)
    * [Revoke](#revoke)
 
 ## Consent Webhooks

 ### Revoke
 When a user revokes their consent, Vipps will issue an `CONSENT_REVOKED` event containing the unique identifier (sub) for the given user.
  
  **Content**  
 The webhook is sent as a `POST` with a `text/plain` body containing a Jason Web Token (JWT).  
 
 The JWT format was chosen to have the opportunity to add signing at a later stage if deemed necessary.
 As of now the jwt is delivered with the algorithm set to none and is therefore no more secure than a regular json and should be handled as such.

**Example request:**
```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTI1NDg3NTIsInN1YiI6ImM5ZDEwNDQ0LWQ5OTItNDg1MC1hYzYwLTlkMzUwMjA5NTAwOCIsImV2ZW50IjoiQ09OU0VOVF9SRVZPS0VEIn0.EZNPsigVS5IWi0NvRgQaOcnvuJ9APgi7q4mePIJaNJA
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
  "exp": 1592548752,
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
