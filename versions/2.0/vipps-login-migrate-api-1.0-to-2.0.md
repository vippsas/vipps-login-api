# Vipps Login API

Document version 1.0.0.

Reference:\
[vipps-login-api-version-1.0](../../vipps-login-api)\
[vipps-login-api-version-2.0](vipps-login-api.md)

## Table of contents
* [Overview](#overview)
* [Convert to vipps login api version 2.0](#convert-to-vipps-login-api-version-20)

## Overview:
To have a unified way of serving a merchant user information between our applications in Vipps we have implemented a new service handling this responsibility.
Vipps Login will be the first service to integrate with this new endpoint.
In vipps Login api version 2.0 we are changing format of scope 'nnin' to 'nin'.
We have also have a major change to the response from the '/userinfo' endpoint.
These are changes to the format of claim 'birthdate' from dd.mm.yyyy to yyyy-mm-dd to comply with the ISO 8601 format.
To comply with the OIDC-standard and to highlight the address that the user has set as default in Vipps the claim 'address' have been converted from list of all addresses to only return the users default address,
and the none default addresses are served under 'other_address'.
We will also no longer serve user information in the 'id_token'.

## Convert to Vipps Login api version 2.0
To control which response to serve we need the merchant to send inn scope 'api_version_2'.
Once starting to use Vipps Login api version 2.0 if the scope 'nnin' is being requested it has to be converted to 'nin'.
In the new api the claim 'address' is now the preferred address (default) for the Vipps user and no longer a list of all addresses.
This new object will contain the same fields, except for field 'default', which is implied.
The other addresses, if any, of the user are now served under claim 'other_address' as a list.
The format for claim 'birthdate' has been updated from dd.mm.yyyy to yyyy-mm-dd.
See here for an example of the [userinfo response](https://vippsas.github.io/vipps-login-api/#/public/userinfo).
If the integration to Vipps Login are using the 'id_token' for fetching user information, then this has to be converted to fetch these from the '/userinfo' endpoint.



