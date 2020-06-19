# Vipps Login API

Document version 1.0.0.

Reference:\
[vipps-login-api-version-1.0](../../vipps-login-api)\
[vipps-login-api-version-2.0](vipps-login-api.md)

## Table of contents
* [Overview](#overview)
* [Convert to vipps login api version 2.0](#convert-to-vipps-login-api-version-20)

## Overview:
In vipps login api version 2.0 we are changing format of scope 'nnin' to 'nin'.
We have also have a major change to the response from the '/userinfo' endpoint.
These are changes to the format of claim 'birthdate' from dd.mm.yyyy to yyyy-mm-dd.
Claim 'address' have been converte from list of all addresses to the users preferred address, 
and the none default addresses are served under 'other_address'.

## Convert to vipps login api version 2.0
Until we can fully switch to vipps login api version 2.0 we need every merchant to additionally send inn scope 'api_version_2'.
We are using this scope to internally handle which response to serve.
Once starting to use vipps login api version 2.0 the scope 'nnin' has to be converted to 'nin'.
New format for claim 'address' and 'birthdate' and new claim 'other_address' has to be handled.
See here for an example of the [userinfo response](https://vippsas.github.io/vipps-login-api/#/public/userinfo).



