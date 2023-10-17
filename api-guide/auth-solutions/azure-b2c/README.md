<!-- START_METADATA
---
title: Vipps in Azure AD B2C
sidebar_label: Azure AD B2C
sidebar_position: 200
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Azure AD B2C

The following will describe how Vipps ePayment and Login can be integrated with Azure AD B2C to achieve a better flow for user registration and payment. The login and payment flow can be implemented both separately and together. Implementing the flows together allow users to be registered during payment and later log in as a registered user.

Azure Active Directory Business-to-Customer (Azure AD B2C) is a customer identity
management solution. Learn more about it on the
[Microsoft pages](https://learn.microsoft.com/en-us/azure/active-directory-b2c/overview), as Vipps support will not help you to configure the service.

These guides describe how to use the login and ePayment APIs with Azure AD B2C to achieve a better flow. The login guide will help you to configure Vipps as an identity provider in Azure AD B2C and retrieve contact info from the user that logs in. 

* [Vipps login in Azure AD B2C using custom policies](CustomPolicyLogin.md)
* [Vipps payment flow using Azure AD B2C](PaymentFlowB2C.md)
