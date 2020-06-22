# Vipps Login API

Document version 1.0.0.

Reference:\
[vipps-login-api-version-1.0](../../vipps-login-api)\
[vipps-login-api-version-2.0](vipps-login-api.md)

## Table of contents
* [Overview](#overview)
* [Convert to vipps login api version 2.0](#convert-to-vipps-login-api-version-20)

## Overview:
To have a unified way of serving a merchant userinfo between our applications in Vipps we have implemented a new service handling this responsibility.
Since there are different requirements on format between our services we have created a new common response that comply with these.
In vipps login api version 2.0 the format of claim 'birthdate' is changed from dd.mm.yyyy to yyyy-mm-dd, 
and claim 'address' have been converted from list of all addresses to the users preferred address, and the none default addresses are served under 'other_address'.
There is also one change to our supported scopes, we are changing format of scope 'nnin' to 'nin'.

## Convert to vipps login api version 2.0
To control which response to serve we need the merchant to send inn scope 'api_version_2'.
Once starting to use vipps login api version 2.0 if the scope 'nnin' is being requested it has to be converted to 'nin'.
In the new api the claim 'address' is now the preferred address (default) for the Vipps user and no longer a list of all addresses.
This new object will contain the same fields, except for field 'default', which is implied.
THe other addresses, if any, of the user are now served under claim 'other_address' as a list.
The format for claim 'birthdate' has been updated from dd.mm.yyyy to yyyy-mm-dd.
See here for an example of the [userinfo response](https://vippsas.github.io/vipps-login-api/#/public/userinfo).



