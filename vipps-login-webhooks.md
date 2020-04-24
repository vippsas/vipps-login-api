# Webhooks (Preview)

This feature is currently in preview and should only be used for testing.  
It is likely that the implementation will change.

## Table of contents
* [Consent](#consent-webhooks)
    * [Revoke](#revoke)
 
 ## Consent Webhooks

 ### Revoke
 When a user revokes their consent, Vipps will send an `CONSENT_REVOKED` event containing the unique identifier (sub) for the given user.
 The merchant should then use the sub to look up the user and remove any data which has been received from Vipps logg inn.
 
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
