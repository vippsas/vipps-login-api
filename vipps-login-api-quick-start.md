<!-- START_METADATA
---
title: Quick start
sidebar_position: 20
---
END_METADATA -->

# Quick start

<!-- START_TOC -->

## Table of Contents

* [Postman](#postman)
  * [Step 1: Get the Postman collection](#step-1-get-the-postman-collection)
  * [Step 2: Import the Postman environment](#step-2-import-the-postman-environment)
  * [Step 3: Set up Postman environment](#step-3-set-up-postman-environment)
* [Make API calls](#make-api-calls)

<!-- END_TOC -->

Document version 1.0.2.

## Postman

### Step 1: Get the Postman collection

Import the collection by following the steps below:

1. Click `Import` in the upper-left corner.
2. Import the [vipps-login-api-postman-collection.json](tools/vipps-login-api.postman_collection.json) file.

### Step 2: Import the Postman environment

1. Click `Import` in the upper-left corner.
2. Import the [vipps-login-api.postman_environment.json](tools/vipps-login-api.postman_environment.json) file.

### Step 3: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
1. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
1. Fill in the `Current Value` for the following fields to get started.
   - `well-known_uri`
   - `client_id`
   - `client_secret`
   - `redirect_uri`

## Make API calls

1. Send request `Get OIDC well-known`.
1. In Postman environment "vipps-login-api" copy value of key `start_login_uri`, and use this url in any browser.
1. Finish Vipps login.
1. Copy query param `code` from url the browser was redirected to after finishing Vipps login. Paste this code into key `code` in Postman environment `vipps-login-api`.
1. Send request `Get token` to get access token and id token.
1. Send request `Get user info` to get user info of logged in user.

See the
[API reference](https://vippsas.github.io/vipps-developer-docs/api/login)
for details about the calls.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
