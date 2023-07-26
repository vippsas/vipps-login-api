<!-- START_METADATA
---
title: How Login works in the browser
sidebar_label: In the browser
sidebar_position: 13
description: How Login works in the browser.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# How Login works in the browser

Let your customers log in to your website or app without the need for usernames and passwords.

For technical documentation go to
[Vipps Login API](https://developer.vippsmobilepay.com/docs/APIs/login-api).

We also have _How it works_ for [Vipps Login from phone number](vipps-login-from-phone-number-api-howitworks.md)

## The login process

![login process](../images/vipps-login-process-v3.svg)

## The first time using Vipps Login

### 1. Log in with Vipps

A user chooses to log in with Vipps on a merchant’s site.

![Buy subscription with Vipps](../images/vipps-login-step1-2.svg)

### 2. The landing page

If the login was started on a desktop device, the user will be sent to the
[landing page](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/landing-page/).
If the user is not remembered in browser from earlier, the user enters their phone number and can choose to be remembered in the browser for a quicker login the next time. The "Remember me in browser" option applies to the browser, independent of merchants. This option is default off on desktop devices.

The user is then prompted to log in to Vipps and follow the instructions in the app.

If the login was started from a mobile device and the user needs to authenticate in the app, the user will confirm to be taken to the Vipps app without having to enter their phone number.

![landing page](../images/vipps-login-step2.svg)

### 3. E-mail verification (first time only)

If the merchant has requested the user's e-mail address, the user have to verify their e-mail address in Vipps the first time they use Vipps login, or if they change their e-mail address at a later time.

The user can check and change their registered e-mail addresses. This is only necessary to do once. If the merchant has not requested the user's e-mail address, this step is skipped.

![Accept agreement](../images/vipps-login-step3-2.svg)

### 4. Confirm login

The user confirms the login in the Vipps app.

If the login was started on a phone, the user can in this step choose to be remembered in the browser for a quicker registration or login the next time. The "Remember me in browser" option is default on for mobile devices.

If the login was started on desktop, the user confirms that the emoji in Vipps matches the one on the landing page, and confirms the login.

![Confirmation of subscription](../images/vipps-login-step4-2.svg)

### 5. Give consent to share information

The user gives consent to share information with the merchant.

The user may click "See your information" to see the actual information that will be shared, but this is optional.

![Overview of agreements](../images/vipps-login-step5.svg)

### 6. Logged in

The user is now logged in on the merchant’s page.

The user may be given the opportunity to edit the information, for instance
to add additional details not present in the user's Vipps profile.

![Overview of agreements](../images/vipps-login-step6-2.svg)

## Great! Now you know how the Vipps Login process works.

Take a look at the technical documentation in the [Vipps Login API Guide](../api-guide/README.md).
