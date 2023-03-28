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

API version: 2.0

<!-- START_COMMENT -->

ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/).

<!-- END_COMMENT -->

Vipps Login is the easiest way to sign in and create an account in both digital and physical settings. No typing required! Users don’t need to worry about forgetting or loosing their usernames and passwords. All they need to remember is their phone number. Vipps Login is easier for the user while you as a merchant get better conversion and data quality.

The following topics are described:

- [Overview](overview.md) provides high-level descriptions of the Login API flows.
- [Important information](important-information.md) to help you integrate with Vipps Login.
- [Core concepts](core-concepts.md) related to relevant OIDC concepts and parameters.
- [Browser flow integration](integration.md) describes how to implement the browser flow.
- [App integration](app-integration.md) describes how to integrate from an app.
- Special flows
    - [Automatic return from Vipps app](flows/automatic-return.md) reducing required user interaction when appropriate.
    - [Vipps login from phone number](flows/phone-number-ciba-flows.md) and direct login in the Vipps app.
    - [No dialog flow](flows/no-dialog.md) to check if a user is already logged in.
- [Partner keys](partner-keys.md) facilitating integration for partners.

**Please note:** The information fetched from the Report API is
asynchronous and trailing behind the other APIs. It is usually behind
by a second or so, but if there are operational problems, it could, in the worst
case, be behind by several hours. Therefore, you should always use other
Vipps APIs as the source of truth for the status of an operation.
