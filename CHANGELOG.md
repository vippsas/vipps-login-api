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

All notable changes to the current API will be documented in this file.
To learn about API versioning, see
[Common topics: API Lifecycle](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/api-lifecycle/).

## January 2023

Removed QR code login flow

## September 2021

Vipps Login support for logging in directly from phone number (CIBA) and QR code.

## February 2021

### Launched Vipps Login API version 2

To have a unified way of providing user information across Vipps services
we are standardizing the data formats, both within Vipps and with the OIDC standard.

### Other improvements

The merchant can choose whether the user should be automatically returned
to the browser after completing the authentication Vipps.
We have also added a specific flow for integrations with native apps.
