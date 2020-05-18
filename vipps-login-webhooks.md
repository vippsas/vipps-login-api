# Webhooks (Preview)

This feature is currently in preview and should only be used for testing.  
It is likely the implementation will change.

## Table of contents
* [Consent](#consent-webhooks)
    * [Revoke](#revoke)
 
 ## Consent Webhooks

 ### Revoke
Users can revoke their consent to share data with merchants. This is done in the profile section -> personal information in the app. If a user removes the consent to share data with a merchant, the practical consequences is that the user needs to give a consent the next time he/she would like to login with Vipps or share data as part of other Vipps services with the merchant.
 
When a user revokes their consent, Vipps will send an `CONSENT_REVOKED` event containing the unique identifier (sub) for the given user to the webhook provided by the merchant.

This service is optional for the merchant and can be used to trigger events on the merchant's side. I.e. the merchant can inform the user that they still have an account, can provide information on alternative login solutions or where the user should go if they would like to delete the data they have stored with the merchant. 
  
 **Verification**  
 Before acting on the received webhook the timestamp field should be checked to ensure that it is not older than five minutes.  
  
**Example response:**
```json
{
  "sub":"c9d10444-d992-4850-ac60-9d3502095008",
  "event":"CONSENT_REVOKED",
  "timestamp": 1584362177,
  "signature":"oADevM9M8z69LzNVeJQaAP3pKquHLgNuvivp+2QfzNZrgt8eMZLFnT7pZz1Sryi39ZqmhBJFE0+T+/hZ3lGvVPm9FP/KuXb22P62VymnuswUD6m8om5G0Vx/ijcLeW1j/czjbGklQuse95NH7POMmxK/40ah1SfX1+tS+HDEHzsivxP8P/6glzepFNS/nDjzPBxoD213TvgjE+QrdlRPbrrhZTG5KTWN5gpw5Fb7Q+NGhYUpo8flmSQezZVpl+4aWV6YdujUQdCqwDQjn6jDKQ9XkvzoNTlBGTo/cF7ywD34sN1jN9oQOu2hhbmPlq0KSqKbrkkrUUCuu8wvgg3fNg=="
}
```

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
