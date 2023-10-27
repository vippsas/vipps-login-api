<!-- START_METADATA
---
title: Overview of the Login API
sidebar_label: Overview
sidebar_position: 10
description: Vipps Login is available for several use cases.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Overview of the Login API

## Introduction

Vipps Login is available for several use cases:

* On web pages and in apps - Vipps Login is used in the browser
* Directly from merchant systems (e.g. point of sales systems or call-center solutions) based on phone number
* Directly from machines and vending machines based on phone number

Users can create a new account through sharing high-quality data from the user’s Vipps or MobilePay profile. Available information includes name, email, addresses, phone number, and birthdate. [Norwegian national identity number](../vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) is also available to some merchants.  The identity of all Vipps MobilePay users is verified as part of our Know Your Customer (KYC) process, so rest assured that these are real people with correct name and information.

When Vipps Login is used in the browser, the user can choose to be remembered in the browser, enabling automatic sign-ins for later visits. The possibility for the user to be remembered in browser is a key feature of the service and cannot be controlled (turned off) by individual merchants. This means that it is not possible to set up Vipps Login as a two-factor authentication (2FA).

Vipps Login is *not* an electronic identification or certified eID.

For *Vipps Login in browser*, there are *plugins* available for several platforms.
You find the complete list on our [plugins page](https://developer.vippsmobilepay.com/docs/plugins).
If a plugin is not available, the easiest - and **strongly recommended** - way to integrate with the service
is to use a well renowned OAuth2.0/OpenID Connect Library for your programming language.
We don't recommend a specific library, but the
[Certified OpenID Developer Tools](https://openid.net/developers/certified/) site is a good starting point.

Vipps Login in browser should only be run in the browser window using redirects (iFrame is not supported, and new window is not recommended).

[**Vipps Login from apps**](#app-to-app-flow) supports switching the user automatically between apps and is based on logins through a browser.

[**Merchant initiated login**](#merchant-initiated-login) is based on the Client initiated backchannel authentication (CIBA) standard and have been developed to support use cases where it is the merchant/merchant's systems that trigger the authentication/registration and thus login cannot be done in the user's browser.

Merchant initiated login is available for all Vipps Login enabled clients.

See the [Test environment](https://developer.vippsmobilepay.com/docs/test-environment#vipps-test-apps)
page for information about our test environment, test apps and test users.

## Flows

A sales unit can use the Login API for many flows.
It is recommended to use the same sales unit for all use cases to ensure that you get the same user ID (`sub`) on the user across different scenarios.

The Login API can also be combined with other APIs to make registration and payment simple for users.
See an example for simplifying registration and payment in [Subscriptions](https://developer.vippsmobilepay.com/docs/solutions/recurring-and-login).

### Login in browser

The generic steps in the *Login in browser* flow are shown in our [How it works guide](../how-it-works/README.md). This also explains how the *Login in browser* flow adapts to different preconditions.

For example, conditions may include whether the user has chosen to be remembered in the browser they are currently using and whether they have already consented to share information with the specific merchant (sales unit).

Below is a more detailed description on alternative flows and the choices that can be done to impact these flows when implementing Vipps Login.

#### Remembered flow

If a user has chosen to be remembered in browser, then the authentication can be completed in the browser. The user will then either be asked to provide consent to share profile information or be logged in directly. This applies to both desktop and mobile.

If the user is not remembered the user needs to confirm the login in the Vipps or MobilePay app. The flow associated with this will differ depending on whether the user is on desktop or mobile:

#### Desktop flow - phone number based push flow

If the user is on desktop and not remembered in browser, then they will follow this flow. If the user is remembered in browser, then only the consent flow at the bottom will be completed. If the user already has provided consent, then this step will be skipped also, allowing a direct login experience.

The user initiates the login by entering the phone number and selecting whether to be remembered in the browser:
![Number input in desktop](../images/Number_input_flow_desktop1.png)

The user goes to the Vipps or MobilePay app and confirms the login:
![Confirmation in app in phone number flow](../images/Number_input_flow_app.png)

The user is then authenticated in browser and can provide consent if required. Then the user is redirected back to the redirect URI provided by merchant:
![Consent in desktop](../images/Number_input_flow_desktop2.png )

#### Mobile flow - deeplink based flow

If the user is on a mobile device, the Vipps MobilePay landing page
in the browser will automatically trigger a deeplink to the Vipps or MobilePay app if the user is not remembered in the browser. The user will not be prompted to enter the phone number.

In the Vipps or MobilePay app, the user confirms the login and can choose whether to be remembered in the browser for later logins. After confirming in the app, the user needs to switch back to the Vipps MobilePay page in the browser/app. On the landing page, the user will finalize the authentication and provide consents if required. The user is then redirected back to the redirect URI provided by merchant (could be webpage or an app).

![Mobile flow with app-switch](../images/Mobile_flow_with_partial-app_switch.png)

Apps should follow the [recommendations](important-information.md#using-vipps-login-in-native-applications) to use correct browser types for their platform.

There are two specialized flows that merchants can use to automatically switch the user back from the Vipps or MobilePay app to the originating browser/app upon login confirmation. From the illustration above, this means that the page "Gå tilbake til butikken" will be skipped and that the "manual app switch" will be replaced by an automatic app-switch (e.g., deeplink). These flows give a better user experience than the standard flow, but they also require the merchant to handle some more complexity in the integration.

Which of the flows to use is controlled with the initiation of the individual login session. The merchant can thus use all available login flows on the same client_id and adapt to the different use cases and login scenarios.

The flows are described below.

##### App-to-app flow

It is possible to enable automatic switch of users back to the merchant app,
from the Vipps or MobilePay app.

This flow is designed to be used with apps. It requires that the app initiate Vipps Login in an external browser that is opened within the app.

See [App integration](app-integration.md) for details about integrating.

##### Automatic return from Vipps or MobilePay app

This flow will automatically take the user back to a browser when they accept the login in the Vipps or MobilePay app.

There are security implications by using this flow which requires the merchant to handle the user session across browsers.
 **It is not suited for every scenario. Merchants must make their own considerations to ensure that it is only used where suitable**.

By using this flow, Vipps Login will be able to complete the login process even if the user ends up in a different browser. However, the merchant **must ensure that logins can complete, even without session information like cookies.**

See [Automatic return from Vipps or MobilePay app](flows/automatic-return.md) for details about implementation and security considerations.

### Merchant initiated login

*Merchant initiated login* is built to support special cases where login does not start in a browser or app. This could typically be physical contexts (e.g., point of sales solutions), on the phone (e.g., call-center solutions), or devices and terminals (e.g., TV boxes).

In this flow, the merchant's system triggers the authentication/registration; thus, log-in cannot be done in the user's browser.
To ensure a consistent user experience, *Merchant initiated login* is not allowed on webpages and in apps.

*Merchant initiated login* is available for all Login-enabled clients.

See:

* [How merchant initiated login works](../how-it-works/merchant-initiated-login-howitworks.md) for illustrations.
* [Merchant initiated login integration](flows/merchant-initiated-login-integration.md) for the technical details.
