# Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

We also have a short [getting started guide to Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md).

## Setting up Postman

### Step 1: Get the Postman Collection

Import the collection by following the steps below:

1. Click `Import` in the upper left corner.
2. Import the [vipps-login-api-postman-collection.json](https://raw.githubusercontent.com/vippsas/vipps-login-api/master/tools/vipps-login-api.postman_collection.json) file.

### Step 2: Import the Postman Environment

1. Click `Import` in the upper left corner.
2. Import the [vipps-login-api-postman-enviroment.json](https://raw.githubusercontent.com/vippsas/vipps-login-api/master/tools/vipps-login-api.postman_environment.json) file.

### Step 3: Setup Postman Environment

1. Click the "eye" icon in the top right corner.
2. In the dropdown window, click `Edit` in the top right corner.
3. Fill in the `Current Value` for the following fields to get started.
   - `well-known_uri`
   - `client_id`
   - `client_secret`
   - `redirect_uri`

### Step 4: Run tests

1. Send request `Get OIDC well-known`.
1. In Postman environment "vipps-login-api" copy value of key `start_login_uri`, and use this url in any browser.
1. Finish Vipps login.
1. Copy query param `code` from url the browser was redirected to after finishing Vipps login. Paste this code into key `code` in Postman environment `vipps-login-api`.
1. Send request `Get token` to get access token and id token.
1. Send request `Get user info` to get user info of logged in user.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-login-api/issues),
a [pull request](https://github.com/vippsas/vipps-login-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
