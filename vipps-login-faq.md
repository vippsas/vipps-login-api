# Frequently Asked Questions for Vipps Login API

See the [Vipps Login API](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md) for all the details.

See also the
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide for the Vipps Developer Portal.
You can also find frequently asked questions on the product here [Product FAQ](https://vipps.no/hjelp/vipps/vipps-logg-inn). 

# Table of contents



# Where do I find the client ID and client secret?
You find more information on this here: https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#getting-the-api-keys


If you cannot access VippsPortalen: do you have a user for the merchant? The person who were added as the contact person in the request is set as the administrator and can add new users. If you have added a new service to an existing merchant, you need to have the existing administrator add new users if required.

If you can login to VippsPortalen but cannot see the “developer” selection in the top-menu you need to have the administrator grant you developer access for the sales unit you need access to.

# Why do I get “Error: invalid_client”?
This means that the client ID and client secret used is not valid for Vipps login.
 
Please check: 
Are you using the client ID and client secret for the correct environment? There are separate keys for test and production. Both available in VippsPortalen under “Developer”. You can choose the environment above the table with sales units.

Have you set up a redirect URI for login with Vipps in the environment in question?

# What are the requirements for Redirect URIs?
We validate redirect URIs against a white list of pre-approved URIs. The URIs needs to be registered in VippsPortalen, and you can register as many as you want to.
You specify the URI that will be used with the query parameter redirect_uri on the initial request to the authentication endpoint

You can use localhost as part of the redirect URI.

You can use “Custom URL Scheme” in the redirect URIs to redirect back an app.

# Can I change my display name used with Vipps login? 
The name which is displayed in login with Vipps and in the app is the name of the Sale Unit. This can be changed in VippsPortalen (https://portal.vipps.no).

Go to “Salgssteder”, press “rediger” under “salgsstedsinfo” and change to the desired name.

# Which scopes can I use? Why do I get “Invalid_scope”?
If you get “Invalid_scope” this means that you have included one or more scopes that you do not have access to or that is not supported. You will find the list of supported scopes here:
https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#scopes 

All merchants get access to these scopes:
•	openid
•	address
•	birthdate
•	email
•	name
•	phoneNumber

You should not ask for more scopes than you need for your application. The user will need to consent to sharing the information with you so adding more scopes increases the chance that they will decline. 

The scopes “openid” is required and does not require end user consent. It provides the claim “sub” which is a unique id for the end user at that particular merchant. Note: Different merchants will get different subs for the same end user.

Some merchants can get access to NNIN. Merchants need to request this separately.
