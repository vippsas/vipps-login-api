<!-- START_METADATA
---
title: No dialog flow
sidebar_position: 30
---
END_METADATA -->

# No dialog flow

This flow is described [here](../overview.md#no-dialog-flow---log-the-user-in-directly-when-possible). To attempt a no dialog login, add the query parameter `requested_flow=no_dialog` to the [Authorization request](../integration.md#oauth-20-authorize) uri.

If the user completes the login, they will be returned to the `redirect_uri` with a code that can be used to complete the login, just like in a regular [Authorization code](../core-concepts.md#authorization-code-grant) login.

`https://example.com/callback?code=12adsa938721987321asd9873&state=123987218dh`

If the user is not logged in, they will be returned with an error. Some possible errors are `interaction_required`, `login_required` or `server_error`.

Not logged in return uri example: `https://example.com/callback?error=interaction_required&error_description=User+interaction+is+required&state=1312312321983212a3b`.

In all cases, a new login can be started by removing the parameter `requested_flow=no_dialog` and initiating a new login for the user.
