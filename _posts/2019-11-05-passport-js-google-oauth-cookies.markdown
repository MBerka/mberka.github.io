---
layout: post
title:  "Passport.js, Google Oauth, and Cookie Sessions: avoiding tummy aches"
date:   2019-11-05 17:21:30 -0700
permalink: /passport-js-google-oauth-cookies
categories: software
---
When new to Express, I was pleased to find an [introduction to using Google OAuth with Passport.js](https://www.freecodecamp.org/news/pora-quick-introduction-to-oauth-using-passport-js-65ea5b621a/). 
With an added email scope and a [state parameter to keep track of the location before sign-in](https://developers.google.com/identity/protocols/OAuth2WebServer#request-parameter-state),

  ```javascript
  app.get('/auth/google', (req, res, next) => {
    passport.authenticate('google', {
      scope: ['profile', 'email'], // profile does not include email
      state: pathToLastPage, // Google will return this
    })(req, res, next);
  });
  ```
it helped create a reliable way of associating Google accounts with accounts in web apps. All worked well until a custom-domain (Google Apps) account stopped being able to sign in this way, when there had been no changes to the authentication code.

All the pieces were there. `gmail.com` accounts continued to work as before, but selecting an account on the custom domain always led to a **502 Bad Gateway** error after Google returned the user.

The first clue about the problem came when the proxy-level error was fixed by [configuring buffers](https://stackoverflow.com/questions/3704626/nginx-502-bad-gateway-error-only-in-firefox) (not where I found the answer, but it should give a clearer hint of what the problem was). Now the user was successfully returned to the app, **still not authenticated**.

Logs showed no errors; the server was receiving all needed information from Google, and serializing it to the user's cookie. **Only gmail accounts' information was succesfully deserialized**.

Comparing requests from accounts of both types, I found a difference:

User object for Gmail account:
```
{
  id: '101234567890123456789',
  displayName: 'First Last',
  name: { familyName: 'Last', givenName: 'First' },
  emails: [ { value: 'email@gmail.com', verified: true } ],
  photos: [
    {
      value: 'https://lh6.googleusercontent.com/115-char-long-url.jpg'
    }
  ],
  provider: 'google',
  _raw: '{\n  "sub": "101234567890123456789",\n  "name": "First Last",\n  ' +
    '"given_name": "First",\n  "family_name": "Last",\n  "picture": ' +
    '"https://lh6.googleusercontent.com/115-char-long-url.jpg",\n' +
    '  "email": "email@gmail.com",\n  "email_verified": true,\n ' +
    ' "locale": "en"\n}',
  _json: {
    sub: '101234567890123456789',
    name: 'First Last',
    given_name: 'First',
    family_name: 'Last',
    picture: 'https://lh6.googleusercontent.com/115-char-long-url.jpg',
    email: 'email@gmail.com',
    email_verified: true,
    locale: 'en'
  }
}
```

The user object for Google Apps account had the same structure, but **each image url was 808 characters**. This more than tripled the length of the object source, from ~1kb to over 3kb, longer once escaped. Browsers may limit cookies to 4kb and I did not need photos or redundant copies of the same information, so the quickest solution was to modify the user object before it was serialized. The callback provided as the last argument to `passport.use(new GoogleStrategy())` provides the chance:

```javascript
(accessToken, refreshToken, profile, done) => {
    // Relay only the fields we need from profile to serializeUser
    done(null, {
        id: profile.id,
        displayName: profile.displayName,
        emails: profile.emails,
        provider: profile.provider,
    });
})
```
(this is also a good spot for other pre-cookie validation).

This allowed the account to sign in as normal. Authentication broke because Google started returning a lsome initialsome initialsome initialonger URL for the Google Apps accounts's profile photo. **When using Cookie Sessions, taking pains to keep the serialized object minimal can save pain later.**
