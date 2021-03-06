# Simple OAuth2 Facebook

This library is a wrapper around [Simple OAuth2 Library](https://github.com/lelylan/simple-oauth2)

Specially made for [Authorization Code Flow](https://tools.ietf.org/html/draft-ietf-oauth-v2-31#section-4.1) with Facebook.

## Requirements

Latest Node 8 LTS or newer versions.

## Getting started

```
npm install --save simple-oauth2 @jimmycode/simple-oauth2-facebook
```

or 

```
yarn add simple-oauth2 @jimmycode/simple-oauth2-facebook
```

### Usage

```js
const simpleOAuth2Facebook = require('@jimmycode/simple-oauth2-facebook');
const facebook = simpleOAuth2Facebook.create(options);
```

`facebook` object exposes 3 keys:
* authorize: Middleware to request user's authorization.
* getToken: Middleware for callback processing and exchange the authorization token for an `access_token`
* oauth2: The underlying [simple-oauth2](https://github.com/lelylan/simple-oauth2) instance.

### Options

SimpleOAuth2Facebook comes with default values for most of the options.

**Required options**

| Option       | Description                                   |
|--------------|-----------------------------------------------|
| clientId     | Your App Id.                                  |
| clientSecret | Your App Secret Id.                           |
| callbackURL  | Callback configured when you created the app. |


**Other options**

| Option           | Default                      | Description                                                                                                                                                                               |
|------------------|------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| scope            | ['email']                    | https://developers.facebook.com/docs/facebook-login/permissions                                                                                                                           |
| state            | ''                           | Your CSRF anti-forgery token. More at: https://auth0.com/docs/protocols/oauth2/oauth-state                                                                                                |
| returnError      | false                        | When is false (default), will call the next middleware with the error object. When is true, will set req.tokenError to the error, and call the next middleware as if there were no error. |
| authorizeHost    | 'https://facebook.com'       |                                                                                                                                                                                           |
| authorizePath    | '/dialog/oauth'              |                                                                                                                                                                                           |
| tokenHost        | 'https://graph.facebook.com' |                                                                                                                                                                                           |
| tokenPath        | '/oauth/access_token'        |                                                                                                                                                                                           |
| authorizeOptions | {}                           | Pass extra parameters when requesting authorization.                                                                                                                                      |
| tokenOptions     | {}                           | Pass extra parameters when requesting access_token.                                                                                                                                       |

## Example

### Original boilerplate

```js
const oauth2 = require('simple-oauth2').create({
  client: {
    id: process.env.FACEBOOK_APP_ID,
    secret: process.env.FACEBOOK_APP_SECRET
  },
  auth: {
    authorizeHost: 'https://facebook.com/',
    authorizePath: '/dialog/oauth',

    tokenHost: 'https://graph.facebook.com',
    tokenPath: '/oauth/access_token'
  }
});

router.get('/auth/facebook', (req, res) => {
  const authorizationUri = oauth2.authorizationCode.authorizeURL({
    redirect_uri: 'http://localhost:3000/auth/facebook/callback',
    scope: ['email']
  });

  res.redirect(authorizationUri);
});

router.get('/auth/facebook/callback', async(req, res) => {
  const code = req.query.code;
  const options = {
    code,
    redirect_uri: 'http://localhost:3000/auth/facebook/callback'
  };

  try {
    // The resulting token.
    const result = await oauth2.authorizationCode.getToken(options);

    // Exchange for the access token.
    const token = oauth2.accessToken.create(result);

    return res.status(200).json(token);
  } catch (error) {
    console.error('Access Token Error', error.message);
    return res.status(500).json('Authentication failed');
  }
});
```

### With SimpleOAuth2Facebook

```js
const simpleOAuth2Facebook = require('@jimmycode/simple-oauth2-facebook');

const facebook = simpleOAuth2Facebook.create({
  clientId: process.env.FACEBOOK_APP_ID,
  clientSecret: process.env.FACEBOOK_APP_SECRET,
  callbackURL: 'http://localhost:3000/auth/facebook/callback'
});

// Ask the user to authorize.
router.get('/auth/facebook', facebook.authorize);

// Exchange the token for the access token.
router.get('/auth/facebook/callback', facebook.accessToken, (req, res) => {
  return res.status(200).json(req.token);
});
```
