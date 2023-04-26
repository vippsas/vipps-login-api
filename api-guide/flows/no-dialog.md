---
title: No dialog flow
sidebar_label: No dialog flow
sidebar_position: 40
description: Log the user in directly when possible.
pagination_next: null
pagination_prev: null
---

# No dialog flow

This flow can be used to log in the user directly, when the required prerequisites are in place.

When using this flow, a spinner will be shown while Vipps Login tries to log the user in. Once the process is completed, the user will be returned to the merchant, as in the ordinary Vipps Login flow.

For example, after the user clicks "Logg inn" on the front page, the flow might be:

![No dialog flow](../../images/No_dialog_flow.png)

The required prerequisites are:

* The user is remembered in the browser and no consent is required.
* The user is remembered in the browser and consent has previously been given.

If the prerequisites are not in place, the Vipps Login process will be stopped and no interaction will be requested from the user in this flow. In this case, you can display your ordinary login screen, where Vipps is an option. The user can then actively log in with Vipps.

## Implementation details

As with the other Vipps Login flows, it is recommended to run Vipps Login in a redirect mode and iFrame is not supported.

To attempt a no dialog login, add the query parameter `requested_flow=no_dialog` to the [Authorization request](../integration.md#oauth-20-authorize) URI.

If the user completes the login, they will be returned to the `redirect_uri` with a code that can be used to complete the login, just like in a regular [Authorization code](../core-concepts.md#authorization-code-grant) login.

`https://example.com/callback?code=12adsa938721987321asd9873&state=123987218dh`

If the user is not logged in, they will be returned with an error. Some possible errors are `interaction_required`, `login_required` or `server_error`.

Not logged in return uri example: `https://example.com/callback?error=interaction_required&error_description=User+interaction+is+required&state=1312312321983212a3b`.

In all cases, a new login can be started by removing the parameter `requested_flow=no_dialog` and initiating a new login for the user.
