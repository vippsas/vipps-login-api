---
title: Login API frequently asked questions
sidebar_label: FAQ
sidebar_position: 50
description: Login API frequently asked questions
pagination_next: null
pagination_prev: null
---

# Frequently asked questions

Here are the Login API Frequently Asked Questions (FAQ).
See the [Vipps Login API](api-guide/README.md) for all the technical details.

For more common Vipps questions, see:

* [Vipps API General FAQ](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs)

You can also find frequently asked questions in the
[Product FAQ](https://vipps.no/hjelp/vipps/vipps-logg-inn).

## Where do I find the `client_id` and `client_secret`?

See:
[Common topics: API Keys](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/api-keys#getting-the-api-keys).

## How can I activate and set up Vipps Login?

You can activate Vipps Login on the [portal.vipps.no](https://portal.vipps.no).
See [Developer resources: Vipps portal: How to setup login on your sales unit](https://developer.vippsmobilepay.com/docs/vipps-developers/developer-resources/portal#how-to-setup-login-on-your-sales-unit).


## What are the requirements for redirect URIs?

We validate redirect URIs used against a whitelist of
pre-approved URIs. The URIs must be registered by the merchant on
[portal.vipps.no](https://portal.vipps.no)
as described above.

You can register as many URIs as you want to. Specify the URI that will be used with
the query parameter, `redirect_uri`, on the initial request to the `authentication`
endpoint.

**Please note:**

* The redirect URI cannot contain `#`.
* You can use `localhost` as part of the redirect URI.
* You can use “Custom URL Scheme” in the redirect URIs to redirect back an app.
  In this case a path is required: `myapp://path-to-something`
  (not just `myapp://`).

## How can I use `client_secret_post` for authentication?

It is possible to change the token endpoint authentication method on
[portal.vipps.no](https://portal.vipps.no).
This setting will then apply to all login transactions on this sales unit.

Under the "Utvikler" section you will find the Setup Vipps Login option for
your sales units.

Here you have the option to change the token endpoint authentication method,
and see which method is currently active:

![Token endpoint authentication method choice in the portal](images/portal_token_endpoint_authentication_method.png)

## Why do I get `invalid_client`?

This means that the `client_id` and `client_secret` used is not valid for Vipps Login.

Please check:

* Have you activated Vipps Login and set up a redirect URI? See:
  [Developer resources: Portal: How to setup login on your sales unit?](https://developer.vippsmobilepay.com/docs/vipps-developers/developer-resources/portal#how-to-setup-login-on-your-sales-unit).
* Have you double checked that the `redirect_uri` used in the API call is
  _exactly_ the same as the one specified on
  [portal.vipps.no](https://portal.vipps.no)?
* Pay extra attention to whether the URI used in the API request has a
  trailing `/` or URL-encoded entities (like `%20`), and whether the URI added on
  [portal.vipps.no](https://portal.vipps.no)
  is an exact match.
* Are you using the `client_id` and `client_secret` for the correct environment?
  There are separate API keys for test and production. See:
  [Common topics: API Keys](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/api-keys).

## Why do I get `invalid_grant`?

The most common reason is that the authorization code has expired.
The code is short-lived, so it will only be valid for a couple of minutes.

From
[RFC 6749](https://www.rfc-editor.org/rfc/rfc6749#section-5.2):

> The provided authorization grant (e.g., authorization code, resource owner credentials)
> or refresh token is invalid, expired, revoked, does not match the redirection URI used
> in the authorization request, or was issued to another client.

## Which scopes can I use? Why do I get `invalid_scope`?

If you get `Invalid_scope` this means that you have included one or more scopes
that you do not have access to or that is not supported.
See:
[Scopes](api-guide/core-concepts.md#scopes).

All merchants have access to these scopes:

* openid
* address
* birthdate
* email
* name
* phoneNumber

**Important:** You should never ask for more scopes than you need for your
application. The user will need to consent to sharing the information with you
so adding more scopes increases the chance that they will decline.

The scope `openid` is required and does not require user consent. It
provides the claim `sub` which is a unique id for the end user at that
particular merchant.

**Please note:** Different sales units will get different `sub`s for the
same end user.

Some merchants can get access to national identity number (NIN) with the `nin`
scope. Merchants need to request this separately. See
[Who can get access to NIN and how?](#who-can-get-access-to-nin-and-how).

You can find the list of scopes that your individual sales units have access to in
[portal.vipps.no](https://portal.vipps.no)
under the _Utvikler_ section and the _Setup Vipps Login_ panel.

## Why do I get `Error: Could not get Vipps Login token” in Vipps`?

You can get this error if you have both the Vipps
[test app](https://developer.vippsmobilepay.com/docs/vipps-developers/test-environment#vipps-test-apps)
and production app on the same phone.

## What is the `sub`?

See:
[Userinfo API FAQ: What is the `sub`](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api/userinfo-api-faq/#what-is-the-sub)
for this and more information about the `sub`.

## How can I get updated information, like addresses, for a user?

When the user consents to sharing information with the merchant, the merchant
has ~10 minutes to retrieve the information.
The information can be retrieved every time the user logs in.
The merchant must save this information and handle everything according to GDPR.  

## Why can I get userinfo after the user has revoked consent?

During a login or a payment session the user consent to share information if
it's requested by the merchant. The users information is then available for
the merchant from the userinfo endpoint. For login sessions, user information
is available for the ongoing login session.

To better support merchants that do not handle online fetching and processing
of the user info as part of a payment session, we keep this information accessible
for the merchant for the next **168 hours**, even though the user revokes the
consent in this period. Revoking consents will immediately affect future login
and payment sessions.

See:
[Userinfo API](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api).

## Can a Vipps user less than 15 years old use Vipps Login?

No, Vipps Login requires a full Vipps profile. Users below the age of 15 can
not use Vipps Login. See the
[help pages for Vipps for those under 15 years](https://vipps.no/hjelp/vipps/under-15/hvordan-oppretter-jeg-vipps-for-de-under-15-år/).

## Who can get access to NIN and how?

Access to the `nin` scope and the National Identity Number (NIN) is a paid service.
Merchants will be billed for each successful login where they request a user's NIN.

Only merchants with legal requirements or other objective needs to use the Norwegian
National Identity Number (NIN) to achieve required user identification can get
access to NIN. We follow the
[guidance from Datatilsynet](https://www.datatilsynet.no/rettigheter-og-plikter/personopplysninger/fodselsnummer/).

According to the above NIN can only be used when it is legally required or when
there is a need for a secure identification of the person and the NIN is
required to achieve this. This applies e.g. when a company is required to
report to the tax authorities or within healthcare and credit.

Keeping a unique and consistent identifier for the user over time is not a
sufficient requirement. For this purpose Vipps offers a unique merchant specific
user identifier that are delivered as part of the registration/login. This is
the claim `sub` that is delivered based on the `openid` scope. This unique
identifier will allow you to keep a consistent user profile even if the user
changes contact information.

Be aware that Vipps Login it not an electronic ID. Thus, the NIN can only be
used to simplify the customers processes by removing manual input or to look up
the customer in your own or external registers. This can be done as part of the
processes to become a customer or to link login with Vipps to an existing user.
If you need to store the NIN for new users we recommend that you use an
electronic ID.

Merchants need to apply for access to NIN by sending an email to
[accessuserinfo@vipps.no](mailto:accessuserinfo@vipps.no).
In the email you must specify:

* Organization number
* Merchant name
* Name and number of the sales unit from [portal.vipps.no](https://portal.vipps.no)
* Information on how you plan to use the NIN
* The legal requirement and/or the reason why you need to use NIN to achieve
  required user identification.

## Who can get access to Vipps Login from phone number and how?

[Vipps Login from phone number](api-guide/overview.md#vipps-login-from-phone-number)
is available for all Vipps Login enabled clients.

_Vipps Login from phone number_ is a paid service.

Remember that _Vipps Login from phone number_ has been developed to support use cases where
authentication/registration does not start in a browser or an app. This means
that it is the merchant that triggers the authentication/registration (either
manually or automatically) and thus the log-in cannot be done in the user’s browser.

_Vipps Login from phone number_ is reserved for such special cases. If a merchant
uses this incorrectly on webpages or in apps used by end users
(on their own device), access to the feature can be withdrawn.

Merchants need to apply for access to _Vipps Login from phone number_
by sending an email to
[accessuserinfo@vipps.no](mailto:accessuserinfo@vipps.no).
In the email you must specify:

 * Recipient name
 * Recipient email
 * Invoicing address

## How can I get started with delegatedConsents?

**Please note:** _Delegated consents_ are only available for the
[Vipps Login from phone number](./how-it-works/vipps-login-from-phone-number-api-howitworks.md)
flow. It is currently not possible to request delegated consents in the
_Vipps Login in browser_ flow.

If you plan on using the scope `delegatedConsents` along with _Vipps Login from phone number_,
you will need to supply some information on what consents you will retrieve and how.
We will then tailor this screen to suit your needs. You can see what the flow looks like at
[Vipps Login from phone number](./how-it-works/vipps-login-from-phone-number-api-howitworks.md).

Please send an email to
[accessuserinfo@vipps.no](mailto:accessuserinfo@vipps.no)
and supply the following information:

| Item                                                  | Description                                                                                                                                                                     | Example                                                                           | Comments                                                                                                                                  |
|-------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| Top text                                              | Specifies the purpose of collecting the consents                                                                                                                                | "Merchant AS would like to send you tailored offers in several digital channels." |                                                                                                                                           |
| Subset of consents you would like to request.         | We offer a set of predefined consents. See [When using delegatedConsents, which consents does Vipps support?](#when-using-delegatedconsents-which-consents-are-supported). | email, sms, digital, personal                                                     | If you want a consent type that we currently don't support, reach out to us at [accessuserinfo@vipps.no](mailto:accessuserinfo@vipps.no). |
| Links to your membership terms and privacy statement. | We must review your terms to ensure that the flow is not intended to mislead or abuse end users.                                                                                | www.merchant.com/termsandconditions <br /> www.merchant.com/privacystatement      |                                                                                                                                           |
| Recipient to sign DPA                                 | Vipps will function as a data processor and not have any ownership to the data involved. For more information, please visit merchant terms and conditions                       | recipient@merchant.com                                                            |                                                                                                                                           |

## When using `delegatedConsents`, which consents are supported?

Vipps Login has support for several consents, and you may select a subset
of these to include in your own flow, as well as select which of the consents
that are required/optional. If the consents you need are not currently supported,
you can make a request to
[accessuserinfo@vipps.no](mailto:accessuserinfo@vipps.no)
and we'll be in touch.

| Id       | Consent text (Norwegian)            | Consent text (English)                    |
|----------|-------------------------------------|-------------------------------------------|
| email    | Få tilbud på E-post                 | Receive offers via email                  |
| sms      | Få tilbud på SMS                    | Receive offers via SMS                    |
| digital  | Jeg vil motta digital markedsføring | I would like to receive digital marketing |
| personal | Få tilpassede tilbud                | Get customized offers                     |

See
[Merchant delegated consents](api-guide/flows/phone-number-ciba-flows.md#merchants-delegated-consents)
for details.

## What's the purpose of the `state` parameter?

The `state` parameter is an opaque value used by the client to maintain state
between the request and callback. The authorization server includes this value
when redirecting the user-agent back to the client. The parameter should be
used for preventing cross-site request forgery. It must be at least 8 characters long
to ensure sufficient entropy. A GUID is a good choice for a state parameter value.

## Can I use partner keys for Vipps login?

Yes, partner keys can be used for both _Vipps Login in Browser_ and _Vipps Login from Phone number_.
Be aware that the partner key integration
[differs slightly from a regular merchant integration.](api-guide/partner-keys.md).

Vipps Login supports some scenarios where a merchant can be used for registration and
login to other sites/merchants. Specific terms and conditions related to
UX/branding, consent, terms, and privacy statement apply to such scenarios.
These can be found in section 8.4 in the
[Terms and Conditions for merchants](https://www.vipps.no/vilkar/vilkar-bedrift/).

## Can I control if a user is remembered in the browser (set up as 2FA)?

No. Vipps Login does not support merchants specifying that the user needs
to use the app to authenticate (two-factor authentication - 2FA). The end user
chooses whether or not to be remembered in browser. This is seen
as a key feature of the service.

## Can I see statistics on Vipps Login usage?

Yes, statistics on successful logins with Vipps in production is available on
the
[API Dashboard](https://developer.vippsmobilepay.com/docs/vipps-developers/developer-resources/api-dashboard).

Choose the sales unit with Vipps Login that you would like to see, the time
period you would like to have covered and click "Fetch data".
Then scroll down until you find the panel named "Successful logins".

![The dashboard looks like this:](images/API_Dashboard_Vipps_login.png)

If you have access to request national identity number (NIN) you will be able
to see the number of logins with and without these
scopes. If you click _Show details_ under the graph, you can see total logins
for the period and the number of unique users in the selected period.

Keep in mind that there may be up to 10 minutes delay from whe a login occurs
until it appears on the dashboard.

**Please note:** The statistics are only available for the production environment.

## How is GDPR handled with Vipps Login?

With regard to the processing of personal data and GDPR, the following applies
to Vipps Login:

1. Vipps Login gives a merchant the opportunity to ask a Vipps end user to share
   a selection of data from their profile in Vipps. This can include name, phone
   number, email, addresses, and date of birth. The merchant controls which of
   this data is requested. The user must consent to the sharing of data.
   The consent is the legal basis for the Vipps AS transfer of
   this information to the merchant.
2. Vipps AS is responsible for our processing of information related to Vipps
   end users and the personal information generated using the Vipps services.
   For the Vipps Login service, the merchant will be responsible for the
   processing of the profile information received, starting when the merchant
   receives the profile information from Vipps end user. The merchant will thus
   be an independent data processor for this data, and there is no need for a
   data processing agreement between Vipps and the merchant.
3. The merchant must therefore obtain a valid basis for further processing of
   the personal data (e.g agreement, terms or consent), to e.g. register the
   information in its customer register and start customer processing from there.
4. When such sharing from Vipps AS to the merchant has been made, a Vipps end
   user can later use Vipps to log in to the merchant, and the merchant will
   then have access to updated information on the data elements that the company
   has requested. A Vipps end user can go into Vipps (the app) and see which
   companies they have shared data with, which data has been shared and they
   can withdraw their consent to share. This means that new consent must be
   obtained before Vipps AS can share data again with the merchant.
5. When an end user uses Vipps Login at a merchant, Vipps AS stores, as part
   of our service to the Vipps end user and with Vipps AS as data processor,
   information about a) what information a user has agreed to share with a
   merchant and b) when a user has used Vipps log in to the relevant merchant.
6. Vipps AS does not receive any information from the merchant about Vipps end user.

See more information in our end user
[terms and condition](https://www.vipps.no/vilkar/vilkar-privat/),
[privacy policy](https://www.vipps.no/vilkar/cookie-og-personvern/)
and
[merchant agreement](https://www.vipps.no/vilkar/vilkar-bedrift/).

## Can we control the language displayed to the user?

No. The language is controlled by the browser settings.

Specifically, language is controlled by `window.navigator.language`; however, it gets more complicated
since there are fallbacks. Refer to the documentation for your browsers.

## Which configuration should I have when integrating using Azure B2C?

Azure B2C overrides the `redirect_uri` parameter to redirect to Azure B2C first, then to your redirect URI.
You will need to look at your call to `https://apitest.vipps.no/access-management-1.0/access/oauth2/auth`
and find the `redirect_uri` query parameter. This will need to be whitelisted in
[portal.vipps.no](https://portal.vipps.no).

See [What are the requirements for Redirect URIs?](#what-are-the-requirements-for-redirect-uris)

Azure B2C uses `client_secret_post` as `token_endpoint_auth_method` and the default value is
`client_secret_basic`, so you'll need to change this in the [portal.vipps.no](https://portal.vipps.no).

See [How can I use `client_secret_post` for authentication?](#how-can-i-use-client_secret_post-for-authentication)

Vipps Login does not return user information in the `id_token`, but there is a
[userinfo endpoint](https://developer.vippsmobilepay.com/api/userinfo#operation/userinfoAuthorizationCode)
 for this use case. See
[the Login API userinfo endpoint documentation](api-guide/integration.md#userinfo).
Azure B2C's User Flows does not use the userinfo endpoint and you will therefore need
to use a [Custom policy](https://docs.microsoft.com/en-us/azure/active-directory-b2c/userinfo-endpoint?pivots=b2c-custom-policy).

## How can we detect users' consent removal?

Or: _How can our system dynamically "know/find out" if the user has revoked the consent
for us to have access to his/her personal data in our system?_

Your system can dynamically detect when a user's consent has been revoked by using _consent webhooks_.
This is a system for notifying merchants when an end user revokes their consent.
See the
[Consent webhooks](api-guide/important-information.md#revoke-consent-webhook) section for more information.

## Can we have multiple URIs as landing pages?

Yes. You can register as many callback URLs as you want; and then you specify which
one you use in the request to
[/auth](https://developer.vippsmobilepay.com/api/login#tag/Vipps-Login-API/operation/oauthAuth).

## Can I use a custom URL scheme for the `redirect_url`?

Yes. If the `redirect_url` is using a custom URL scheme, such as `myapp://`, a path
is required: `myapp://path-to-something`.

## Can we change the name that appears in customer's Vipps app under `Login and Access`?

The name which is displayed in the app is the name of the sales unit.
You can change it yourself on
[portal.vipps.no](https://portal.vipps.no/).
Press  _rediger_ (_edit_) under _salgsstedsinfo_ (_sales unit information_) and change to the desired name.

## How can I log a user out?

_Vipps Login_ does not support merchant-initiated log-out in the browser. as this
would effectively log the user out of Vipps Login (meaning that the user will no
longer be remembered in the browser across sites that use Vipps Login). You
are of course free to log the user out of your service (by disabling your own session).

If a user wants to log out of a specific browser remembered in Vipps Login, they need to do this in the Vipps app by navigating to:
_Profile_ > _Personal Information_ > _Browsers that remember you_, select a browser, and press the _logout_ button.

## Common errors

See
[FAQ: Common errors](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/common-errors-faq)
for more questions.

### Why do I get `The client or its related sales unit or merchant does not exist or is not active`?

This means that the API credentials are for a MSN (Merchant Serial Number)
that does not exist, or is not active. This can happen if the organization number
has been deactivated at [Brønnøysundregistrene](https://www.brreg.no).

### Why do I get a CORS error?

We do not currently support any flows that requires requests being made from browsers.

See:
[Common problems: Why do I get a CORS error?](https://developer.vippsmobilepay.com/docs/vipps-developers/faqs/common-problems-faq#why-do-i-get-a-cors-error).

### Why do I get `No CSRF value available in the session cookie`?

Certain versions of Chrome gives the error `No+CSRF+value+available+in+the+session+cookie`.
Upgrading to the latest version of Chrome should solve this.

### Why do I get `HTTP 502 Bad Gateway`?

Some merchants have experienced a 502 Bad gateway response from [api.vipps.no](https://api.vipps.no).
This typically occurs in situations in which the `state` or `nonce` parameter
is 1000+ characters. We've seen this issue when any of the requests in the
redirect sequence are too long (i.e 2000+ characters). Therefore, try to keep
these parameters at sane lengths. If there is a need to encode some payload in
the `state` (i.e a JWT), it would be a better option to cache this at the
client server and use the key as `state`.
