# Frequently Asked Questions for Vipps Login API

See the [Vipps Login API](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md) for all the details.

See also the
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide for the Vipps Developer Portal.

## Questions and answers

### Compliance  

**Q**: How can our system dynamically "know/find out" if the user has revoked their concent for us to have access to his personal data in our system?  
**A**: We are working on a system for notifying merchants when an end user revokes their consent. It will consist of an additional callback url that you register on the merchant portal which we will send a notification to when an end user removes their consent  

### Technical

**Q**: Can we have multiple URIs as landing pages?  
**A**: You can register as many callback urls as you want; and then you specify which one you use in the request to [/auth](https://vippsas.github.io/vipps-login-api/#/Vipps%20Log%20In%20API/oauthAuth)  

### UX

**Q**: Can we change the name that appears in customer's Vipps app under `Login and Access`?  
**A**: The name which is displayed in the app is the name of the Sale Unit. You can do it yourself in the merchant portal, [https://portal.vipps.no/](https://portal.vipps.no/). Press `rediger`/`edit` under `salgsstedsinfo`/`?` and change to the desired name.  
