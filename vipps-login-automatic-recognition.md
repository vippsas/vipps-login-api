:warning: This feature is still under development, and not available at the time of writing.

---

Vipps Login will offer the option to authenticate end-users automatically when they return to a merchant's site. This gives merchants the opportunity to provide custom content to the user directly without the need for manual authentication. A precondition is that the user has consented to use Vipps login at the merchant's site, and has opted into being remembered in the browser being used.

### Automatic authentication in an iframe
There are several ways of performing automatic authentications. For the best user experience, we recommend a silent authentication in an iframe.
Initiate an authentication in a hidden iframe, [GET /oauth2/auth](https://vippsas.github.io/vipps-login-api/#/public/oauthAuth) *(Note: Iframe must not have `display: none`. Instead, set `height: 0`, `width: 0`)*.
Depending on the implementation on a merchant site, a successful authentication may invoke a reload in the browser. To reduce the impact of such reload during the users browsing session, it is preferable to initiate the silent authentication as soon as possible after the site is loaded.

See the [Vipps Login API](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md) documentation for details about the OIDC protocol used to integrate with Vipps Login, especially the [OAuth 2.0 Authorize](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api.md#oauth-20-authorize) chapter.

**window.postMessage()**  
To avoid a reload upon successful authentication, `window.postMessage()` can be used from within the iframe to notify and eventually send data to the parent window. When receiving the postMessage event, the parent window can log in the user, without disrupting the browsing experience.
Example of usage of postMessage. *(Node.js)*

```javascript
// server-side
router.get('/callback', (req, res) => {
  const code = req.query['code'];
  getToken(code)
    .then(getUserinfo)
    .then((data) => {
      res.send(`<script> window.parent.postMessage('${JSON.stringify(data)}', '*'); </script>`);
    });
});

// client side
document.addEventListener('message', receivePostMessage);
```

*Read more about [window.postMessage()](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage).*

#### Request
**login_hint=unsolicited:nodialog**  
In order to perform automatic authentication, the authentication must be initiated with the query param`login_hint=unsolicited:nodialog.  
This will initate a login session without any dialoges.

Note: The only way to load Vipps Login successfully in an iframe is by using `login_hint=unsolicited:nodialog`!


#### Response
**Successful**  
Similar to a normal authentication, an automatic authentication will end with a redirect to the provided redirect URI. After a successful authentication, the *callback URI* will be called with *code* and *state* query parameters.

**Error**  
A silent authentication will redirect back to the callback URI if an error occurs. Conditions leading to an error include:
* User interaction needed
* authentication required
* consent required
* Other error situations

Note: Under any circumstances where direct user interaction is needed, the automatic authentication will return an error, with these elements as query parameters on the callback:
- `state=<state given on the auth call>`
- `error=access_denied`
- `error_description=User+cancelled+the+login`

### Session handling
Merchants using the automatic authentication option should implement this feature across the website to support such login on all pages where a user can enter the website. The user should be logged in on the first page of the visit using Vipps login and then the merchant should keep the user logged in during the visit using a session cookie. This session should expire at the end of the user's visit to ensure that authentication of the user is done through Vipps login and not a cookie from the merchant on the next visit. This improves the security of the authentication and ensures that possible changes to the user's consents and profile information are taken into account on the next visit.

### Content Security Policy
Beware that some security policies, like `frame-src`, may prevent the content of the iframe to load. Merchants using such policies must ensure that they're whitelisting the Vipps Login domain.
Example.
```
Content-Security-Policy: frame-src https://api.vipps.no
```
*This will only allow Vipps Login to be loaded in an iframe.*
```html
<iframe src="https://api.vipps.no/.../oauth2/auth"></iframe>
```
