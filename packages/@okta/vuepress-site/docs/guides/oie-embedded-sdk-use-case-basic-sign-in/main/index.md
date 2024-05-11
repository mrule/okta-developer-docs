---
title: Basic sign-in flow with the password factor
excerpt: Enable a password-only sign-in flow in your web app using the embedded SDK
layout: Guides
---

<ApiLifecycle access="ie" />


> **Note**: Passwords are a security vulnerability because they can be easily stolen and are prone to phishing attacks. Give your users the ability to use other authenticators by replacing password-only sign-in experiences with either a [password-optional](https://developer.okta.com/docs/guides/pwd-optional-overview) or a multifactor experience.
> > To learn about a sign-in use case where the password is optional, see the [Sign in with email only](https://developer.okta.com/docs/guides/pwd-optional-sign-in-email/nodeexpress/main/) guide.
<StackSnippet snippet="pwdoptionalusecase" />

---

### Learning outcomes

Add a sign-in flow to a server-side web app that requires only a password.

### What you need
* An app that uses the embedded [Okta Auth JavaScript SDK](https://github.com/okta/okta-auth-js)
* [Identity Engine SDK set up for your own app](https://developer.okta.com/docs/guides/oie-embedded-common-download-setup-app/nodejs/main/)

### Sample code

[Node.js Identity Engine sample app](https://github.com/okta/okta-auth-js/tree/master/samples/generated/express-embedded-auth-with-sdk)

---

## Configuration updates

To configure your app so it requires only a password, see [Set up your Okta org for a password factor-only use case](https://developer.okta.com/docs/guides/oie-embedded-common-org-setup/nodejs/main/#set-up-your-okta-org-for-a-password-factor-only-use-case).

## Summary of steps

The following diagram summarizes the steps involved in a sign-in flow using only a password:

![oie-embedded-nodejs-sign-in-pwd-only-flow-diagram](https://github.com/okta/okta-developer-docs/assets/66759469/f9b9f7a8-9cb5-4a0e-9f51-d3d9d817c1f0)

## Integration steps

### 1: Your app displays the sign-in page

Build a sign-in page that captures both the user's name and their password.

<img width="401" alt="sign-in-form-username-password" src="https://github.com/okta/okta-developer-docs/assets/66759469/ca49cd2b-45af-4f4d-96b2-79b35f793b84">

### 2: The user submits their username and password

When the user submits their `username` and `password`, pass them as parameters to `OktaAuth.idx.authenticate()`.

```
const authClient = getAuthClient(req);
const transaction = await authClient.idx.authenticate({ username, password });
```

### 3: Your app handles an authentication success response

`authenticate()` returns a `transaction` object with a `status` property indicating the current state of the sign-in flow. Handle the returned `IdxStatus` value accordingly:

#### Success status

When the user correctly supplies their password, `IdxStatus` equals `IdxStatus.SUCCESS`. Call `tokenManager.setTokens()` to save the tokens retrieved from the response for future requests, and then redirect the user back to the home page. The user is now signed in.

```
 const { nextStep, tokens, status, error, } = transaction;
  // Persist states to session
  req.setFlowStates({ idx: transaction });

  switch (status) {
    case IdxStatus.SUCCESS:
      // Save tokens to storage (req.session)
      authClient.tokenManager.setTokens(tokens);
      // Redirect back to home page
      res.redirect('/');
      return;

   // Handle other statuses
}
```

#### Other authentication statuses 

Handle other returned `IdxStatus` cases if the user didn't sign in successfully or there are other factors to verify. For example:

```
switch (status) {
   case IdxStatus.SUCCESS:
      // handle success
   return;
   case IdxStatus.PENDING:
      // Proceed to next step
      try {
         if (!proceed({ req, res, nextStep })) {
            next(new Error(`
            Oops! The current flow cannot support the policy configuration in your org.
            `));
         }
      } catch (err) {
         next(err);
      }
   return;
   case IdxStatus.FAILURE:
      authClient.transactionManager.clear();
      next(error);
   return;
   case IdxStatus.TERMINAL:
      redirect({ req, res, path: '/terminal' });
   return;
   case IdxStatus.CANCELED:
      res.redirect('/');
   return;
}
```

### 4: Get the user profile informtaion

After the user signs in successfully, request basic user information from the authorization server using the tokens that were returned in the previous step.

```
module.exports = async function userContext(req, res, next) {
  const authClient = getAuthClient(req);
  const { idToken, accessToken, refreshToken } = authClient.tokenManager.getTokensSync();
  if (idToken && accessToken) {
    const userinfo = await authClient.token.getUserInfo(accessToken, idToken);
    req.userContext = {
      userinfo,
      tokens: {
        idToken, accessToken, refreshToken
      }
    };
  }

  next();
};
```
