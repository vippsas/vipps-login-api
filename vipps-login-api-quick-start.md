<!-- START_METADATA
---
title: Quick start
sidebar_position: 20
---
END_METADATA -->

# Quick start

Use the Login API to confirm your identity through the log-in process.

<!-- START_TOC -->
## Table of Contents

* [Postman](#postman)
  * [Prerequisites](#prerequisites)
  * [Step 1: Get the Vipps Postman collection and environment](#step-1-get-the-vipps-postman-collection-and-environment)
  * [Step 2: Import the Vipps Postman files](#step-2-import-the-vipps-postman-files)
  * [Step 3: Set up Postman environment](#step-3-set-up-postman-environment)
* [Make API calls](#make-api-calls)
<!-- END_TOC -->

Document version 1.0.4.

## Postman

### Prerequisites

Review
[Vipps quick start guides](https://github.com/vippsas/vipps-developers/blob/master/vipps-quick-start-guides.md)
for information about getting your test environment set up.

### Step 1: Get the Vipps Postman collection and environment

Save the following files to your computer:

* [Vipps Login API Postman collection](tools/vipps-login-api.postman_collection.json)
* [Vipps API Global Postman environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json)

### Step 2: Import the Vipps Postman files

1. In Postman, click *Import* in the upper-left corner.
1. In the dialog that opens, with *File* selected, click *Upload Files*.
1. Select the two files you have just downloaded and click *Import*.

### Step 3: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
1. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
1. Fill in the `Current Value` for the following fields to get started.
   * `client_id` - Merchant key is required for getting the access token.
   * `client_secret` - Merchant key is required for getting the access token.
   * `well-known_uri` - URL to well-known endpoint for used environment. In the test environment, you can use <https://apitest.vipps.no/access-management-1.0/access/.well-known/openid-configuration>. See [API Guide: well known](vipps-login-api#openid-connect-discovery-endpoint) for more details.
   * `redirect_uri` - The URL where the user is sent after finishing a login. The URL must be exactly the same as the one specified on <https://portal.vipps.no> in the "Setup login" section for your sale unit. For testing, you could, for example, use <http://localhost> in both places.

## Make API calls

1. Send request `Get OIDC well-known`.
1. In your active Postman environment, "vipps-login-api", copy the value of key `start_login_uri`, and use this URL in any browser.
1. Finish Vipps login. This request includes a scope parameter that requests access to user information. This means that if you have not yet consented to sharing your user information, a distinct screen asking for your consent will appear the first time.
1. Copy the query param `code` from the URL that the browser was redirected to after finishing Vipps Login. Paste this code into the key `code` in the active Postman environment.
1. Send request `Get token` to get the access token and id token.
1. Send request `Get user info` to get user info of the logged-in user.

See the
[API reference](https://vippsas.github.io/vipps-developer-docs/api/login)
for details about the calls.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
