# Usage

1. `npm init`
2. `npm install express --save`
3. `npm install webtask-tools --save`
4. Create `index.js` with the minimal express extension:

  ```js
  var express = require('express');
  var Webtask = require('webtask-tools');

  var app = express();

  app.get('/', function (req, res) {
    res.status(200).send('Hello World');
  });

  if ((process.env.NODE_ENV || 'development') === 'development') {
    app.listen(3000);
  } else {
    module.exports = Webtask.fromExpress(app);
  }
  ```

## Authenticating the extension with Auth0

A useful extension would typically use the Auth0 API. We created a module that abstract away the OAuth2 consent flow with the Auth0 API.

1. `npm install auth0-oauth2-express --save`
2. Add the `auth0-oauth2-express` module and configure the scopes you want. This will ask the right consent to the user.

  ```js
  var auth0   = require('auth0-oauth2-express');

  app.use(auth0({
    scopes: 'read:connections'
  }));
  ```

## Adding server side logic on the webtask

Auth0 Extensions would be typically be client side but if you want to have server side logic you can create endpoints on `index.js` and we would provide by default JWT authentication so all the calls run under the user context.

### Configuring JWT generation

```js
app.use(auth0({
  scopes: 'read:connections',
  apiToken: {
    payload: function (req, res, next) {
      // Add extra info to the API token
      req.userInfo.MoreInfo = "More Info";
      next();
    },
    secret: function (req) {
      return req.webtaskContext.data.EXTENSION_SECRET;
    }
  }
}));
```

### Configuring JWT validation

```js
api.use(jwtExpress({
  secret: function(req, payload, done) {
    done(null, req.webtaskContext.data.TOKEN_SECRET);
  }
}));
```

### Adding a secured endpoint

```js
api.get('/secured', function (req, res) {
  if (!req.user) {
    return res.sendStatus(401);
  }

  res.status(200).send({user: req.user});
});
```

## Running locally

To run the sample extension locally:

```bash
$ npm install
$ npm start
```

## Deploying as Auth0 Custom Extension

1. Go to [Auth0 Extensions](https://manage.auth0.com/#/extensions)
2. Click on `+ Create Extension`
3. Fill in the textbox with `https://github.com/auth0/auth0-extension-boilerplate`
4. Click on `continue`
5. Finally, click on `install`

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amont others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a free Auth0 Account

1. Go to [Auth0](https://auth0.com/signup) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for more info.
