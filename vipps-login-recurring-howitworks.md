<!-- START_METADATA
---
title: How it works with Vipps Recurring
sidebar_position: 16
---
END_METADATA -->

# Vipps Login + Vipps Recurring: How It Works

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

Vipps Login and Vipps Recurring can be used together making registration and payment simple for your customers.

For the complete API documentation go to

* [Vipps Login API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/login-api)
* [Vipps Recurring API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api)

## The process

![Login and recurring process](images/login-recurring-process-v2.svg)

## 1. Buy a subscription

A user wants to buy a subscription on a merchant’s website or app.

![Buy subscription with Vipps](images/login-recurring-step1-v2.svg)

## 2. Login with Vipps

The user logs in with Vipps on the merchant’s site.
If the user is remembered in browser, the login will be completed directly in the browser. If not, the user will be taken to the app to authenticate.

![Login with Vipps](images/login-recurring-step2-v2.svg)

## 3. Confirm login

If the user needs to authenticate in the app, the user will be taken to Vipps to confirm the login.

![Confirm login](images/login-recurring-step3.svg)

## 4. Give consent to share information

If the user has not consented to sharing information with the merchant earlier the user needs to give this consent.
The user may click "See your information" to see the actual information that will be shared, but this is optional.

![Give consent to share information](images/login-recurring-step4.svg)

## 5. Logged in and ready to checkout

This step is controlled and designed by the individual merchant. Typically the user will now be logged in on the merchant’s page, and can proceed to set up the payment for the subscription. The information the user has shared with the merchant is automatically filled in. The merchant can also provide the user with the possibility to edit or add information if necessary.

![Checkout](images/login-recurring-step5-v3.svg)

## 6. Accept agreement in Vipps

The user accepts the agreement in Vipps.

![Agreement in Vipps](images/login-recurring-step6-v2.svg)

## 7. Subscription confirmed

The user is returned to the merchant's web site or app, and the subscription is confirmed on the merchant's site.

![Confirmation page](images/login-recurring-step7.svg)

## Great! Now you know how you can use Vipps Login and Vipps Recurring together

Take a look at the technical documentation in the [Vipps Login API Guide](vipps-login-api.md) and the [Vipps Recurring API Guide](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api).

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).
