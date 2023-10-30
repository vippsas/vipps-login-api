<!-- START_METADATA
---
title: Login API changelog
sidebar_label: Changelog
sidebar_position: 200
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Changelog

![Vipps](./images/vipps.png) *Available for Vipps now.*

![MobilePay](./images/mp.png) *Available for MobilePay in selected markets at the [Vipps MobilePay joint platform launch](https://www.vippsmobilepay.com/about).*

All notable changes to the current API will be documented in this file.
To learn about API versioning, see
[API Lifecycle](https://developer.vippsmobilepay.com/docs/knowledge-base/api-lifecycle/).

## January 2023

Removed QR code login flow.

## September 2021

Vipps Login support for logging in directly from phone number (CIBA) and QR code.

## February 2021

### Launched Login API version 2

To have a unified way of providing user information across Vipps services
we are standardizing the data formats, both within Vipps and with the OIDC standard.

### Other improvements

The merchant can choose whether the user should be automatically returned
to the browser after completing the authentication Vipps.
We have also added a specific flow for integrations with native apps.
