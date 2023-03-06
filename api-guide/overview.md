<!-- START_METADATA
---
title: Overview
sidebar_position: 30
---
END_METADATA -->

# Overview



## Introduction

Vipps Login is available for several use cases:

* On web pages and in apps - Vipps Login is used in the browser
* Directly from merchant systems (e.g. point of sales systems or call-center solutions) based on phone number
* Directly from machines and vending machines based on phone number

Users can create a new account through sharing high-quality data from the user’s Vipps profile. Available information includes name, email, addresses, phone number, and birth date. [Norwegian national identity number](../vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) is also available to some merchants.  The identity of all Vipps users is verified as part of Vipps' know your customer process, so rest assured that these are real people with correct name and information.

When Vipps Login is used in the browser, the user can choose to be remembered in the browser, enabling automatic sign-ins for later visits. The possibility for the user to be remembered in browser is a key feature of the service and cannot be controlled (turned off) by individual merchants. This means that it is not possible to set up Vipps Login is a 2 factor authentication (2FA).

Vipps Login is _not_ an electronic identification or certified eID.

For **Vipps Login in browser** there are **plugins** available for several platforms. You find the complete list on our [plugins page](https://vippsas.github.io/vipps-developer-docs/docs/vipps-plugins).
If a plugin is not available, the easiest - and **strongly recommended - way to integrate with the service is to use a well renowned OAuth2.0/OpenID Connect Library for your programming language**. Vipps does not recommend a specific library, but the list of [OIDC Relying Party libraries](https://openid.net/developers/certified/) certified by the OpenID Foundation is a good starting point.

Vipps Login in browser should only be run in the browser window using redirects (iFrame is not supported and new window is not recommended).

[**Vipps Login from phone number**](#vipps-login-from-phone-number) is based on the Client initiated backchannel authentication (CIBA) standard and have been developed to support use cases where it is the merchant/merchant's systems that trigger the authentication/registration and thus login cannot be done in the user's browser.

Vipps Login from phone number is available for all Vipps Login enabled clients.

See our [developer section](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/test-environment#vipps-test-apps) for information about our test environment, test apps and test users.


## Flows

### Vipps Login in browser

The generic steps in the Vipps Login in browser flow are shown in our [How it works guide](../how-it-works). This also explains how the login in browser flow adapts to different preconditions in regard to whether the user has chosen to be remembered in the browser they are currently using and whether the user has already consented to share information with the specific merchant (sales unit). Below is a more detailed description on alternative flows and the choices that can be done to impact these flows when implementing Vipps Login.

#### Remembered flow

If a user has chosen to be remembered in browser, then the authentication can be completed in the browser. The user will then either be asked to provide consent to share profile information or be logged in directly. This applies to both desktop and mobile.

If the user is not remembered the user needs to confirm the login in the Vipps-app. The flow associated with this will differ depending on whether the user is on desktop or mobile:

#### Desktop flow - phone number based push flow

If the user is on desktop and not remembered in browser, then they will follow this flow. If the user is remembered in browser, then only the consent flow at the bottom will be completed. If the user already has provided consent, then this step will be skipped also, allowing a direct login experience.

The user initiates the login by entering the phone number and selecting whether to be remembered in the browser:
![Number input in desktop](../images/Number_input_flow_desktop1.png)

The user goes to the Vipps app and confirms the login:
![Confirmation in app in phone number flow](../images/Number_input_flow_app.png)

The user is then authenticated in browser and can provide consent if required. Then the user is redirected back to the redirect URI provided by merchant:
![Consent in desktop](../images/Number_input_flow_desktop2.png )

#### Mobile flow - deeplink based flow

If the user is on a mobile device, the Vipps landing page in the browser will automatically trigger a deeplink to the Vipps app if the user is not remembered in the browser. The user will not be prompted to enter the phone number.

In the Vipps app, the user confirms the login and can choose whether to be remembered in the browser for later logins. After confirming in the app, the user needs to switch back to the Vipps page in the browser/app. On the Vipps landing page, the user will finalize the authentication and provide consents if required. The user is then redirected back to the redirect URI provided by merchant (could be webpage or an app).

![Mobile flow with app-switch](../images/Mobile_flow_with_partial-app_switch.png)

Apps should follow the [recommendations](important-information.md#using-vipps-login-in-native-applications) to use correct browser types for their platform.

There are two specialised flows that merchants can use to automatically switch the user back from the Vipps-app to the originating browser/app upon login confirmation. From the illustration above, this means that the page "Gå tilbake til butikken" will be skipped and that the "manual app switch" will be replaced by an automatic app-switch (eg. deeplink). These flows give a better user experience than the standard flow, but they also require the merchant to handle some more complexity in the integration.

Which of the flows to use is controlled with the initiation of the individual login session. The merchant can thus use all available login flows on the same client_id and adapt to the different use cases and login scenarios.

The flows are described below.

##### App to app flow

This flow is designed to be used with apps. It requires that the app initiate Vipps Login in an external browser that is opened within the app, see [specification](important-information.md#using-vipps-login-in-native-applications). In this flow the merchant need to specify the app URI where the user will be returned after completing the confirmation in the Vipps app.

See [how to implement](flows/app-integration).

##### Automatic return from Vipps app

Requires the merchant to handle user session cross browsers.

This flow is designed for web-pages that would like to have the user automatically returned to a browser after completing the confirmation in the Vipps app. Note that there are security implications by using this flow. **It is not suited for every scenario. Merchants must make their own considerations to ensure that it is only used where suitable**.

Due to how the different mobile operating systems handle app-switch to browser, the user can be returned to a different browser than the one he/she started in. On iOS the user can e.g. start the login in Chrome and be returned to Safari after confirming in the Vipps app. This means that the merchant site cannot rely on cookies being present in the browser the user is returned to.

By using this flow Vipps Login will be able to complete the login process even if the user ends up in a different browser. However, the merchant **must ensure that logins can complete, even without session information like cookies.**

See [how to implement](flows/automatic-return), including more information on the security considerations.

#### No dialog flow - log the user in directly when possible

This flow can be used to log in the user directly, if the required prerequisites are in place. If the prerequisites are not in place, then the Vipps Login process will be stopped and no interaction will be requested from the user in this flow. When using this flow, a spinner will be shown while Vipps Login tries to log the user in. Once the process is completed, the user will be returned to the merchant as in the ordinary Vipps Login flow. As with the other Vipps Login flow, it is recommended to run Vipps Login in a redirect mode and iFrame is not supported.

The user will be logged in with this flow if:

* they are remembered in the browser and no consent is required
* they are remembered in the browser and consent has previously been given

Illustration of how the flow can look when the user clicks "Logg inn" on the front page:

![No dialog flow](../images/No_dialog_flow.png)

Possible use cases includes:

* When the user is going to a section of your site/service that requires them to be logged in, e.g. my page or a personalized chatbot. This might be when the user is already on the webpage, or if the user is being linked directly to the page from an email/newsletter.
* When a user clicks on your login option, it is possible to try to log the user in with Vipps first.

In such scenarios, a user that can be logged in directly will get an even better login experience and quickly come to the information and services that are relevant for them.

If the user cannot be logged in directly, you can e.g. show them your ordinary login screen. On the login screen, Vipps will be an option. A user that is not remembered in browser or has not yet given consent can actively login with Vipps from here.

See [how to implement](flows/no-dialog).

### Vipps Login from phone number

Vipps Login from phone number is based on the Client Initiated Backchannel Authentication (CIBA) standard and built to support special cases where login does not start in browser or app. This means that it is the merchant/merchant's systems that trigger the authentication/registration and thus login cannot be done in the user's browser. This could typically be physical contexts e.g point of sales solutions, on the phone e.g call-center solutions or devices/terminals e.g TV-boxes. To ensure a consistent user experience on webpages and in apps, it is not allowed to use Vipps Login from phone number for such use cases.

It is recommended to use the same sales unit for all use cases to ensure you, as a merchant, get the same user id ('sub') on the user across different scenarios.

Vipps Login from phone number is initiated using the user's mobile number. This triggers a push message from the user's Vipps app. By clicking the push message, the user is taken to Vipps to confirm the authentication/registration. If the user has not already consented to share information with a merchant, such consent will be required. If the user has not enabled push from Vipps, they need to manually open the Vipps app and possibly pull the home screen down for a refresh to receive the authentication request.

The merchant controls whether the user should get the confirmation of completion in the Vipps app or if they should be taken to the merchant's web page to finalize the flow. The merchant can e.g. take the user to their web page to enable input of more information, accept terms and condition, log the user in at their web page, show relevant information/offers or to continue to set up an agreement or completing a purchase. This is illustrated in [How It Works](../how-it-works/vipps-login-from-phone-number-api-howitworks.md).

Illustration of how the flow will look like when the user end the flow and get the confirmation of completion in the Vipps app:

![Confirm completion in Vipps app](../images/CIBA_flow_in_app.png)

Illustration of how the flow will look like if the user is taken to the merchant's web page:

![Redirect to browser](../images/CIBA_flow_take_to_merchant.png)

The merchant has the option to show a confirmation code (`binding_message`) to the user in the app for added security:

![Optional confirmation code (`binding_message`)](../images/CIBA_Confirmation_code.png)

See [how to implement](flows/ciba-flows).

### Vipps Login + Vipps Recurring

* [Vipps Login + Vipps Recurring](../how-it-works/vipps-login-recurring-howitworks.md): Learn how you can use Login and Recurring together.
