# JavaScript

We use [Babel](https://babeljs.io/) for compiling our code into Javascript. It
polyfills most of the standard built-in objects to match the documentation on [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects)
and compiles from ES6 into Javascript. Unless otherwise specified, an ES6 feature
is considered good practice, specially if it makes the code easier to understand.


## Modules

We have chosen to use the CommonJS syntax (`require()` and `module.exports`) for
requiring and exporting modules.

Include all your requires at the top of the file using `const`. Both single declaration and multiple declaration are valid.

```javascript
// SomeComponent.js
const React = require('react'),
    classnames = require('classnames'),
    { TiltLogo, TiltAirplane } = require('tilt-images'),
    SendEmailLightbox = require('../lightboxes/SendEmailLightbox'),
    LightboxBase = require('root/lightbox/LightboxBase');

// SomeOtherComponent.js
const React = require('react');
const classnames = require('classnames');
const { TiltLogo, TiltAirplane } = require('tilt-images');
const SendEmailLightbox = require('../lightboxes/SendEmailLightbox');
const LightboxBase = require('root/lightbox/LightboxBase');
```

- The base of the project is `root`. Use it as an alternative to relative paths.
- Don't use default exports as they are not easy to include using CommonJS.
- Don't require modules dynamically as this makes them hard to validate on compile
time.


## Declaring objects

Take advantage of the shorthand functions for declaring objects. It's best to declare
an object in a single statement.

```javascript
const eventName = "mouseout";
const attr = {};
const obj = {
    attr,
    [eventName]: function() {},
    click() {},
};

// Avoid creating obj['undefined'] when this.props.mouseover is undefined
if (this.props.mouseover) {
    obj.mouseover = this.props.mouseover;
}
```

## Isomorphic Javascript

Our code will run in the server and the browser. Some common assumptions of single
environment Javascript are not valid in an Isomorphic world.

### Window object.

Use `typeof window === "undefined"` to check for availability of the `window`
object. The common `window === undefined` will throw a `ReferenceError: window is not defined`
inside node.

```javascript
if (typeof window === "undefined") {
  window.addEventListener('resize', function (e) {});
}
```

## Asynchronous calls, callbacks and promises

Try to avoid callback hell by avoiding asynchronous calls ;) and delegate to the
dispatcher as much as possible. When it's necessary to handle a callback, consider
Promises first as they keep the interface predictable.

### Ajax requests

We use [superagent](http://visionmedia.github.io/superagent/) for http calls.
Ajax calls should live inside Actions and ideally handle callbacks via Promises.

```javascript
const request = require('superagent');

UserActions.verifyEmail: function(dispatcher, email) {
  return new Promise(function(resolve, reject) {
    request
      .put('/email/verify')
      .csrf()
      .send({ email: email })
      .end(function(err, res) {
          if(err) {
            reject(err);
          }
          else {
            dispatcher.dispatch('userResetInfo', res.body);
            resolve(res.body);
          }
      });
  });
};
```

Most of the time it's ok to ignore the resulting promise as the state transitions
will be handled by the dispatcher. However, many components benefit from using
the promise to track the loading state.

```javascript
// UserEmailVerify.js
//........
  handleVerifySubmit(email) {
    this.setState({
      isLoading: true
    });
    UserActions.verifyEmail(this.context.dispatcher, email)
      .then(() => {
        this.setState({
          isLoading: false
        });
      })
      .catch(() => {
        // Don't forget to handle the error case!
        this.setState({
          isLoading: false
        });
      });
  },
//.........
```

## Replacing library patterns

We have deprecated usage of jQuery and underscore within our codebase. Most of the utility functions they provide are readily available in ES6.

### Merging objects:

Don't do

```javascript
_.extend(object1, object2, object3, ...);
$.extend(object1, object2, object3, ...);
```

Do:

```javascript
Object.assign(object1, object2, object3, ...)
```

**[tip]** Object.assign has the side effect of modifying object1. If you want to
get a new copy of object then pass an empty object as the first argument. This is
also a good way to set default properties:

```javascript
position = Object.assign({
  position: "absolute"
}, position);
```

### Checking if an object is an array

Use `Array.isArray`

```javascript
Array.isArray(someObject);
```

### Checking if an object is a function

```javascript
typeof someObject === "function";
```

### Iterating over collections

Both jQuery and underscore provide shorthand functions for iterating over key-value
pairs in objects.

```javascript
$.each(someObject, function(value, key) {
});
_.each(someObject, function(value, key) {
});
```

```javascript
// Iterating over values on an array
someArray.forEach(function(index, value) {});

// Iterating over key value pairs on an object.
Object.keys(someObject).forEach(function(key) {
    const value = someObject[key];
});
```
