<!-- START_METADATA
---
title: Automatic return from Vipps or MobilePay app
sidebar_label: Automatic return from Vipps or MobilePay app
sidebar_position: 30
description: Enable automatic return from the Vipps or MobilePay app.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Automatic return from Vipps or MobilePay app

This flow is designed for web pages. It will return users to a browser after completing the login in the Vipps or MobilePay app.
This requires the merchant to handle the user session across browsers.

*This flow has security implications and is not suited for every scenario. Merchants must make their own considerations to ensure that it is only used where suitable*.

Due to how the various mobile operating systems handle app-switch to browser, the user can be returned to a different browser than the one he/she started in.

On iOS, the user can, for example, start the login in Chrome and be returned to Safari after confirming in the Vipps or MobilePay app. This means that the merchant site cannot rely on cookies being present in the browser the user is returned to.

By using this flow, Vipps Login will be able to complete the login process even if the user ends up in a different browser. However, the merchant**must ensure that logins can complete, even without session information, such as cookies.*

## Implementation suggestions

This flow can be enabled per login by adding the parameter `requested_flow=automatic_return_from_vipps_app` to the [Authorize](../browser-flow-integration.md#oauth-20-authorize) request.

It is not possible to give a single description that ensures secure use of this flow for all scenarios. The suggestions given here may not apply to every scenario and must be considered in relation to the specifics of the implementation.

### Session information

The [state parameter](https://developer.vippsmobilepay.com/docs/APIs/login-api/vipps-login-api-faq.md#whats-the-purpose-of-the-state-parameter)  passed in the [OAuth2 authorize endpoint](../browser-flow-integration.md#oauth-20-authorize) request can carry some information from the start of a login until the callback.
The state parameter cannot be thought of as a direct replacement of a user agent bound session.

Some relevant considerations:

* Always use [PKCE](https://oauth.net/2/pkce/).
* Avoid logging the callback URI.
* [Session fixation](https://owasp.org/www-community/attacks/Session_fixation). Be aware of what is possible to set up before a login is started.
* [Login CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#login-csrf). Be aware if it's possible to input sensitive information after a login.
* [The state parameter and CSRF](https://tools.ietf.org/html/rfc6749#section-10.12). Be aware of the recommendations of the OIDC/OAuth standards.

### Verification

Verify that users being returned to a browser different from where the login started are handled as expected.
Test starting the login in private/incognito mode, as this will have similar effects as being returned to a different browser.
