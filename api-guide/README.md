<!-- START_METADATA
---
title: Login API guide
sidebar_label: API guide
sidebar_position: 30
description: Login API guide.
pagination_prev: Null
pagination_next: Null
---
END_METADATA -->

# Login API guide

![Vipps](../images/vipps.png) *Available for Vipps now.*

![MobilePay](../images/mp.png) *Available for MobilePay in selected markets at the [Vipps MobilePay joint platform launch](https://www.vippsmobilepay.com/#about).*

<!-- START_COMMENT -->
ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/login-api).
<!-- END_COMMENT -->

API version: 2.0

Vipps Login is the easiest way to sign in and create an account in both digital and physical settings. No typing required! Users don’t need to worry about forgetting or loosing their usernames and passwords. All they need to remember is their phone number. Vipps Login is easier for the user while you as a merchant get better conversion and data quality.

The following topics are described:

* [Overview](overview.md) provides high-level descriptions of the Login API flows.
* [Important information](important-information.md) to help you integrate with Vipps Login.
* [Core concepts](core-concepts.md) related to relevant OIDC concepts and parameters.
* [Website integration](browser-flow-integration.md) describes how to integrate from a website.
* [App integration](app-integration.md) describes how to integrate from an app.
* Special flows
  * [Automatic return from Vipps or MobilePay app](flows/automatic-return.md) reducing required user interaction when appropriate.
  * [Merchant initiated login](flows/merchant-initiated-login-integration.md) describes implementing a flow where the merchant's system initiates the login towards a user.
* [Partner keys](partner-keys.md) facilitating integration for partners.
* [Authentication solutions](./auth-solutions/README.md): How to use the Login API with Auth0 and Azure AD B2C authentication solutions.
