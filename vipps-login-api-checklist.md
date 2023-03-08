<!-- START_METADATA
---
title: Login API checklist
sidebar_label: Checklist
sidebar_position: 40
description: Checklist for full integration with Vipps login.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Login API checklist

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

API version: 2.0.

## Checklist for full integration

- [ ] Integration with the Vipps Login API is redirect based (i.e no embedded iframe)
- [ ] Native app integrations use the [app-to-app flow](api-guide/overview.md#app-to-app-flow)
- [ ] Ensure that your solution is verified to work if the user start in a "non-default" browser on mobile, e.g. start Vipps Login from Chrome browser on iOS.
- [ ] If you as a merchant will act on behalf of others (share data you have gotten from Vipps with other merchants) ensure that you comply with our terms and conditions to do this. This is shown in section 8.4 in our [terms and conditions for merchants](api-guide/overview.md#vipps-login-from-phone-number)
- [ ] [Vipps Login from phone number](api-guide/overview.md#vipps-login-from-phone-number)
      must _not_ be used for web based login
- [ ] Vipps-buttons and Vipps branding on your website follow the
      [Vipps design guidelines](https://vippsas.github.io/vipps-developer-docs/docs/vipps-design-guidelines).
- [ ] Terms and conditions are presented to the user, and necessary consents are
      collected from the user (i.e consent to marketing purposes, etc)  
- [ ] The sales unit name (appears on the Vipps Login landing page) corresponds
      to the name of your company on your website. See
      [how to change name](vipps-login-api-faq.md#how-can-i-change-my-name-and-logo)
- [ ] The company logo, to be shown in Vipps on the list which the user has
      shared information with, is set. See
      [how to change logo](vipps-login-api-faq.md#how-can-i-change-my-name-and-logo)
- [ ] Whitelist redirect URIs:
  - [ ] All `redirect_uri`s for your integration has been added using to the
          merchant portal white list, see the
      [FAQ](vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login) for how to do this
  - [ ] All URLs use HTTPS or native URL schemes (like `vipps://`), not HTTP.
  - [ ] ALL URLs match the exact URI sent on `/auth` request. No query parameters, no additional trailing `/`.
- [ ] The scopes requested are only the scopes required, no "nice to have" scopes.
- [ ] If using national identity number: You have been granted access to request these scopes.
      [Apply here](vipps-login-api-faq.md#who-can-get-access-to-nin-and-how)
- [ ] A unique `state`
      [parameter](vipps-login-api-faq.md#whats-the-purpose-of-the-state-parameter)
      is generated for each `/auth` request.
- [ ] The `client_secret` is kept secret and is never be shared to browsers or native apps.
- [ ] Cancelled logins and error situations on the redirect back to `redirect_uri`
      (redirects with a `error` query parameter) are handled.
- [ ] Proper linking of the Vipps user to your own user registry is implemented.
      This login must be based on either phone number or e-mail address. See
      [recommendations on linking to user account](api-guide/important-information.md#Recommendations-on-linking-to-user-account)
- [ ] Send the [Vipps HTTP headers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/common-topics/http-headers)
      in all API requests for better tracking and troubleshooting:
  - [ ] `Merchant-Serial-Number`
  - [ ] `Vipps-System-Name`
  - [ ] `Vipps-System-Version`
  - [ ] `Vipps-System-Plugin-Name`
  - [ ] `Vipps-System-Plugin-Version`
