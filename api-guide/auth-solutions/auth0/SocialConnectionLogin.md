<!-- START_METADATA
---
title: Login in Auth0 using Social Connections
sidebar_label: Login using Social Connections
sidebar_position: 191
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Login in Auth0 using Social Connections

This guide will describe how to use Auth0's actions to automatically create and login users using Vipps MobilePay.
The following implementation will provide the same as the
[user initiated login from a website](../../../how-it-works/user-initiated-login-howitworks.md#the-login-process) flow.

With Auth0's Social Connections, merchants can implement a login flow using Vipps MobilePay as an identity provider.

## Prerequisites

* [Create an Auth0 account](https://auth0.com/signup)
* Configure an Application:

  * Under *Applications*, select *Create Application* and choose your preferred type of application. For this guide, we will select a Single Page Application.
  * Copy the *Domain* for use in later steps.
  * Under *Allowed Callback URLs*, specify where a user will be returned after logging in.

* Create a test unit in the [merchant portal](https://developer.vippsmobilepay.com/docs/developer-resources/portal/).

  * Save the `client_id` and `client_secret` for use in later steps.
  * Set the Token endpoint authentication method to `client_secret_post`.
  * Add the following redirect URI to the list of callback URIs and replace `yourdomainname` with the domain name recorded previously.

  ```bash
  https://yourdomainname/login/callback
  ```

## Configure a Social Connection

Log in to the Auth0 portal. Under *Authentication*, select *Social* and click on *Create Social Connection*. In the URLs you must change `<Vipps environment>` to the environment you are using. This could be either `api.vipps.no` (Prod) or `apitest.vipps.no` (Test). To set up Vipps login, fill in the following fields:

* **Name**: Name of your connection, for example, "VippsLogin".

* **Authorization URL**: `https://{{Vipps environment}}/access-management-1.0/access/oauth2/auth>`

* **Token URL**: `https://{{Vipps environment}}/access-management-1.0/access/oauth2/token`

* **Scope**: The scope defines the information you are requesting from the users. The `openid` scope must be specified, but additional
[scope](../../core-concepts.md#scopes) parameters can be provided.
The scope parameters will be used to decide which user attributes are used to create an Auth0 user.

  If multiple scopes are provided, separate them with spaces and select *Separate scopes using a space*.
For example, enter `openid name phoneNumber email`, request name, phone number, and the email of the user.

* **Client ID** Enter your `client_id` recorded earlier.

* **Client Secret** Enter your `client_secret` recorded earlier.

* **Fetch User profile script**: Here you can enter JavaScript code to fetch data from the API and store it to an Auth0 user. See [Fetch user profile script](#fetch-user-profile-script) below for an example of how it can be implemented.

### Fetch user profile script

An example script is provided below. You must change `<Vipps environment>` to the environment you're using. This could be either `api.vipps.no` (Prod) or `apitest.vipps.no` (Test).

```js
function fetchUserProfile(accessToken, context, callback) {
  request.get(
    {
      url: "https://<Vipps environment>/vipps-userinfo-api/userinfo",
      headers: {
        Authorization: "Bearer " + accessToken,
      },
    },
    (err, resp, body) => {
      if (err) {
        return callback(err);
      }
      if (resp.statusCode !== 200) {
        return callback(new Error(body));
      }
      let bodyParsed;
      try {
        bodyParsed = JSON.parse(body);
      } catch (jsonError) {
        return callback(new Error(body));
      }
      const profile = {
        user_id: bodyParsed.sub,
        email: bodyParsed.email,
        phone_number: "+" + bodyParsed.phone_number,
        name: bodyParsed.name,
      };
      callback(null, profile);
    }
  );
}
```

In the body of `profile`, you can specify the [scope](../../core-concepts.md#scopes) to define data will be required of the user. You can also specify what it will be mapped to in Auth0 as well as how it should be formatted.

Values

* `url` - This is the URL to the endpoint used for getting user information from the [Userinfo API](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api/).
* `headers` - The user info API uses Bearer Token Authentication. For more information, see the [Userinfo API](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api/).
* `profile` - Which attributes will be collected and how they will be used to create an Auth0 user. The `user_id` parameter is used to create a unique identifier for the signed-up user. This should be created from the `sub` which is a unique identifier corresponding to a Vipps user. For more information about Vipps's unique identifier see [What is the sub?](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api/userinfo-api-faq/#what-is-the-sub)

## Add a Social Connection to your Application

Under *Applications*, click on *Connections*. Make sure that your new *Social Connection* is toggled.

## Test your connection

Under *Authentication* -> *Social*, select your newly created *Social Connection*. Click *Try Connection*. You should be redirected to the Vipps login page.

To check if a user has been created correctly, go to *User Management* -> *Users*. There should be a new user in the Auth0 database. To check if the attributes have been collected, click on the user and select *Raw JSON*. It should look something like this:

```json
{
  "created_at": "2023-07-04T08:16:58.209Z",
  "email": "ola.nordmann@vipps.no",
  "identities": [
    {
      "provider": "oauth2",
      "user_id": "VippsLogin|{sub}",
      "connection": "VippsLogin",
      "isSocial": true
    }
  ],
  "name": "Ola Nordmann",
  "nickname": "ola.nordmann",
  "phone_number": "+4712345678",
  "picture": "https://s.gravatar.com/avatar/{...}.png",
  "updated_at": "2023-07-04T10:57:43.939Z",
  "user_id": "oauth2|VippsLogin|{sub}",
  "last_ip": "77.40.174.194",
  "last_login": "2023-07-04T10:57:43.939Z",
  "logins_count": 2,
  "blocked_for": [],
  "guardian_authenticators": []
}
```

## References

Create a custom Social Connection with Vipps as an identity provider.

* [Connect Apps to Generic OAuth2 Authorization Servers](https://auth0.com/docs/authenticate/identity-providers/social-identity-providers/oauth2)

Terminology:

* [Common terms](https://developer.vippsmobilepay.com/docs/knowledge-base/terminology/#common-terms)
