## Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

### Setting up Postman

#### Step 1: Import the Postman Collection

Click the link underneath to get access to the eCommerce Postman collection.

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/cd2476c0175210f62d4a)

Alternatively, manually import the collection by following the steps below:
1. Click `Import` in the upper left corner.
2. Import the [vipps-login-api-postman-collection.json](https://raw.githubusercontent.com/vippsas/vipps-login-api/master/tools/vipps-login-api.postman_collection.json) file

#### Step 2: Import the Postman Environment

1. Click `Import` in the upper left corner.
2. Import the [vipps-login-api-postman-enviroment.json](https://raw.githubusercontent.com/vippsas/vipps-login-api/master/tools/vipps-login-api.postman_environment.json) file

#### Step 3: Setup Postman Environment

1. Click the "eye" icon in the top right corner.
2. In the dropdown window, click `Edit` in the top right corner.
3. Fill in the `Current Value` for the following fields to get started.
   - `well-known_uri`
   - `client_id`
   - `client_secret`
   - `redirect_uri`

#### Step 4: Run tests

1. Send request `Get OIDC well-known`.
1. In Postman environment "vipps-login-api" copy value of key `start_login_uri`, and use this url in any browser.
1. Finish Vipps login.
1. Copy query param `code` from url the browser was redirected to after finishing Vipps login. Paste this code into key `code` in Postman environment `vipps-login-api`.
1. Send request `Get token` to get access token and id token.
1. Send request `Get user info` to get user info of logged in user.
