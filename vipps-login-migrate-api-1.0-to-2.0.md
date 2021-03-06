# Vipps Login API 2.0 migration

Document version 1.0.2.

Version 1 of the Vipps login API will have end of life on February 28th 2021.

Reference:\
[vipps-login-api-version-1.0](versions/1.0/vipps-login-api.md)\
[vipps-login-api-version-2.0](vipps-login-api.md)

## Table of contents
* [Overview](#overview)
* [Convert to vipps login api version 2.0](#convert-to-vipps-login-api-version-20)
* [Api changes](#api-changes)

## Overview
To have a unified way of serving a merchant user information between our applications in Vipps we have implemented a new service handling this responsibility.
Vipps Login will be the first service to integrate with this new endpoint.

## API changes
User information will no longer be available as part of the 'id_token'. The 'id_token' will return the ID for the user ('sub'). To get the additional user information a request has to be sent to the '/userinfo' endpoint using this 'sub'. See here for an example of the [userinfo response](https://vippsas.github.io/vipps-login-api/#/public/userinfo).


The following changes apply to the respons from the '/userinfo' endpoint:
1. Scope `nnin` is changed to `nin`.
2. Format of claim `birthdate` is changed from `DD.MM.YYYY` to `YYYY-MM-DD`, to comply with the ISO 8601 format.
3. Claim address have been converted from a list of all address to an object of the default address. All other addresses are served under `other_addresses` as a list. 
This is done to comply with the OIDC-standard which expect address to be a JSON object.
4. User information is no longer served on the `id_token`. Therefore, the `id_token` will only contain user ids. 
This is done since the data in the id token should be used to _look up_ the authenticated user, while the userinfo endpoint should be used to _fetch and store_ user information.   
5. The updated URL to userinfo can be found under [discovery endpoints](vipps-login-api.md#openid-connect-discovery-endpoint), we recommend to fetch these dynamically.


## Convert to Vipps Login API version 2.0

To control which response to serve we need the merchant to send inn scope `api_version_2`.

If your current integration to Vipps Login are getting the users information directly from the 'id_token', then this has to be converted to fetch these from the '/userinfo' endpoint. It is recommended to dynamically get the latest URL for this endpoint from our [discovery endpoints]( https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#openid-connect-discovery-endpoint).

Once starting to use Vipps Login API version 2.0 if the scope `nnin` is being requested the requested scope and the reference to the claim in the respons has to be changed to `nin.

In the new API the claim `address` is now the preferred address (default) for the Vipps user and no longer a list of all addresses.
This new object will contain the same fields, except for field `default`, which is implied.
The other addresses, if any, of the user are now served under claim `other_addresses` as a list.

The format for claim `birthdate` has been updated from `DD.MM.YYYY` to `YYYY-MM-DD`.
