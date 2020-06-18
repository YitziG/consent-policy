## How to Make Your Project GDPR Compliant

Our legal team requires us to ensure that all new and existing projects are fully [GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation) compliant.

A big part of being GDPR compliant is getting explicit user permission when creating or accessing information that could potentially be used to identify or track users.

For specific categories of user information (i.e., Cookies), permission needs to be requested from the user at runtime, before we can execute the "sensitive" code.

Therefore, it is crucial to have a clear picture of exactly which categories of permissions your app will need to request so that you can do so in the right place and at the right time.

### Big Picture

To make this transition as painless as possible, we provide a small javascript library that you can include in your app's header or in any location where you are pulling in external scripts(i.e., main.js).

Using this library, you can get a boolean flag map from the server that describes precisely which permissions your app must request. 

Using [EJS](https://ejs.co) or any other templating language or framework, you can bind those values to DOM elements displaying and hiding permission requests as appropriate.

### Integration

The first step is to include our script in your project header (or wherever you pull in external scripts).

```html
<!doctype html>
<html>

<head>
  <meta charset="utf-8">
  <title>Awesome Wix Integration!</title>
  <!-- Here is a good place for our script tag -->
  <script src="https://static.parastorage.com/services/consent-policy-client/1.0.0/app.bundle.js"></script>
</head>

<body>  
  <p>Hello world! This is our teams integration.</p>
</body>
</html>
```

**Tip:** If you are sure that your project doesn't need to request permissions until after the page finishes loading, you can include our script tag at the end of your `<body>` for a faster page load.
  
```html
<body>  
  <p>Hello world! This is our teams integration.</p>
  <script src="https://static.parastorage.com/services/consent-policy-client/1.0.0/app.bundle.js"></script>
</body>
```

Once the page finishes loading you can initialize the library provided "consent policy" object with the live data from the server like so: 
```javascript
consentPolicy.init(policy);
```

You can access the flag map with:
```javascript
consentPolicy.categories
```

To visualize this you can type `consentPolicy.categories` in the console, and you will get something like this:
```json
{
  marketing: false, 
  logging: false, 
  stability: true
};
```

It is now trivial to bind relevant parts of your policy to the appropriate fields on this object!

```html
<p>Please grant the following required permissions:</p>

<% if (consentPolicy.categories.marketing) { %>
<div>
  <input type="checkbox" id="marketing" name="marketing"
         checked>
  <label for="marketing">Marketing</label>
</div>
<% } %>
  
<% if (consentPolicy.categories.logging) { %>
<div>
  <input type="checkbox" id="logging" name="logging">
  <label for="logging">Logging</label>
</div>
<% } %>

<% if (consentPolicy.categories.stability) { %>
<div>
  <input type="checkbox" id="stability" name="stability">
  <label for="stability">Stability</label>
</div>
<% } %>
```

Using the example map from above the rendered HTML would look like so:
```html
<p>Please grant the following required permissions:</p>

<div>
  <input type="checkbox" id="stability" name="stability">
  <label for="stability">Stability</label>
</div>
```
And in the browser like so:

Please grant the following required permissions:

- [ ] Stability

### Real-time Permission Checks

Now that you are requesting the appropriatte permissions you are going to want to make sure that the user has indeed granted the required permission before you store any identifying data in the users browser local storage or on Wix servers. We provide an easy way to do that integrated right into our ["data-capsule" library](https://github.com/wix/data-capsule).

The data capsule library is a neat and easy way to store key/value data for your application. You can use it to store data on the user's browsers local storage as well as on Wix servers.

First use your package manager (we recommend NPM) to install the ["data-capsule"](https://github.com/wix/data-capsule) package. 
```shell
npm install --save data-capsule
```

In the past (before GDPR) setting a browser cookie was as simple as:
```javascript
import {LocalStorageCapsule} from 'data-capsule';

const capsule = LocalStorageCapsule({namespace: '${your-app-namespace}'});
await capsule.setItem('shahata', 123);
```

Now in order to be GDPR compliant we need to wrap that code in a try/catch that will throw and handle an error if the appropriatte permission has not been granted. 

Here are the changes you will need to make:

1. On the top line, add COOKIE_CONTENT_DISALLOWED to your import
```javascript
import { LocalStorageCapsule, COOKIE_CONSENT_DISALLOWED } from 'data-capsule';
```
2. Pass the cookie category along with the cookie in `capsule.setItem()
```javascript
await capsule.setItem('shahata', 123, { category: 'advertising' });
```
This will throw a `COOKIE_CONTENT_DISSALOWED` error if the user has not granted the appropriatte permission.

Therefore, this line needs to be wrapped in a try/catch block and handled appropriately.

Your code should now look something like this:
```javascript
import { LocalStorageCapsule, COOKIE_CONSENT_DISALLOWED } from 'data-capsule';

const capsule = LocalStorageCapsule({namespace: '${your-app-namespace}'});

try {
  await capsule.setItem('shahata', 123, { category: 'advertising' });
} catch (e) {
  if (e === COOKIE_CONSENT_DISALLOWED) {
    requestPermission()
  } else {
    displayErrorMessage()
  }
}
```
