<!-- START_METADATA
---
title: Automatic return from Vipps app
sidebar_position: 30
---
END_METADATA -->

# Automatic return from Vipps app

When enabled, this flow will automatically take the user back to a browser when they accept the login in the Vipps app. It is not suitable for every scenario. Please see [the detailed description](../overview.md#automatic-return-from-vipps-app) and be aware of the security implications mentioned there.

This flow can be enabled per login by adding the parameter `requested_flow=automatic_return_from_vipps_app` to the [Authorize](../integration.md#oauth-20-authorize) request.

## Implementation suggestions

**It is not possible to give a single description that ensures secure use of this flow for all scenarios. The suggestions given here may not apply to every scenario and must be considered in relation to the specifics of the implementation.**

### Session information

The [state parameter](https://vippsas.github.io/vipps-developer-docs/docs/APIs/login-api/vipps-login-api-faq.md#whats-the-purpose-of-the-state-parameter)  passed in the [OAuth2 authorize endpoint](../integration.md#oauth-20-authorize) request can carry some information from the start of a login until the callback.
The state parameter cannot be thought of as a direct replacement of a user agent bound session.

Some relevant considerations:

* Always use [PKCE](https://oauth.net/2/pkce/).
* Avoid logging the callback URI.
* [Session fixation](https://owasp.org/www-community/attacks/Session_fixation). Be aware of what is possible to set up before a login is started.
* [Login CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#login-csrf). Be aware if it's possible to input sensitive information after a login.
* [The state parameter and CSRF](https://tools.ietf.org/html/rfc6749#section-10.12). Be aware of the recommendations of the OIDC/OAuth standards.

### Verification

It is important that merchants verify that users returning to a different browser than where the login started are handled as expected. It is also recommended to test starting the login in private/incognito mode, as this will have similar effects as being returned to a different browser.
