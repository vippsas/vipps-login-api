# Vipps Login integrations

#Auth0

Add a new Enterprise Connection: Open ID Connect

Issuer URL can be found as the "issuer" field in the openid-configuration. See https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#openid-connect-discovery-endpoint.

Type: Back Channel

Client ID and Client Secret: See https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#api-key-details

Scopes: See https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#scopes, not "profile" is not a supported scope

The client must be configured in the Vipps portal to use the correct configuration https://github.com/vippsas/vipps-login-api/blob/f5246c1365b17ad56561d509acb90cf9af37cd48/vipps-login-api-faq.md#how-can-i-use-client_secret_post-for-authentication

