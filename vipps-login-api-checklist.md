# Vipps Login API Checklist

Checklist for integrating Vipps Login in browser:
- [ ] Integration with Vipps Login is redirect based (i.e no embedded iframe)
- [ ] Native app integrations use the [app-to-app flow](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#app-to-app-flow)
- [ ] CIBA flows should _not_ be used for web based login
- [ ] You use Vipps-buttons and ensure that Vipps branding on your website is according to the
      [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines).
- [ ] You present your terms and conditions to the user, and collect necessary consents from the user (i.e consent to marketing purposes, etc)  
- [ ] Your sale unit name (appears on the Vipps Login landing page) corresponds to the name of your company on your website. See [how to change name](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-change-my-name-and-logo)
- [ ] You should add your company logo to be shown on the list which the user has shared information with. See [how to change logo](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-change-my-name-and-logo)
- [ ] Whitelist redirect URIs:
    - [ ] You've added all `redirect_uri`s your integration is using to the merchant portal white list, see [FAQ](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login) for how to do this
    - [ ] Are not HTTP. Either HTTPS or app deeplinks
    - [ ] Matches the exact URI sent on `/auth` request (no query parameters, no additional trailing `/`)
- [ ] You're not requesting more scopes than is actually needed. 
- [ ] If you need access to national identity number and/or account numbers, you have gotten access to request these scopes. [Apply here](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nin-and-how)
- [ ] You generate a unique `state` [parameter](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#whats-the-purpose-of-the-state-parameter) for each `/auth` request. 
- [ ] `client_secret` should never be shared to browser or native app 
- [ ] You handle cancelled logins or error situations on the redirect back to `redirect_uri` (redirects with a `error` query parameter)  
- [ ] You've implemented proper linking of Vipps Login user to your own user registry. This login should be based on either phone number or e-mail address. See [recommendations on linking to user account](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#Recommendations-on-linking-to-user-account)
- [ ] Your API requests should be sending `Vipps-System-*` headers. These headers are required for plugin integrators, but recommended for all integrations 
  - [ ] `Vipps-System-Name`
  - [ ] `Vipps-System-Version`
  - [ ] `Vipps-System-Plugin-Name`
  - [ ] `Vipps-System-Plugin-Version`
