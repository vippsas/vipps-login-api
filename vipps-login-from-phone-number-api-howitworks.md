<!-- START_METADATA
---
title: How it works from mobile
sidebar_position: 14
---
END_METADATA -->

# Vipps Login from phone number: How It Works

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

Let your customers share their information or confirm who they are by triggering a Login flow on their device based on their phone number.

For technical documentation go to
[Vipps Login API - login from phone number](vipps-login-api.md#vipps-login-from-phone-number).

We also have _How it works_ for [Vipps Login in browser](vipps-login-api-howitworks.md) and [Vipps Login from QR-code](vipps-login-from-QR-api-howitworks.md).

This flow also features in our [Loyalty at POS solution](https://vippsas.github.io/vipps-developer-docs/docs/vipps-solutions/loyalty-in-pos/).

## The login from phone number process

The merchant has several options available to make the flow tailored to the specific use case:
 * Binding message: Include a binding message in the confirmation step for some extra assurance
 * Delegated consents: In this extra step, merchants can collect the consents required to enroll the customer to your loyalty program
 * Redirect: You may choose to redirect the user to a website of your choosing after login is completed

![login process](images/vipps-login-from-phone-number-process.png)

The different steps are described and illustrated with screenshots below.

### 1. Initiate Vipps Login from phone number

The merchant initiates Vipps Login on the users device from their system, based on the user's phone number.
The user receives a push notification from Vipps and clicks the push to open the Vipps app.

If the user has turned off push notifications or for some other reason do not get the push, they can open Vipps themselves and automatically start the login flow.

![User gets push message from Vipps](images/vipps-login-phone-push.png)

### 2. Confirmation

After opening Vipps, the user will be asked to confirm sharing information with the merchant. The first screen will specify the information requested by the merchant.

On this screen, the merchant can choose to show a binding message (confirmation code). This is for added security if the merchant would like their representatives or their customers to have an explicit check that it is the correct transaction that is being accepted.

After confirming the login and sharing of information, the user will be asked for the formal consent to share information from Vipps to the merchant.
The end user must only consent once per merchant and the consent applies across all Vipps services. In other words, if the user has already consented to share the
requested information with the merchant, no consent will be required, and this step will be skipped.

![User confirm in Vipps](images/vipps-login-confirm.png)

### 3. Delegated consents (optional)

The merchant may choose to include the delegatedConsents scope to collect consents from the end user in Vipps, on behalf of the merchant.
This scope is relevant for merchants who use Vipps Login to enroll end-users into customer club/loyalty program from POS system when the user is present in-store.
If the scope is included in the request, the end user will be asked to give consents according to merchant specifications.

|                       Initial delegated consents screen                       |                           Delegated consents approved                            |
|:-----------------------------------------------------------------------------:|:--------------------------------------------------------------------------------:|
| ![User sees delegated consents in Vipps](images/delegated-consents-step1.png) | ![User confirm delegated consents in Vipps](images/delegated-consents-step2.png) |

Some of the elements on the delegatedConsents screen can be adjusted to fit the merchant's needs
 * A top text where the purpose of collecting consents should be stated. This is customizable for you as a merchant.
 * The consents wanted by the merchant (see more [When using delegatedConsents, which consents does Vipps support?](vipps-login-api-faq.md#when-using-delegatedconsents-which-consents-does-vipps-support)).
   These can be individually specified to be mandatory or optional. Mandatory consents must be ticked before the user can complete the login flow. See what consents we currently support, or reach out if you need something else and we'll se what we can do!
 * A bottom text where we provide some contextual information for the end user. The text is fixed, but the links will point to the merchant's terms and conditions, as well as privacy statement.

### 4. Redirect or end the login in Vipps

The merchant controls what happens after the user has confirmed the login and sharing of information.

The merchant can end the login process in Vipps, where the user will receive a confirmation screen that the login is complete.

Alternatively, the merchant may redirect the user to it's website. The merchant will then have a identified user. This flow can be used to welcome a user to the customer club, ask for more information/preferences, show offers or information of relevance.

![User gets confirmation in Vipps or is taken to browser](images/vipps-login-confirmation.png)

## Great! Now you know how Vipps Login from phone number works!

Take a look at the technical documentation in the [Vipps Login API - login from phone number](vipps-login-api.md#vipps-login-from-phone-number).

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).
