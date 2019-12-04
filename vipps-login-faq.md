# Frequently Asked Questions for Vipps Login API

See the [Vipps Login API](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md) for all the details.

See also the
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide for the Vipps Developer Portal.
You can also find frequently asked questions on the product here [Product FAQ](https://vipps.no/hjelp/vipps/vipps-logg-inn).

# Table of contents

- [Where do I find the `client_id` and `client_secret`?](#where-do-i-find-the-client_id-and-client_secret)
- [How can I activate and set up Vipps login?](#how-can-i-activate-and-set-up-vipps-login)
- [What are the requirements for Redirect URIs?](#what-are-the-requirements-for-redirect-uris)
- [Why do I get “Error: invalid_client”?](#why-do-i-get-error-invalid_client)
- [Can I change my display name used with Vipps login?](#can-i-change-my-display-name-used-with-vipps-login)
- [Which scopes can I use? Why do I get “Invalid_scope”?](#which-scopes-can-i-use-why-do-i-get-invalid-scope)
- [Who can get access to NNIN?](#who-can-get-access-to-nnin)

# Where do I find the `client_id` and `client_secret`?

You find more information in
[Getting started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#getting-the-api-keys).

Be aware that you do _not_ use the API-keys for Vipps login, just the `client_id`
and `client_secret`.

If you cannot access [portal.vipps.no](https://portal.vipps.no): do you have a
user for the merchant? The person who were added as the contact person in the
request is set as the administrator and can add new users. If you have added a
new service to an existing merchant, you need to have the existing administrator
add new users if required.

If you can login to [portal.vipps.no](https://portal.vipps.no) but cannot see
the “developer” selection in the top-menu you need to have the administrator
grant you developer access for the sales unit you need access to.

# How can I activate and set up Vipps login?

On [portal.vipps.no](https://portal.vipps.no), in the same place as the client
ID and `client_secret` described above, you can activate Vipps login, and also
add the redirect URIs needed for the service to work. This is the URL/URI of
the page which the user is redirected to after finishing a login. See more on
this in the
[documentation](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#api-endpoints)
as well as requirements for the URIs below.

You will find the Setup Vipps login option in the same place as you find your
`client_id` and `client_secret` (click "setup login" to get started):

![You will find the Setup Vipps login option in the same place as you find your `client_id` and `client_secret`](images/portal_setup_login.png)

First you activate Vipps login:
![First you activate Vipps login](images/portal_setup.jpeg)

Then you can add the redirect URIs you need:
![Then you can add the redirect URIs you need](images/portal_direct_uris.jpeg)

# What are the requirements for Redirect URIs?

We validate redirect URIs against a white list of pre-approved URIs. The URIs
need to be registered on [portal.vipps.no](https://portal.vipps.no), and you
can register as many as you want to. You specify the URI that will be used with
the query parameter `redirect_uri` on the initial request to the authentication
endpoint.

You can use localhost as part of the redirect URI.

You can use “Custom URL Scheme” in the redirect URIs to redirect back an app.

# Why do I get “Error: invalid_client”?
This means that the `client_id` and `client_secret` used is not valid for Vipps login.

Please check:
Are you using the `client_id` and `client_secret` for the correct environment?
There are separate keys for test and production. Both available on
[portal.vipps.no](https://portal.vipps.no) under “Developer”. You can choose
the environment above the table with sales units.

Have you set up a redirect URI for login with Vipps in the environment in question?

# Can I change my display name used with Vipps login?

The name which is displayed in login with Vipps and in the app is the name of
the Sale Unit. This can be changed on [portal.vipps.no](https://portal.vipps.no).

Go to “Salgssteder”, press “rediger” under “salgsstedsinfo” and change to the
desired name.

# Which scopes can I use? Why do I get “Invalid_scope”?

If you get “Invalid_scope” this means that you have included one or more scopes
that you do not have access to or that is not supported. You will find the list
of supported scopes here:
https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#scopes

All merchants get access to these scopes:
•	openid
•	address
•	birthdate
•	email
•	name
•	phoneNumber

**Important:** You should not ask for more scopes than you need for your
application. The user will need to consent to sharing the information with you
so adding more scopes increases the chance that they will decline.

The scopes “openid” is required and does not require end user consent. It
provides the claim “sub” which is a unique id for the end user at that
particular merchant. Note: Different merchants will get different subs for the same end user.

Some merchants can get access to NNIN. Merchants need to request this separately.

# Who can get access to NNIN?

Merchants with legal requirements or other objective needs to use Norwegian
National Identity Number (NNIN) to achieve required user identification can get
access to NNIN.  We correspond to
[the guidance from Datatilsynet](https://www.datatilsynet.no/rettigheter-og-plikter/personopplysninger/fodselsnummer/) on this.

According to this NNIN can only be used when it is legally required or when
there is a need for a secure identification of the person and the NNIN is
required to achieve this. This applies e.g. when a company is required to
report to the tax authorities or within healthcare and credit.

Keeping a unique and consistent identifier for the user over time is not a
sufficient requirement. For this purpose Vipps offers a unique merchant specific
user identifier that are delivered as part of the registration/login. This is
the claim "sub" that is delivered based on the "openid" scope. This unique
identifier will allow you to keep a consistent user profile even if the user
changes contact information.

Beware that login with Vipps it not an electronic ID. Thus the NNIN can only be
used to simplify the customers processes by removing manual input or to lookup
the customer in your own or external registers. This can be done as part of the
processes to become a customer or to link login with Vipps to an existing user.
If you need to store the NNIN for new users we recommend that you use an
electronic ID, e.g  BankID.
