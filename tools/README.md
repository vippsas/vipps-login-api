# Vipps login Postman guide
API version: 0.9

Document version 0.9

##How to use postman
1. Import vipps-login-api.postman_collection.json and vipps-login-api.postman_environment.json to Postman. 
1. Add values to the following keys in the Postman environment `vipps-login-api`;
    * `well-known_uri`
    * `client_id`
    * `client_secret`
    * `redirect_uri`
1. Send request `Get OIDC well-known`.
1. In Postman environment "vipps-login-api" copy value of key `start_login_uri`, and use this url in any browser.
1. Finish Vipps login.
1. Copy query param `code` from url the browser was redirected to after finishing Vipps login. Paste this code into key `code` in Postman environment `vipps-login-api`.
1. Send request `Get token` to get access token and id token.
1. Send request `Get user info` to get user info of logged in user.