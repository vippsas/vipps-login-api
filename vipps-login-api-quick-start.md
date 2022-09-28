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
[Vipps quick start guides](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/vipps-quick-start-guides) for information about getting your test environment set up.

### Step 1: Get the Vipps Postman collection and environment

Save the following files to your computer:

* [Vipps Login API Postman collection](tools/vipps-login-api.postman_collection.json)
* [Vipps Login API Postman environment](tools/vipps-login-api.postman_environment.json)

### Step 2: Import the Vipps Postman files

1. In Postman, click *Import* in the upper-left corner.
1. In the dialog that opens, with *File* selected, click *Upload Files*.
1. Select the two files you have just downloaded and click *Import*.

### Step 3: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
1. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
1. Fill in the `Current Value` for the following fields to get started.
   * `well-known_uri`
   * `client_id`
   * `client_secret`
   * `redirect_uri`

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
