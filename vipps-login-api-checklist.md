# Vipps Login API Checklist

- [ ] The integration with Vipps Login should be redirect based (i.e no embedded iframe)
- [ ] Client secret should never be shared to client (not part of native app or sent in the browser)
- [ ] You've implemented proper linking of Vipps Login user to your own user registry. This login should be based on either phone number or e-mail address
- [ ] You collect necessary consents from the user after redirection back to your site (i.e consent to marketing purposes etc)  
- [ ] You generate a unique `state` parameter for each `/auth` request
- [ ] You handle cancelled logins or error situations on your redirect endpoint (redirects with a `error` query parameter)  
- [ ] Whitelisted redirect URIs:
    - [ ] Not http. Either https or app deep links
    - [ ] Matches the exact uri sent on `/auth` request (no query parameters, no additional trailing ``/)
    - [ ] You've added all `redirect_uri`s your integration is using to the merchant portal white list
- [ ] You're not requesting more scopes than is actually needed
- [ ] Native app integrations should use the app-to-app flow
- [ ] The Vipps branding on your website is according to the
      [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines).
- [ ] Your sale unit name (appears on the Vipps Login landing page) corresponds to the name of your company on your website
- [ ] CIBA flows should _not_ be used for web based logins
- [ ] For plugin providers only:  Your API requests should be sending the headers:
  - [ ] `Vipps-System-Name`
  - [ ] `Vipps-System-Version`
  - [ ] `Vipps-System-Plugin-Name`
  - [ ] `Vipps-System-Plugin-Version`